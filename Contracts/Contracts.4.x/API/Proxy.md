# Proxies
这是一组低级别的合约，实现了不同的代理模式，包括具有和不具有升级功能的代理模式。要详细了解此模式，请参阅[Proxy Upgrade Pattern](../../../Upgrades-Plugins/Proxy-Upgrade-Pattern.md)页面。

下面的大多数代理都是基于一个抽象基础合约构建的。

* [Proxy](#proxy)：实现核心委托功能的抽象合约。

为了避免与代理后面的实现合约的存储变量冲突，我们使用[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)存储槽。

* [ERC1967Upgrade](#erc1967upgrade)：用于获取和设置[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)定义的存储槽的内部函数。

* [ERC1967Proxy](#erc1967proxy)：使用EIP1967存储槽的代理。默认情况下不可升级。

有两种不同的方法可以向ERC1967代理添加升级功能。它们的区别在下面的[Transparent vs UUPS Proxies](#透明代理和uups代理之间的区别)中解释。

* [TransparentUpgradeableProxy](#transparentupgradeableproxy)：具有内置管理员和升级接口的代理。

* [UUPSUpgradeable](#uupsupgradeable)：一种可以包含在实现合约中的可升级性机制。

> CAUTION
正确和安全地使用可升级代理是一项困难的任务，需要对代理模式、Solidity和EVM有深入的了解。除非您需要很多低级别的控制，否则我们建议使用[OpenZeppelin Upgrades](../../../Upgrades-Plugins/Overview.md)插件来进行Truffle和Hardhat的升级。

另一组代理是beacon代理。这种模式由Dharma推广，允许多个代理在单个事务中升级到不同的实现。

* [BeaconProxy](#beaconproxy)：从beacon合约检索其实现的代理。

* [UpgradeableBeacon](#upgradeablebeacon)：带有内置管理员的beacon合约，可以升级指向它的[BeaconProxy](#beaconproxy)。

在这种模式中，代理合约不像ERC1967代理那样在存储中保存实现地址。相反，地址存储在一个单独的beacon合约中。升级操作发送到beacon而不是代理合约，并且所有遵循该beacon的代理都会自动升级。

在不可升级的情况下，代理还可以用于创建廉价的合约克隆，例如由一个在链工厂合约创建的多个相同合约实例。这些实例旨在部署廉价，并且调用廉价。

* [Clones](#clones)：一个可以部署廉价的最小非升级代理的库。

## 透明代理和UUPS代理之间的区别
OpenZeppelin中最初提供的代理遵循了[透明代理模式](https://blog.openzeppelin.com/the-transparent-proxy-pattern/)。虽然仍然提供该模式，但我们现在的建议正在转向UUPS代理，这些代理既轻量又灵活。UUPS的名称来自于[EIP1822](https://eips.ethereum.org/EIPS/eip-1822)，该规范首次记录了该模式。

虽然这两种代理都共享相同的升级接口，但在UUPS代理中，升级由实现处理，并且最终可以被移除。而透明代理则在代理本身中包含了升级和管理员逻辑。这意味着[TransparentUpgradeableProxy](#transparentupgradeableproxy)的部署成本比UUPS代理更高。

UUPS代理是使用[ERC1967Proxy](#erc1967proxy)实现的。请注意，该代理本身不可升级。实现的角色是在合约的逻辑之外，包含所有必要的代码来更新存储在代理存储空间的特定槽位中的实现地址。这就是*UUPSUpgradeable*合约的作用。继承它（并使用相关的访问控制机制重写 [_authorizeUpgrade](#_authorizeupgradeaddress-newimplementation) 函数）将使您的合约成为符合UUPS的实现。

请注意，由于两种代理都使用相同的存储槽位来存储实现地址，使用符合UUPS的实现与[TransparentUpgradeableProxy](#transparentupgradeableproxy)结合使用可能会允许非管理员执行升级操作。

默认情况下，[UUPSUpgradeable](#uupsupgradeable)中包含的升级功能包含一种安全机制，可以防止对非符合UUPS的实现进行升级。这可以防止对不包含必要升级机制的实现合约进行升级，因为这将永远锁定代理的可升级性。这种安全机制可以通过以下任一方式绕过：

* 在实现中添加一个标志机制，在触发时禁用升级函数。

* 升级到具有升级机制但没有附加安全检查的实现，然后再次升级到另一个没有升级机制的实现。

当前这种安全机制的实现使用了[EIP1822](https://eips.ethereum.org/EIPS/eip-1822)来检测实现使用的存储槽位。以前的实现，现在已经弃用，依赖于回滚检查。可以从使用旧机制的合约升级到新的合约。但反过来则不可能，因为旧的实现（版本4.5之前）不包含ERC1822接口。

## 核心

### Proxy
```
import "@openzeppelin/contracts/proxy/Proxy.sol";
```

这个抽象合约提供了一个回退函数，它使用EVM指令delegatecall将所有调用委托给另一个合约。我们将第二个合约称为代理后面的实现，需要通过重写虚拟函数[_implementation](#implementation-→-address)来指定它。

此外，可以通过[_fallback](#_fallback)函数手动触发对实现的委托，或者通过[_delegate](#_delegateaddress-implementation)函数委托给另一个合约。

委托调用的成功和返回数据将返回给代理的调用者。

**FUNCTIONS**
[_delegate(implementation)](#_delegateaddress-implementation)
[_implementation()](#_implementation-→-address)
[_fallback()](#_fallback)
[fallback()](#_fallback)
[receive()](#receive)
[_beforeFallback()](#_beforefallback)

#### _delegate(address implementation)
内部#
将当前调用委托给实现。

该函数不会返回到其内部调用点，而是直接返回给外部调用者。

#### _implementation() → address
内部#
这是一个应该被重写的虚函数，它应该返回fallback函数和[_fallback](#_fallback)函数委托的地址。

#### _fallback()
内部#
将当前调用委托给由_implementation()返回的地址。

这个函数不会返回到它的内部调用点，它将直接返回给外部调用者。

#### fallback()
外部#
回退函数将调用委托给由_implementation()返回的地址。如果合约中没有其他函数与调用数据匹配，将运行该函数。

#### receive()
外部#
回退函数将调用委托给 _implementation() 返回的地址。如果调用数据为空，则会运行。

#### _beforeFallback()
内部#
在回退到实现之前调用的 hooks 。可以作为手动回退调用的一部分，也可以作为Solidity回退或接收函数的一部分。

如果被重写，应该调用super._beforeFallback()。

## ERC1967

### ERC1967Proxy
```
import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";
```

该合约实现了一个可升级的代理。它是可升级的，因为调用被委托给一个可以更改的实现地址。该地址存储在存储器中，位置由[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)指定，以避免与代理背后的实现的存储布局冲突。

**FUNCTIONS**
[constructor(_logic, _data)](#constructoraddress-_logic-bytes-_data)
[_implementation()](#_implementation-→-address-impl)

ERC1967UPGRADE
[_getImplementation()](#_getimplementation-→-address)
[_upgradeTo(newImplementation)](#_upgradetoaddress-newimplementation)
[_upgradeToAndCall(newImplementation, data, forceCall)](#_upgradetoandcalladdress-newimplementation-bytes-data-bool-forcecall)
[_upgradeToAndCallUUPS(newImplementation, data, forceCall)](#_upgradetoandcalluupsaddress-newimplementation-bytes-data-bool-forcecall)
[_getAdmin()](#_getadmin-→-address)
[_changeAdmin(newAdmin)](#_changeadminaddress-newadmin)
[_getBeacon()](#_getbeacon-→-address)
[_upgradeBeaconToAndCall(newBeacon, data, forceCall)](#_upgradebeacontoandcalladdress-newbeacon-bytes-data-bool-forcecall)

PROXY
[_delegate(implementation)](#_delegateaddress-implementation)
[_fallback()](#_fallback)
[fallback()](#_fallback)
[receive()](#receive)
[_beforeFallback()](#_beforefallback)

**EVENTS**

IERC1967
[Upgraded(implementation)](./Interfaces.md)
[AdminChanged(previousAdmin, newAdmin)](./Interfaces.md)
[BeaconUpgraded(beacon)](./Interfaces.md)

#### constructor(address _logic, bytes _data)
公开#
使用由_logic指定的初始实现初始化可升级代理。

如果_data非空，则将其用作对_logic的委托调用中的数据。这通常是一个编码的函数调用，并允许像Solidity构造函数一样初始化代理的存储。

#### _implementation() → address impl
内部#
返回当前实现地址。

### ERC1967Upgrade
```
import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Upgrade.sol";
```

这个抽象合约为[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)槽位提供了getter和事件发射的更新函数。
*从v4.1版本开始可用。*

**FUNCTIONS**
[_getImplementation()](#_getimplementation-→-address)
[_upgradeTo(newImplementation)](#_upgradetoaddress-newimplementation)
[_upgradeToAndCall(newImplementation, data, forceCall)](#_upgradetoandcalladdress-newimplementation-bytes-data-bool-forcecall)
[_upgradeToAndCallUUPS(newImplementation, data, forceCall)](#_upgradetoandcalluupsaddress-newimplementation-bytes-data-bool-forcecall)
[_getAdmin()](#_getadmin-→-address)
[_changeAdmin(newAdmin)](#_changeadminaddress-newadmin)
[_getBeacon()](#_getbeacon-→-address)
[_upgradeBeaconToAndCall(newBeacon, data, forceCall)](#_upgradebeacontoandcalladdress-newbeacon-bytes-data-bool-forcecall)

**EVENTS**

IERC1967
[Upgraded(implementation)]
[AdminChanged(previousAdmin, newAdmin)]
[BeaconUpgraded(beacon)]

#### _getImplementation() → address
内部#
返回当前实现地址。

#### _upgradeTo(address newImplementation)
内部#
执行升级实现

发出{升级}事件。

#### _upgradeToAndCall(address newImplementation, bytes data, bool forceCall)
内部#
执行升级实施，并进行额外的设置调用。

发出一个{升级}事件。

#### _upgradeToAndCallUUPS(address newImplementation, bytes data, bool forceCall)
内部#
进行UUPS代理的实施升级，包括安全检查和额外的设置调用。

触发一个{Upgraded}事件。

#### _getAdmin() → address
内部#
返回当前管理员。

#### _changeAdmin(address newAdmin)
内部#
更改代理的管理员。

触发一个{AdminChanged}事件。

#### _getBeacon() → address
内部#
返回当前的指引。

#### _upgradeBeaconToAndCall(address newBeacon, bytes data, bool forceCall)
内部#
进行信标升级并进行附加的设置调用。注意：这将升级信标的地址，而不是升级信标中包含的实现（有关此操作，请参阅 [UpgradeableBeacon._setImplementation](#_setbeaconaddress-beacon-bytes-data)）。

发出 {BeaconUpgraded} 事件。

## Transparent Proxy

### TransparentUpgradeableProxy
```
import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
```

该合约实现了一个可由管理员升级的代理。

为了避免潜在的[代理选择器冲突](https://medium.com/nomic-labs-blog/malicious-backdoors-in-ethereum-proxies-62629adf3357)，该合约采用[透明代理模式](https://blog.openzeppelin.com/the-transparent-proxy-pattern/)。该模式意味着两个相辅相成的事情：

1. 如果除管理员之外的任何账户调用代理，即使该调用与代理本身公开的管理员函数相匹配，调用也会被转发到实现上。

2. 如果管理员调用代理，它可以访问管理员函数，但它的调用将永远不会被转发到实现上。如果管理员尝试调用实现上的函数，将会失败并显示错误信息"admin cannot fallback to proxy target"。

这些属性意味着管理员账户只能用于管理员操作，如升级代理或更改管理员，所以最好是一个专用账户，不要用于其他任何用途。这样可以避免在尝试从代理实现中调用函数时出现突然的错误，从而带来麻烦。

我们建议专用账户是[ProxyAdmin](#proxyadmin)合约的一个实例。如果以这种方式设置，您应该将ProxyAdmin实例视为代理的真正管理接口。

> NOTE
该代理的真正接口是在ITransparentUpgradeableProxy中定义的接口。该合约不继承该接口，而是在_fallback中使用自定义调度机制隐式实现管理员函数。因此，编译器不会为该合约生成ABI。这是为了完全实现透明性，避免由于代理和实现之间的选择器冲突而解码失败。

> WARNING
不建议扩展该合约以添加其他外部函数。如果这样做，由于上述说明，编译器将不会检查是否存在选择器冲突。任何新函数与[ITransparentUpgradeableProxy]中声明的函数之间的选择器冲突将优先选择新函数。这可能导致无法访问管理员操作，从而阻止升级能力。透明性也可能受到损害。

**MODIFIERS**
[ifAdmin()](#ifadmin)

**FUNCTIONS**
[constructor(_logic, admin_, _data)](#constructoraddress-_logic-bytes-_data)
[_fallback()](#_fallback)
[_admin()](#_admin-→-address)

ERC1967PROXY
[_implementation()](#_implementation-→-address-impl)

ERC1967UPGRADE
[_getImplementation()](#_getimplementation-→-address)
[_upgradeTo(newImplementation)](#_upgradetoaddress-newimplementation)
[_upgradeToAndCall(newImplementation, data, forceCall)](#_upgradetoaddress-newimplementation)
[_upgradeToAndCallUUPS(newImplementation, data, forceCall)](#_upgradetoandcalluupsaddress-newimplementation-bytes-data-bool-forcecall)
[_getAdmin()](#_getadmin-→-address)
[_changeAdmin(newAdmin)](#_changeadminaddress-newadmin)
[_getBeacon()](#_getbeacon-→-address)
[_upgradeBeaconToAndCall(newBeacon, data, forceCall)](#_upgradebeacontoandcalladdress-newbeacon-bytes-data-bool-forcecall)

PROXY
[_delegate(implementation)](#_delegateaddress-implementation)
[fallback()](#fallback)
[receive()](#receive)
[_beforeFallback()](#_beforefallback)

**EVENTS**
IERC1967
[Upgraded(implementation)](./Interfaces.md)
[AdminChanged(previousAdmin, newAdmin)](./Interfaces.md)
[BeaconUpgraded(beacon)](./Interfaces.md)

#### ifAdmin()
修饰符# 
用于内部使用的修饰符，除非发送者是管理员，否则将调用委托给实现。

> CAUTION
该修饰符已弃用，因为如果修改的函数具有参数，并且实现提供具有相同选择器的函数，则可能会导致问题。

#### constructor(address _logic, address admin_, bytes _data)
公开#
通过_admin初始化一个可升级的代理，由_logic的实现支持，并可选择使用[ERC1967Proxy.constructor](#constructoraddress-_logic-bytes-_data)中解释的_data进行初始化。

#### _fallback()
内部#
如果调用者是管理员进程，则在内部处理呼叫，否则透明地回退到代理行为。

#### _admin() → address
内部#
返回当前管理员。

此函数已弃用。请使用[ERC1967Upgrade._getAdmin](#_getadmin-→-address)代替。

### ProxyAdmin
```
import "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";
```

这是一个辅助合约，用于将其分配为[TransparentUpgradeableProxy](#transparentupgradeableproxy)的管理员。有关为什么要使用它的解释，请参阅[TransparentUpgradeableProxy](#transparentupgradeableproxy)的文档。

**FUNCTIONS**
[getProxyImplementation(proxy)](#getproxyimplementationcontract-itransparentupgradeableproxy-proxy-→-address)
[getProxyAdmin(proxy)](#getproxyadmincontract-itransparentupgradeableproxy-proxy-→-address)
[changeProxyAdmin(proxy, newAdmin)](#changeproxyadmincontract-itransparentupgradeableproxy-proxy-address-newadmin)
[upgrade(proxy, implementation)](#upgradecontract-itransparentupgradeableproxy-proxy-address-implementation)
[upgradeAndCall(proxy, implementation, data)](#upgradeandcallcontract-itransparentupgradeableproxy-proxy-address-implementation-bytes-data)

OWNABLE
[owner()](./Access.md#owner-→-address)
[_checkOwner()](./Access.md#_checkowner)
[renounceOwnership()](./Access.md#renounceownership)
[transferOwnership(newOwner)](./Access.md#transferownershipaddress-newowner)
[_transferOwnership(newOwner)](./Access.md#_transferownershipaddress-newowner)

**EVENTS**
OWNABLE
[OwnershipTransferred(previousOwner, newOwner)](./Access.md#ownershiptransferredaddress-indexed-previousowner-address-indexed-newowner)

#### getProxyImplementation(contract ITransparentUpgradeableProxy proxy) → address
公开#
返回代理的当前实现。
要求：
* 此合约必须是代理的管理员。

#### getProxyAdmin(contract ITransparentUpgradeableProxy proxy) → address
公开#
返回代理的当前管理员。
要求：
* 此合约必须是代理的管理员。

#### changeProxyAdmin(contract ITransparentUpgradeableProxy proxy, address newAdmin)
公开#
将代理的管理员更改为新管理员。
要求：
* 此合约必须是代理的当前管理员。

#### upgrade(contract ITransparentUpgradeableProxy proxy, address implementation)
公开#
将代理升级为实现。请参阅{TransparentUpgradeableProxy-upgradeTo}。
要求：
* 此合约必须是代理的管理员。

#### upgradeAndCall(contract ITransparentUpgradeableProxy proxy, address implementation, bytes data)
公开#
将代理升级为实现合约，并在新实现上调用一个函数。参见{TransparentUpgradeableProxy-upgradeToAndCall}。
要求：
* 此合约必须是代理的管理员。

## 信标

### BeaconProxy
```
import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";
```

此合约实现了一个代理，它从[UpgradeableBeacon](#upgradeablebeacon)获取每个调用的实现地址。

beacon地址存储在存储槽uint256(keccak256('eip1967.proxy.beacon')) - 1中，以避免与代理后面的实现的存储布局冲突。

*自v3.4起可用。*

*FUNCTIONS*
[constructor(beacon, data)](#constructoraddress-beacon-bytes-data)
[_beacon()](#_beacon-→-address)
[_implementation()](#_implementation-→-address)
[_setBeacon(beacon, data)](#_setbeaconaddress-beacon-bytes-data)

ERC1967UPGRADE
[_getImplementation()](#_getimplementation-→-address)
[_upgradeTo(newImplementation)](#_upgradetoaddress-newimplementation)
[_upgradeToAndCall(newImplementation, data, forceCall)](#_upgradetoandcalladdress-newimplementation-bytes-data-bool-forcecall)
[_upgradeToAndCallUUPS(newImplementation, data, forceCall)](#_upgradetoandcalluupsaddress-newimplementation-bytes-data-bool-forcecall)
[_getAdmin()](#_getadmin-→-address)
[_changeAdmin(newAdmin)](#_changeadminaddress-newadmin)
[_getBeacon()](#_getbeacon-→-address)
[_upgradeBeaconToAndCall(newBeacon, data, forceCall)](#_upgradebeacontoandcalladdress-newbeacon-bytes-data-bool-forcecall)

PROXY
[_delegate(implementation)](#_delegateaddress-implementation)
[_fallback()](#_fallback)
[fallback()](#_fallback)
[receive()](#receive)
[_beforeFallback()](#_beforefallback)

**EVENTS**
IERC1967
[Upgraded(implementation)](./Interfaces.md)
[AdminChanged(previousAdmin, newAdmin)](./Interfaces.md)
[BeaconUpgraded(beacon)](./Interfaces.md)

#### constructor(address beacon, bytes data)
公开#
使用信标初始化代理。

如果数据非空，则将其用作委托调用到信标返回的实现中的数据。这通常是一个编码的函数调用，并允许像Solidity构造函数一样初始化代理的存储。

要求：

信标必须是一个具有[IBeacon](#ibeacon)接口的合约。

#### _beacon() → address
内部#
返回当前信标地址。

#### _implementation() → address
内部#
返回关联信标的当前实施地址。

#### _setBeacon(address beacon, bytes data)
内部#
将代理更改为使用一个新的信标。已弃用：请参见[_upgradeBeaconToAndCall](#_upgradebeacontoandcalladdress-newbeacon-bytes-data-bool-forcecall)。

如果数据不为空，则将其用作委托调用中使用的数据，该委托调用将发送到信标返回的实现。
要求：
* 信标必须是一个合约。
* 信标返回的实现必须是一个合约。

### IBeacon
```
import "@openzeppelin/contracts/proxy/beacon/IBeacon.sol";
```

这是[BeaconProxy](#implementation-→-address)对其beacon所期望的接口。

**FUNCTIONS**
[implementation()](#implementation-→-address)

#### implementation() → address
外部#
必须返回一个可用作委托调用目标的地址。

[BeaconProxy](#beaconproxy)将检查该地址是否为一个合约。

#### UpgradeableBeacon
```
import "@openzeppelin/contracts/proxy/beacon/UpgradeableBeacon.sol";
```

该合同与一个或多个[BeaconProxy](#beaconproxy)实例一起使用，以确定它们的实现合同，即它们将委托所有函数调用的位置。
所有者可以更改beacon指向的实现，从而升级使用该beacon的代理。

**FUNCTIONS**
[constructor(implementation_)](#constructoraddress-implementation_)
[implementation()](#implementation-→-address)
[upgradeTo(newImplementation)](#upgradetoaddress-newimplementation)

OWNABLE
[owner()](./Access.md#owner-→-address)
[_checkOwner()](./Access.md#_checkowner)
[renounceOwnership()](./Access.md#renounceownership)
[transferOwnership(newOwner)](./Access.md#transferownershipaddress-newowner)
[_transferOwnership(newOwner)](./Access.md#_transferownershipaddress-newowner)

**EVENTS**
[Upgraded(implementation)](#upgradedaddress-indexed-implementation)

OWNABLE
[OwnershipTransferred(previousOwner, newOwner)](./Access.md#ownershiptransferredaddress-indexed-previousowner-address-indexed-newowner)

#### constructor(address implementation_)
公开#
将初始实现的地址设置为部署者账户，部署者账户将成为可以升级信标的所有者。

#### implementation() → address
公开#
返回当前实现的地址。

#### upgradeTo(address newImplementation)
公开#
对信标进行升级以实现新的实现。
发出一个[Upgraded](#upgradedaddress-indexed-implementation)事件。
要求：
* msg.sender必须是合约的所有者。
* newImplementation必须是一个合约。

#### Upgraded(address indexed implementation)
事件#
当由beacon返回的实现发生变化时发出。

## Minimal Clones

### Clones
```
import "@openzeppelin/contracts/proxy/Clones.sol";
```

[EIP 1167](https://eips.ethereum.org/EIPS/eip-1167)是部署最小代理合约（也称为“克隆”）的标准。

    为了以一种不可变的方式简单且廉价地复制合约功能，该标准规定了一个最小的字节码实现，将所有调用委托给一个已知的固定地址。

该库包括使用create（传统部署）或create2（带有盐的确定性部署）部署代理的函数。它还包括使用确定性方法部署的克隆的地址预测函数。

*自版本3.4以来可用。*

**FUNCTIONS**
[clone(implementation)](#cloneaddress-implementation-→-address-instance)
[cloneDeterministic(implementation, salt)](#clonedeterministicaddress-implementation-bytes32-salt-→-address-instance)
[predictDeterministicAddress(implementation, salt, deployer)](#predictdeterministicaddressaddress-implementation-bytes32-salt-address-deployer-→-address-predicted)
[predictDeterministicAddress(implementation, salt)](#predictdeterministicaddressaddress-implementation-bytes32-salt-→-address-predicted)

#### clone(address implementation) → address instance
内部#
部署并返回一个克隆实现行为的地址。

该函数使用了create操作码，应该不会发生回滚。

#### cloneDeterministic(address implementation, bytes32 salt) → address instance
内部#
部署并返回一个模仿实现行为的克隆的地址。

该函数使用create2操作码和盐值来确定性地部署克隆体。多次使用相同的实现和盐值将导致回滚，因为克隆体不能在相同的地址上重复部署。

#### predictDeterministicAddress(address implementation, bytes32 salt, address deployer) → address predicted
内部#
计算使用[Clones.cloneDeterministic](#clonedeterministicaddress-implementation-bytes32-salt-→-address-instance)部署的克隆体的地址。

#### predictDeterministicAddress(address implementation, bytes32 salt) → address predicted
内部#
使用[Clones.cloneDeterministic](#clonedeterministicaddress-implementation-bytes32-salt-→-address-instance)部署的克隆体的地址计算。

## 实用程序

#### Initializable
```
import "@openzeppelin/contracts/proxy/utils/Initializable.sol";
```

这是一个基本合约，用于帮助编写可升级合约或任何将部署在代理后面的合约。由于代理合约不使用构造函数，所以通常将构造逻辑移动到一个外部的初始化函数中，通常称为[initialize](#initializer)函数。因此，需要保护这个初始化函数，使其只能被调用一次。这个合约提供的初始化修饰符将具有这种效果。

初始化函数使用一个版本号。一旦使用了一个版本号，它就被消耗掉，不能再重复使用。这个机制防止了每个“步骤”的重新执行，但允许在升级中创建新的初始化步骤，以便初始化新添加的模块。

例如：
```
contract MyToken is ERC20Upgradeable {
    function initialize() initializer public {
        __ERC20_init("MyToken", "MTK");
    }
}

contract MyTokenV2 is MyToken, ERC20PermitUpgradeable {
    function initializeV2() reinitializer(2) public {
        __ERC20Permit_init("MyToken");
    }
}
```

> TIP
为了避免将代理留在未初始化的状态，应尽早通过将编码的函数调用作为_data参数提供给[ERC1967Proxy.constructor](#constructoraddress-_logic-bytes-_data)来调用初始化函数。

> CAUTION
在使用继承时，需要手动注意不要两次调用父级初始化函数，或者确保所有的初始化函数都是幂等的。与Solidity自动验证构造函数不同，这一点不会自动验证。

> CAUTION
避免留下未初始化的合约。

未初始化的合约可能会被攻击者接管。这适用于代理合约及其实现合约，可能会对代理产生影响。为了防止实现合约被使用，你应该在构造函数中调用[_disableInitializers](#_disableinitializers)函数，在部署时自动锁定它：
/// @custom:oz-upgrades-unsafe-allow constructor
constructor() {
    _disableInitializers();
}

**MODIFIERS**
[initializer()](#initializer)
[reinitializer(version)](#reinitializeruint8-version)
[onlyInitializing()](#onlyinitializing)

**FUNCTIONS**
[_disableInitializers()](#_disableinitializers)
[_getInitializedVersion()](#_getinitializedversion-→-uint8)
[_isInitializing()](#_isinitializing-→-bool)

**EVENTS**
[Initialized(version)](#initializeduint8-version)

#### initializer()
修饰符#
一个定义了受保护的初始化函数的修饰符，该函数最多只能被调用一次。在其作用域内，只能使用Initializing函数来初始化父合约。

与reinitializer(1)类似，但是标记为initializer的函数可以嵌套在构造函数的上下文中。

触发一个[Initialized(version)](#initializeduint8-version)事件。

#### reinitializer(uint8 version)
修饰符#
一个修饰符，定义了一个受保护的重新初始化函数，最多只能调用一次，并且只有在合同在之前没有被初始化为更高版本的情况下才能调用。在其作用域内，只能使用Initializing函数来初始化父合同。

重新初始化器可以在原始初始化步骤之后使用。这对于配置通过升级添加的需要初始化的模块是必要的。

当版本为1时，这个修饰符类似于initializer，只是被标记为reinitializer的函数不能嵌套。如果在另一个上下文中调用其中一个，执行将会回滚。

请注意，版本可以以大于1的增量跳跃；这意味着如果在合同中存在多个reinitializer，以正确的顺序执行它们是开发人员或操作员的责任。

> WARNING
将版本设置为255将阻止任何未来的重新初始化。

发出一个[Initialized(version)](#initializeduint8-version)事件。

#### onlyInitializing()
修饰符#
修改以保护初始化函数，使其只能被具有[initializer()](#initializer)和[reinitializer(version)](#reinitializeruint8-version)修饰符的函数直接或间接调用。

#### _disableInitializers()
内部# 
锁定合约，阻止任何未来的重新初始化。这不能是初始化函数的一部分。在合约的构造函数中调用此函数将阻止该合约被初始化或重新初始化为任何版本。建议将此函数用于锁定通过代理调用的实现合约。

在第一次成功执行时，会发出一个[Initialized(version)](#initializeduint8-version)事件。

#### _getInitializedVersion() → uint8
内部# 
返回已初始化的最高版本。请参阅[reinitializer](#reinitializeruint8-version).

#### _isInitializing() → bool
内部# 
如果合同当前正在初始化，则返回true。请参阅[onlyInitializing()](#onlyinitializing).

#### Initialized(uint8 version)
事件#
当合同已初始化或重新初始化时触发。

### UUPSUpgradeable
```
import "@openzeppelin/contracts/proxy/utils/UUPSUpgradeable.sol";
```

为UUPS代理设计的可升级性机制。这里包含的函数可以在将该合约设置为代理背后的实现时，执行[ERC1967Proxy](#erc1967proxy)的升级。

安全机制确保升级不会意外关闭升级能力，尽管如果升级保留了升级能力但移除了安全机制（例如通过替换UUPSUpgradeable为自定义的升级实现），则此风险将重新出现。

必须重写[_authorizeUpgrade](#_authorizeupgradeaddress-newimplementation)函数以包括对升级机制的访问限制。

*自v4.1版本起可用。*

**MODIFIERS**
[onlyProxy()](#onlyproxy)
[notDelegated()](#notdelegated)

**FUNCTIONS**
[proxiableUUID()](#proxiableuuid-→-bytes32)
[upgradeTo(newImplementation)](#upgradetoaddress-newimplementation-1)
[upgradeToAndCall(newImplementation, data)](#upgradetoandcalladdress-newimplementation-bytes-data)
[_authorizeUpgrade(newImplementation)](#_authorizeupgradeaddress-newimplementation)

ERC1967UPGRADE
[_getImplementation()](#_getimplementation-→-address)
[_upgradeTo(newImplementation)](#_upgradetoaddress-newimplementation)
[_upgradeToAndCall(newImplementation, data, forceCall)](#_upgradetoandcalladdress-newimplementation-bytes-data-bool-forcecall)
[_upgradeToAndCallUUPS(newImplementation, data, forceCall)](#_upgradetoandcalluupsaddress-newimplementation-bytes-data-bool-forcecall)
[_getAdmin()](#_getadmin-→-address)
[_changeAdmin(newAdmin)](#_changeadminaddress-newadmin)
[_getBeacon()](#_getbeacon-→-address)
[_upgradeBeaconToAndCall(newBeacon, data, forceCall)](#_upgradebeacontoandcalladdress-newbeacon-bytes-data-bool-forcecall)

**EVENTS**

IERC1967
[Upgraded(implementation)](./Interfaces.md)
[AdminChanged(previousAdmin, newAdmin)](./Interfaces.md)
[BeaconUpgraded(beacon)](./Interfaces.md)

#### onlyProxy()
修饰符#
检查执行是否通过delegatecall调用，并且执行上下文是指向自身的实现（根据ERC1967定义）。这仅适用于使用当前合约作为其实现的UUPS和透明代理。通过ERC1167最小代理（克隆）执行函数通常不会通过此测试，但不能保证一定会失败。

#### notDelegated()
修饰符#
检查执行是否未通过委托调用进行。这允许一个函数可以在实现合约上调用，但不能通过代理调用。

#### proxiableUUID() → bytes32
外部#
实现ERC1822 [proxiableUUID](#proxiableuuid-→-bytes32)函数。该函数返回实现使用的存储槽。在执行升级时，它用于验证实现的兼容性。

> IMPORTANT
指向proxiable合约的代理本身不应被视为可代理的，因为这会导致代理升级到自身并耗尽所有的gas。因此，如果通过代理调用此函数，它是关键的，该函数会触发revert。这是通过notDelegated修饰符来保证的。

#### upgradeTo(address newImplementation)
公开#
升级代理的实现到newImplementation。

调用[_authorizeUpgrade](#_authorizeupgradeaddress-newimplementation)。
发出一个[Upgraded](#upgradedaddress-indexed-implementation)事件。

#### upgradeToAndCall(address newImplementation, bytes data)
公开#
将代理的实现升级为newImplementation，并随后执行data中编码的函数调用。

调用[_authorizeUpgrade](#_authorizeupgradeaddress-newimplementation)函数。

发出一个[Upgraded](#upgradedaddress-indexed-implementation)事件。

#### _authorizeUpgrade(address newImplementation)
内部#
当msg.sender没有权限升级合约时，应该回滚的函数。由[upgradeTo](#upgradetoaddress-newimplementation-1)和[upgradeToAndCall](#upgradetoandcalladdress-newimplementation-bytes-data)调用。

通常，这个函数会使用一个[access control](./Access.md)修饰符，比如[Ownable.onlyOwner](./Access.md#ownable)。

```
function _authorizeUpgrade(address) internal override onlyOwner {}
```

