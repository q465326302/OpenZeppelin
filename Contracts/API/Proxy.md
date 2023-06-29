# Proxies
这是一组低级别的合约，实现了不同的代理模式，包括具有和不具有升级功能的代理模式。要详细了解此模式，请参阅*Proxy Upgrade Pattern*页面。

下面的大多数代理都是基于一个抽象基础合约构建的。

* *Proxy*：实现核心委托功能的抽象合约。

为了避免与代理后面的实现合约的存储变量冲突，我们使用[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)存储槽。

* *ERC1967Upgrade*：用于获取和设置EIP1967定义的存储槽的内部函数。

* *ERC1967Proxy*：使用EIP1967存储槽的代理。默认情况下不可升级。

有两种不同的方法可以向ERC1967代理添加升级功能。它们的区别在下面的*Transparent vs UUPS Proxies*中解释。

* *TransparentUpgradeableProxy*：具有内置管理员和升级接口的代理。

* *UUPSUpgradeable*：一种可以包含在实现合约中的可升级性机制。

> CAUTION
正确和安全地使用可升级代理是一项困难的任务，需要对代理模式、Solidity和EVM有深入的了解。除非您需要很多低级别的控制，否则我们建议使用*OpenZeppelin Upgrades*插件来进行Truffle和Hardhat的升级。

另一组代理是beacon代理。这种模式由Dharma推广，允许多个代理在单个事务中升级到不同的实现。

* *BeaconProxy*：从beacon合约检索其实现的代理。

* *UpgradeableBeacon*：带有内置管理员的beacon合约，可以升级指向它的*BeaconProxy*。

在这种模式中，代理合约不像ERC1967代理那样在存储中保存实现地址。相反，地址存储在一个单独的beacon合约中。升级操作发送到beacon而不是代理合约，并且所有遵循该beacon的代理都会自动升级。

在不可升级的情况下，代理还可以用于创建廉价的合约克隆，例如由一个在链工厂合约创建的多个相同合约实例。这些实例旨在部署廉价，并且调用廉价。

* *Clones*：一个可以部署廉价的最小非升级代理的库。

## 透明代理和UUPS代理之间的区别
OpenZeppelin中最初提供的代理遵循了[透明代理模式](https://blog.openzeppelin.com/the-transparent-proxy-pattern/)。虽然仍然提供该模式，但我们现在的建议正在转向UUPS代理，这些代理既轻量又灵活。UUPS的名称来自于[EIP1822](https://eips.ethereum.org/EIPS/eip-1822)，该规范首次记录了该模式。

虽然这两种代理都共享相同的升级接口，但在UUPS代理中，升级由实现处理，并且最终可以被移除。而透明代理则在代理本身中包含了升级和管理员逻辑。这意味着*TransparentUpgradeableProxy*的部署成本比UUPS代理更高。

UUPS代理是使用*ERC1967Proxy*实现的。请注意，该代理本身不可升级。实现的角色是在合约的逻辑之外，包含所有必要的代码来更新存储在代理存储空间的特定槽位中的实现地址。这就是*UUPSUpgradeable*合约的作用。继承它（并使用相关的访问控制机制覆盖 *_authorizeUpgrade* 函数）将使您的合约成为符合UUPS的实现。

请注意，由于两种代理都使用相同的存储槽位来存储实现地址，使用符合UUPS的实现与*TransparentUpgradeableProxy*结合使用可能会允许非管理员执行升级操作。

默认情况下，*UUPSUpgradeable*中包含的升级功能包含一种安全机制，可以防止对非符合UUPS的实现进行升级。这可以防止对不包含必要升级机制的实现合约进行升级，因为这将永远锁定代理的可升级性。这种安全机制可以通过以下任一方式绕过：

* 在实现中添加一个标志机制，在触发时禁用升级函数。

* 升级到具有升级机制但没有附加安全检查的实现，然后再次升级到另一个没有升级机制的实现。

当前这种安全机制的实现使用了[EIP1822](https://eips.ethereum.org/EIPS/eip-1822)来检测实现使用的存储槽位。以前的实现，现在已经弃用，依赖于回滚检查。可以从使用旧机制的合约升级到新的合约。但反过来则不可能，因为旧的实现（版本4.5之前）不包含ERC1822接口。

## 核心

### Proxy
```
import "@openzeppelin/contracts/proxy/Proxy.sol";
```
这个抽象合约提供了一个回退函数，它使用EVM指令delegatecall将所有调用委托给另一个合约。我们将第二个合约称为代理后面的实现，需要通过覆盖虚拟函数*_implementation*来指定它。

此外，可以通过*_fallback*函数手动触发对实现的委托，或者通过*_delegate*函数委托给另一个合约。

委托调用的成功和返回数据将返回给代理的调用者。

**FUNCTIONS**
_delegate(implementation)
_implementation()
_fallback()
fallback()
receive()
_beforeFallback()

#### _delegate(address implementation)
内部#
将当前调用委托给实现。

该函数不会返回到其内部调用点，而是直接返回给外部调用者。

#### _implementation() → address
内部#
这是一个应该被覆盖的虚函数，它应该返回fallback函数和*_fallback*函数委托的地址。

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
在回退到实现之前调用的钩子。可以作为手动回退调用的一部分，也可以作为Solidity回退或接收函数的一部分。

如果被重写，应该调用super._beforeFallback()。

## ERC1967

### ERC1967Proxy
```
import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";
```
该合约实现了一个可升级的代理。它是可升级的，因为调用被委托给一个可以更改的实现地址。该地址存储在存储器中，位置由[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)指定，以避免与代理背后的实现的存储布局冲突。

**FUNCTIONS**
constructor(_logic, _data)
_implementation()

ERC1967UPGRADE
_getImplementation()
_upgradeTo(newImplementation)
_upgradeToAndCall(newImplementation, data, forceCall)
_upgradeToAndCallUUPS(newImplementation, data, forceCall)
_getAdmin()
_changeAdmin(newAdmin)
_getBeacon()
_upgradeBeaconToAndCall(newBeacon, data, forceCall)

PROXY
_delegate(implementation)
_fallback()
fallback()
receive()
_beforeFallback()

**EVENTS**

IERC1967
Upgraded(implementation)
AdminChanged(previousAdmin, newAdmin)
BeaconUpgraded(beacon)

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
_getImplementation()
_upgradeTo(newImplementation)
_upgradeToAndCall(newImplementation, data, forceCall)
_upgradeToAndCallUUPS(newImplementation, data, forceCall)
_getAdmin()
_changeAdmin(newAdmin)
_getBeacon()
_upgradeBeaconToAndCall(newBeacon, data, forceCall)

**EVENTS**

IERC1967
Upgraded(implementation)
AdminChanged(previousAdmin, newAdmin)
BeaconUpgraded(beacon)

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
进行信标升级并进行附加的设置调用。注意：这将升级信标的地址，而不是升级信标中包含的实现（有关此操作，请参阅 *UpgradeableBeacon._setImplementation*）。

发出 {BeaconUpgraded} 事件。

## Transparent Proxy

### TransparentUpgradeableProxy
```
import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
```
该合约实现了一个可由管理员升级的代理。

为了避免潜在的[代理选择器冲突](https://medium.com/nomic-labs-blog/malicious-backdoors-in-ethereum-proxies-62629adf3357)，该合约采用*透明代理模式*。该模式意味着两个相辅相成的事情：

1. 如果除管理员之外的任何账户调用代理，即使该调用与代理本身公开的管理员函数相匹配，调用也会被转发到实现上。

2. 如果管理员调用代理，它可以访问管理员函数，但它的调用将永远不会被转发到实现上。如果管理员尝试调用实现上的函数，将会失败并显示错误信息"admin cannot fallback to proxy target"。

这些属性意味着管理员账户只能用于管理员操作，如升级代理或更改管理员，所以最好是一个专用账户，不要用于其他任何用途。这样可以避免在尝试从代理实现中调用函数时出现突然的错误，从而带来麻烦。

我们建议专用账户是*ProxyAdmin*合约的一个实例。如果以这种方式设置，您应该将ProxyAdmin实例视为代理的真正管理接口。

> NOTE
该代理的真正接口是在ITransparentUpgradeableProxy中定义的接口。该合约不继承该接口，而是在_fallback中使用自定义调度机制隐式实现管理员函数。因此，编译器不会为该合约生成ABI。这是为了完全实现透明性，避免由于代理和实现之间的选择器冲突而解码失败。

> WARNING
不建议扩展该合约以添加其他外部函数。如果这样做，由于上述说明，编译器将不会检查是否存在选择器冲突。任何新函数与*ITransparentUpgradeableProxy*中声明的函数之间的选择器冲突将优先选择新函数。这可能导致无法访问管理员操作，从而阻止升级能力。透明性也可能受到损害。

**MODIFIERS**
ifAdmin()

**FUNCTIONS**
constructor(_logic, admin_, _data)
_fallback()
_admin()

ERC1967PROXY
_implementation()

ERC1967UPGRADE
_getImplementation()
_upgradeTo(newImplementation)
_upgradeToAndCall(newImplementation, data, forceCall)
_upgradeToAndCallUUPS(newImplementation, data, forceCall)
_getAdmin()
_changeAdmin(newAdmin)
_getBeacon()
_upgradeBeaconToAndCall(newBeacon, data, forceCall)

PROXY
_delegate(implementation)
fallback()
receive()
_beforeFallback()

**EVENTS**
IERC1967
Upgraded(implementation)
AdminChanged(previousAdmin, newAdmin)
BeaconUpgraded(beacon)

#### ifAdmin()
修饰符# 
用于内部使用的修饰符，除非发送者是管理员，否则将调用委托给实现。

> CAUTION
该修饰符已弃用，因为如果修改的函数具有参数，并且实现提供具有相同选择器的函数，则可能会导致问题。

#### constructor(address _logic, address admin_, bytes _data)
公开#
通过_admin初始化一个可升级的代理，由_logic的实现支持，并可选择使用*ERC1967Proxy.constructor*中解释的_data进行初始化。

#### _fallback()
内部#
如果调用者是管理员进程，则在内部处理呼叫，否则透明地回退到代理行为。

#### _admin() → address
内部#
返回当前管理员。

此函数已弃用。请使用*ERC1967Upgrade._getAdmin*代替。

### ProxyAdmin
```
import "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";
```
这是一个辅助合约，用于将其分配为*TransparentUpgradeableProxy*的管理员。有关为什么要使用它的解释，请参阅*TransparentUpgradeableProxy*的文档。

**FUNCTIONS**
getProxyImplementation(proxy)
getProxyAdmin(proxy)
changeProxyAdmin(proxy, newAdmin)
upgrade(proxy, implementation)
upgradeAndCall(proxy, implementation, data)

OWNABLE
owner()
_checkOwner()
renounceOwnership()
transferOwnership(newOwner)
_transferOwnership(newOwner)

**EVENTS**
OWNABLE
OwnershipTransferred(previousOwner, newOwner)

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
此合约实现了一个代理，它从*UpgradeableBeacon*获取每个调用的实现地址。

beacon地址存储在存储槽uint256(keccak256('eip1967.proxy.beacon')) - 1中，以避免与代理后面的实现的存储布局冲突。

*自v3.4起可用。*

*FUNCTIONS*
constructor(beacon, data)
_beacon()
_implementation()
_setBeacon(beacon, data)

ERC1967UPGRADE
_getImplementation()
_upgradeTo(newImplementation)
_upgradeToAndCall(newImplementation, data, forceCall)
_upgradeToAndCallUUPS(newImplementation, data, forceCall)
_getAdmin()
_changeAdmin(newAdmin)
_getBeacon()
_upgradeBeaconToAndCall(newBeacon, data, forceCall)

PROXY
_delegate(implementation)
_fallback()
fallback()
receive()
_beforeFallback()

**EVENTS**
IERC1967
Upgraded(implementation)
AdminChanged(previousAdmin, newAdmin)
BeaconUpgraded(beacon)

#### constructor(address beacon, bytes data)
公开#
