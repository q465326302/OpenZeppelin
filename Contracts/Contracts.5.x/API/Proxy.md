# Proxies
这是一套实现不同代理模式（包括可升级和不可升级）的底层合约。如果你想深入了解这种模式，请查看*代理升级模式*页面。

以下大多数代理都是基于一个抽象基础合约构建的。

* *Proxy*：实现核心委托功能的抽象合约。

为了避免与代理后面的实现合约的存储变量冲突，我们使用[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)存储槽。

* ERC1967Utils：获取和设置EIP1967中定义的存储槽的内部函数。

* ERC1967Proxy：使用EIP1967存储槽的代理。默认情况下不可升级。

有两种替代方法可以为ERC1967代理添加升级功能。他们的区别在下面的*透明代理与UUPS代理*中有所解释。

* *TransparentUpgradeableProxy*：具有内置不可变管理员和升级接口的代理。

* *UUPSUpgradeable*：要包含在实现合约中的升级机制。

> CAUTION
正确、安全地使用可升级代理是一项困难的任务，需要对代理模式、Solidity和EVM有深入的了解。除非你想要大量的低级控制，我们建议使用*OpenZeppelin升级插件*，适用于Truffle和Hardhat。

另一类代理是信标代理。这种模式由Dharma普及，允许多个代理在一次交易中升级到不同的实现。

* *BeaconProxy*：从信标合约获取其实现的代理。

* *UpgradeableBeacon*：具有内置管理员的信标合约，可以升级指向它的*BeaconProxy*。

在这种模式中，代理合约不像ERC1967代理那样在存储中持有实现地址。相反，地址存储在单独的信标合约中。升级操作发送到信标，而不是代理合约，所有跟随该信标的代理都会自动升级。

在升级性之外，代理也可以用来制作便宜的合约克隆，比如由链上工厂合约创建的许多相同合约的实例。这些实例旨在既便宜的部署，又便宜的调用。

* *Clones*：可以部署便宜的最小非升级代理的库。

## 透明代理与UUPS代理
OpenZeppelin中原始的代理遵循*透明代理模式*。虽然我们仍然提供这种模式，但我们现在推荐使用UUPS代理，它们既轻量又多功能。UUPS的名字来自[EIP1822](https://eips.ethereum.org/EIPS/eip-1822)，它首次记录了这种模式。

虽然这两种代理在升级接口上是相同的，但在UUPS代理中，升级由实现处理，并最终可以被移除。另一方面，透明代理在代理本身中包含升级和管理员逻辑。这意味着*TransparentUpgradeableProxy*的部署成本比UUPS代理可能的成本更高。

UUPS代理使用*ERC1967Proxy*实现。注意，这个代理本身并不可升级。实现的角色是在合约的逻辑旁边，包含所有必要的代码，以更新存储在代理存储空间特定槽中的实现地址。这就是*UUPSUpgradeable*合约的作用。从它继承（并覆盖*_authorizeUpgrade*函数，使用相关的访问控制机制）将把你的合约变成一个符合UUPS的实现。

请注意，由于两个代理使用相同的存储槽来存储实现地址，使用符合UUPS的实现与*TransparentUpgradeableProxy*可能会允许非管理员执行升级操作。

默认情况下，*UUPSUpgradeable*中包含的升级功能包含一个安全机制，该机制将阻止任何升级到非UUPS符合的实现。这防止了升级到一个不包含必要升级机制的实现合约，因为这将永远锁定代理的可升级性。这个安全机制可以通过以下任何一种方式绕过：

* 在实现中添加一个标志机制，当触发时将禁用升级功能。

* 升级到一个包含升级机制但没有额外安全检查的实现，然后再升级到另一个没有升级机制的实现。

当前的这种安全机制的实现使用[EIP1822](https://eips.ethereum.org/EIPS/eip-1822)来检测实现使用的存储槽。以前的实现，现在已经被弃用，依赖于回滚检查。可以从使用旧机制的合约升级到新的合约。然而，反过来则不可能，因为旧的实现（在4.5版本之前）并未包含ERC1822接口。

## 核心

### [Proxy](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/proxy/Proxy.sol)
```
import "@openzeppelin/contracts/proxy/Proxy.sol";
```
这个抽象合约提供了一个回退函数，该函数通过EVM指令delegatecall将所有调用委托给另一个合约。我们将第二个合约称为代理背后的实现，它必须通过覆盖虚拟*_implementation*函数来指定。

此外，可以通过*_fallback*函数手动触发对实现的委托，或通过*_delegate*函数触发对不同合约的委托。

委托调用的成功和返回数据将返回给代理的调用者。

**FUNCTIONS**
_delegate(implementation)

_implementation()

_fallback()

fallback()

#### _delegate(address implementation)
internal#
将当前调用委托给实现。

这个函数不会返回到其内部调用站点，它将直接返回到外部调用者。

#### _implementation() → address
internal#
这是一个虚拟函数，应该被重写以返回回退函数和*_fallback*应该委托的地址。

#### _fallback()
internal#
此函数将当前调用委托给_implementation()返回的地址。

此函数不会返回到其内部调用站点，它将直接返回给外部调用者。

#### fallback()
external#
如果合约中没有其他函数匹配调用数据，将运行回退函数，该函数将调用委托给_implementation()返回的地址。

## ERC1967

### [ERC1967Proxy](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/proxy/ERC1967/ERC1967Proxy.sol)
```
import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";
```

此合约实现了一个可升级的代理。之所以可升级，是因为调用被委托给可以更改的实现地址。这个地址存储在[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)指定的位置，以避免与代理背后的实现的存储布局冲突。

**FUNCTIONS**
constructor(implementation, _data)

_implementation()

PROXY
_delegate(implementation)

_fallback()

fallback()

#### constructor(address implementation, bytes _data)
public#
使用由实现指定的初始实现初始化可升级的代理。

如果_data非空，它将被用作到实现的委托调用中的数据。这通常会是一个编码的函数调用，允许像Solidity构造函数一样初始化代理的存储。

要求：
* 如果数据为空，msg.value必须为零。

#### _implementation() → address
internal#
返回当前实现地址。

要获取此值，客户端可以使用[eth_getStorageAt](https://eth.wiki/json-rpc/API#eth_getstorageat) RPC调用直接从下面显示的存储槽（由EIP1967指定）读取。0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc

### [ERC1967Utils](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/proxy/ERC1967/ERC1967Utils.sol)
```
import "@openzeppelin/contracts/proxy/ERC1967/ERC1967Utils.sol";
```

此抽象合约为[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)插槽提供了获取器和事件发射更新功能。

**FUNCTIONS**
getImplementation()

upgradeToAndCall(newImplementation, data)

getAdmin()

changeAdmin(newAdmin)

getBeacon()

upgradeBeaconToAndCall(newBeacon, data)

**EVENTS**
Upgraded(implementation)

AdminChanged(previousAdmin, newAdmin)

BeaconUpgraded(beacon)

**ERRORS**
ERC1967InvalidImplementation(implementation)

ERC1967InvalidAdmin(admin)

ERC1967InvalidBeacon(beacon)

ERC1967NonPayable()

#### getImplementation() → address
internal#
返回当前实现地址。

#### upgradeToAndCall(address newImplementation, bytes data)
internal#
如果数据非空，则执行实现升级并进行额外的设置调用。只有在执行设置调用时，此功能才可支付，否则将拒绝msg.value以避免合同中的价值卡住。

触发一个*IERC1967.Upgraded*事件。

#### getAdmin() → address
internal#
返回当前管理员。

客户端可以直接从下面显示的存储槽（由EIP1967指定）读取此值，使用[eth_getStorageAt](https://eth.wiki/json-rpc/API#eth_getstorageat) RPC调用。0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103

#### changeAdmin(address newAdmin)
internal#
更改代理的管理员。

触发一个*IERC1967.AdminChanged*事件。

#### getBeacon() → address
internal#
返回当前信标。

#### upgradeBeaconToAndCall(address newBeacon, bytes data)
internal#
如果数据非空，更改信标并触发设置调用。只有在执行设置调用时，此功能才可支付，否则将拒绝msg.value以避免合同中的价值卡住。

发出一个*IERC1967.BeaconUpgraded*事件。

自v5以来，调用此函数对*BeaconProxy*的实例没有影响，因为它使用不可变的信标，而不查看ERC-1967信标插槽的值以提高效率。

#### Upgraded(address indexed implementation)
event#
当实施升级时发出。

#### AdminChanged(address previousAdmin, address newAdmin)
event#
当管理员账户发生变化时发出。

#### BeaconUpgraded(address indexed beacon)
event#
当信标改变时发出。

#### ERC1967InvalidImplementation(address implementation)
error#
代理的实现是无效的。

#### ERC1967InvalidAdmin(address admin)
error#
代理服务器的管理员无效。

#### ERC1967InvalidBeacon(address beacon)
error#
代理服务器的信标无效。

#### ERC1967NonPayable()
error#
一个升级函数看到的msg.value > 0可能会丢失。

## Transparent Proxy

### [TransparentUpgradeableProxy](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/proxy/transparent/TransparentUpgradeableProxy.sol)
```
import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
```

该合约实现了一个代理，该代理可以通过关联的*ProxyAdmin*实例进行升级。

为了避免*代理选择器冲突*，这可能被用于攻击，该合约使用了*透明代理模式*。这种模式意味着两个相互关联的事情：

1. 如果除管理员账户外的任何账户调用代理，即使该调用与代理本身公开的*ITransparentUpgradeableProxy.upgradeToAndCall*函数匹配，调用也会被转发到实现。

2. 如果管理员调用代理，它可以调用upgradeToAndCall函数，但任何其他调用都不会被转发到实现。如果管理员试图在实现上调用函数，它将失败，并显示一个错误，指出代理管理员无法回退到目标实现。

这些属性意味着管理员账户只能用于升级代理，所以最好是一个专用账户，不用于任何其他事情。这将避免由于试图从代理实现调用函数时突然出现错误的麻烦。因此，代理部署了一个ProxyAdmin实例，并只允许通过它进行升级。你应该把*ProxyAdmin*实例看作是代理的管理接口，包括改变谁可以触发升级的能力。

> NOTE
这个代理的真正接口是在ITransparentUpgradeableProxy中定义的。这个合约并没有从那个接口继承，而是通过在_fallback中使用自定义的分派机制隐式地实现了upgradeToAndCall。因此，编译器不会为这个合约产生ABI。这是为了完全实现透明性，而不解码由代理和实现之间的选择器冲突引起的反转。

> NOTE
这个代理故意不从*Context*继承。这个合约的*ProxyAdmin*不会以任何方式发送元交易，任何其他元交易设置应在实现合约中进行。

> IMPORTANT
这个合约通过在构造过程中只设置管理员为不可变变量，避免了不必要的存储读取，防止了之后的任何改变。然而，定义在ERC-1967中的管理员插槽仍然可以被这个代理指向的实现逻辑覆盖。在这种情况下，合约可能会处于不希望的状态，其中管理员插槽与实际管理员不同。

> WARNING
>不建议扩展此合约以添加额外的外部函数。如果你这样做，由于上面的注意事项，编译器将不会检查是否有选择器冲突。任何新函数与*ITransparentUpgradeableProxy*中声明的函数之间的选择器冲突将以新函数为优先。这可能会使upgradeToAndCall函数无法访问，阻止升级性和透明性。

**FUNCTIONS**
constructor(_logic, initialOwner, _data)

_proxyAdmin()

_fallback()

ERC1967PROXY
_implementation()

PROXY
_delegate(implementation)

fallback()

**ERRORS**
ProxyDeniedAdminAccess()

#### constructor(address _logic, address initialOwner, bytes _data)
public#
初始化一个由*ProxyAdmin*实例管理的可升级代理，该代理由初始所有者initialOwner支持，后台实现在_logic中，并且可以选择使用_data进行初始化，如*ERC1967Proxy.constructor*中所解释的。

#### _proxyAdmin() → address
internal#
返回此代理的管理员。

#### _fallback()
internal#
如果呼叫者是管理员，则内部处理呼叫，否则透明地回退到代理行为。

#### ProxyDeniedAdminAccess()
error#
代理调用者是当前管理员，不能回退到代理目标。

### [ProxyAdmin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/proxy/transparent/ProxyAdmin.sol)
```
import "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";
```

这是一个辅助合约，旨在被指定为*TransparentUpgradeableProxy*的管理员。关于为什么要使用这个，可以参考*TransparentUpgradeableProxy*的文档说明。

**FUNCTIONS**
constructor(initialOwner)

upgradeAndCall(proxy, implementation, data)

UPGRADE_INTERFACE_VERSION()

OWNABLE
owner()

_checkOwner()

renounceOwnership()

transferOwnership(newOwner)

_transferOwnership(newOwner)

**EVENTS**
OWNABLE
OwnershipTransferred(previousOwner, newOwner)

**ERRORS**
OWNABLE
OwnableUnauthorizedAccount(account)

OwnableInvalidOwner(owner)

#### constructor(address initialOwner)
public#
设置可以执行升级的初始所有者。

#### upgradeAndCall(contract ITransparentUpgradeableProxy proxy, address implementation, bytes data)
public#
将代理升级到实现并在新实现上调用函数。请参阅*TransparentUpgradeableProxy._dispatchUpgradeToAndCall*。

要求：
* 此合约必须是代理的管理员。

* 如果数据为空，msg.value必须为零。

#### UPGRADE_INTERFACE_VERSION() → string
public#
合约升级接口的版本。如果缺少这个getter，upgrade(address)和upgradeAndCall(address,bytes)都存在，如果不应调用任何函数，必须使用upgradeTo，而如果第二个参数是空字节字符串，upgradeAndCall将调用接收函数。如果getter返回"5.0.0"，只有upgradeAndCall(address,bytes)存在，如果不应调用任何函数，第二个参数必须是空字节字符串，这使得在升级过程中无法调用接收函数。

## Beacon