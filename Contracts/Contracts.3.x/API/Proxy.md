# Proxies
这是一个低级别的合约集，实现了不同的代理模式，包括可升级和不可升级的模式。如果想深入了解这个模式，请参阅*代理升级模式*页面。

抽象*代理*合约实现了核心委托功能。如果我们提供的具体代理不适合您的需求，我们鼓励在这个基础合约上进行构建，因为它包含了一个可能难以正确实现的汇编块。

Upgradeability 是在 *UpgradeableProxy *合约中实现的，尽管它只提供了一个内部的升级接口。为了向管理员提供外部暴露的升级接口，我们提供了 *TransparentUpgradeableProxy*。这两个合约都使用了 [EIP1967](https://eips.ethereum.org/EIPS/eip-1967) 中指定的存储槽来避免与代理后面的实现合约的存储冲突。

*Beacon*提供了一种替代的升级机制。这种模式由Dharma推广，允许在单个事务中将多个代理升级到不同的实现。在这种模式中，代理合约不像*UpgradeableProxy*那样在存储中保存实现地址，而是保存*UpgradeableBeacon*合约的地址，实际的实现地址存储在其中并从中检索。改变实现合约地址的升级操作被发送到beacon而不是代理合约，所有遵循该beacon的代理都会自动升级。

*Clones*库提供了一种以廉价方式部署最小化的不可升级代理。这对于需要部署许多相同合约实例的应用程序非常有用（例如每个用户一个实例，或每个任务一个实例）。这些实例旨在部署成本低廉，并且调用成本低廉。缺点是它们无法升级。

> CAUTION
正确且安全地使用可升级代理是一项困难的任务，需要对代理模式、Solidity和EVM有深入的了解。除非您需要很多低级别的控制，我们建议使用*OpenZeppelin Truffle和Buidler的升级插件*。

## 核心

### 代理
这个抽象合约提供了一个回退函数，该函数使用EVM指令delegatecall将所有调用委托给另一个合约。我们将第二个合约称为代理背后的实现，并且必须通过重写虚拟的*_implementation*函数来指定它。

此外，可以通过*_fallback*函数手动触发对实现的委托，或者通过*_delegate*函数委托给其他合约。

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

此函数不会返回到其内部调用点，而是直接返回给外部调用者。

#### _implementation() → address
内部#
这是一个虚函数，应该被重写，以便返回后备函数和*_fallback*应该委派的地址。

#### _fallback()
内部#
将当前调用委托给_implementation()返回的地址。

该函数不会返回到其内部调用位置，而是直接返回给外部调用者。

#### fallback()
外部#
回退函数将调用委托给由_implementation()返回的地址。如果合约中没有其他函数与调用数据匹配，则会运行该函数。

#### receive()
外部#
回退函数将调用委托给_implementation()返回的地址。如果调用数据为空，则会运行。

#### _beforeFallback()
外部#
在回退到实现之前调用的 hooks 。可以作为手动回退调用的一部分，也可以作为Solidity回退或接收函数的一部分。

如果被重写，则应调用super._beforeFallback()。

### UpgradeableProxy
该合约实现了一个可升级的代理。它之所以可升级，是因为调用被委托给一个可以更改的实现地址。该地址存储在存储器中，位置由[EIP1967](https://eips.ethereum.org/EIPS/eip-1967)指定，以避免与代理后面的实现的存储布局冲突。

升级功能仅通过*_upgradeTo*在内部提供。对于可外部升级的代理，请参见*TransparentUpgradeableProxy*。

**FUNCTIONS**
constructor(_logic, _data)

_implementation()

_upgradeTo(newImplementation)

PROXY
_delegate(implementation)

_fallback()

fallback()

receive()

_beforeFallback()

**EVENTS**
Upgraded(implementation)

#### constructor(address _logic, bytes _data)
公开#
使用_logic参数初始化可升级代理的初始实现。

如果_data参数非空，则将其用作对_logic进行委托调用的数据。这通常是一个编码的函数调用，并允许像Solidity构造函数一样初始化代理的存储。

#### _implementation() → address impl
内部#
返回当前实现地址。

#### _upgradeTo(address newImplementation)
内部#
将代理升级到新的实现。

发出一个升级事件。

#### Upgraded(address implementation)
事件#
当实现被升级时发出。

### TransparentUpgradeableProxy
该合约实现了一个可由管理员升级的代理。

为了避免可能被用于攻击的*代理选择器冲突*，该合约使用了*透明代理模式*。这种模式意味着两件事情相辅相成：

1. 如果除管理员外的任何账户调用代理，即使该调用与代理本身公开的管理员函数匹配，调用也将被转发到实现。

2. 如果管理员调用代理，它可以访问管理员函数，但它的调用永远不会被转发到实现。如果管理员尝试调用实现上的函数，将会失败，并显示错误信息“管理员无法回退到代理目标”。

这些属性意味着管理员账户只能用于管理员操作，如升级代理或更改管理员，因此最好使用专用账户，不要用于其他任何操作。这样可以避免在尝试调用代理实现的函数时突然出现错误的麻烦。

我们建议专用账户是*ProxyAdmin*合约的一个实例。如果以这种方式设置，您应该将ProxyAdmin实例视为代理的真正管理界面。

**MODIFIERS**
ifAdmin()

**FUNCTIONS**
constructor(_logic, admin_, _data)

admin()

implementation()

changeAdmin(newAdmin)

upgradeTo(newImplementation)

upgradeToAndCall(newImplementation, data)

_admin()

_beforeFallback()

UPGRADEABLEPROXY
_implementation()

_upgradeTo(newImplementation)

PROXY
_delegate(implementation)

_fallback()

fallback()

receive()

**EVENTS**
AdminChanged(previousAdmin, newAdmin)

#### ifAdmin()
修饰符#
除非发送者是管理员，否则将调用委托给实现的内部使用的修饰符。

#### constructor(address _logic, address admin_, bytes _data)
公开#
通过_admin管理的升级代理进行初始化，由_logic的实现支持，并可选择使用*UpgradeableProxy.constructor*中解释的_data进行初始化。

#### admin() → address admin_
外部#
返回当前管理员。

> NOTE
只有管理员才能调用此函数。请参阅*ProxyAdmin.getProxyAdmin*。

> TIP
要获取此值，客户端可以使用[eth_getStorageAt](https://eth.wiki/json-rpc/API#eth_getstorageat) RPC调用直接从下面显示的存储槽（由EIP1967指定）读取。0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103

#### implementation() → address implementation_
外部#
返回当前的实现。

> NOTE
只有管理员才能调用此函数。请参阅*ProxyAdmin.getProxyImplementation*。

> TIP
要获取此值，客户端可以使用[eth_getStorageAt](https://eth.wiki/json-rpc/API#eth_getstorageat) RPC调用直接从下面显示的存储槽中读取（由EIP1967指定）。 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc

#### changeAdmin(address newAdmin)
外部#
更改代理的管理员。

触发*AdminChanged*事件。

> NOTE
只有管理员可以调用此函数。请参阅*ProxyAdmin.changeProxyAdmin*。

#### upgradeTo(address newImplementation)
外部#
升级代理的实现。

> NOTE
只有管理员可以调用此函数。请参阅*ProxyAdmin.upgrade*。

#### upgradeToAndCall(address newImplementation, bytes data)
外部#
升级代理的实现，并根据数据中的指示调用新实现的函数，该数据应为一个编码的函数调用。这对于在被代理的合约中初始化新的存储变量非常有用。

> NOTE
只有管理员可以调用此函数。请参阅ProxyAdmin.upgradeAndCall。

#### _admin() → address adm
内部#
返回当前管理员。

#### _beforeFallback()
内部#
确保管理员不能访问回退函数。请参阅*Proxy._beforeFallback*。

#### AdminChanged(address previousAdmin, address newAdmin)
事件#
当管理员账户发生变化时发出。

## Beacon

### BeaconProxy
此合约实现了一个代理，每次调用从*UpgradeableBeacon*获取实现地址。

beacon地址存储在存储槽uint256(keccak256('eip1967.proxy.beacon')) - 1中，以便不与代理后面的实现的存储布局冲突。

*从v3.4开始可用。*

**FUNCTIONS**
constructor(beacon, data)

_beacon()

_implementation()

_setBeacon(beacon, data)

PROXY
_delegate(implementation)

_fallback()

fallback()

receive()

_beforeFallback()

#### constructor(address beacon, bytes data)
公开#
使用beacon初始化代理。

如果数据非空，则将其用作对由beacon返回的实现进行委托调用的数据。这通常是一个编码的函数调用，并允许像Solidity构造函数一样初始化代理的存储。

要求：
* beacon必须是一个具有IBeacon接口的合约。

#### _beacon() → address beacon
内部#
返回当前信标地址。

#### _implementation() → address
内部#
返回关联信标的当前实现地址。

#### _setBeacon(address beacon, bytes data)
内部#
更改代理以使用新的信标。

如果数据非空，则将其用作委托调用信标返回的实现的数据。

要求：

* 信标必须是一个合约。

* 信标返回的实现必须是一个合约。

### IBeacon
这是*BeaconProxy*对其信标期望的接口。

**FUNCTIONS**
implementation()

#### implementation() → address
外部#
必须返回一个可以用作委托调用目标的地址。

*BeaconProxy*将检查该地址是否为合约。

### UpgradeableBeacon
该合约与一个或多个*BeaconProxy*实例一起使用，以确定它们的实现合约，该合约将委托所有函数调用。

所有者可以更改beacon指向的实现，从而升级使用该beacon的代理。

**FUNCTIONS**
constructor(implementation_)

implementation()

upgradeTo(newImplementation)

OWNABLE
owner()

renounceOwnership()

transferOwnership(newOwner)

**EVENTS**
Upgraded(implementation)

OWNABLE
OwnershipTransferred(previousOwner, newOwner)

#### constructor(address implementation_)
公开#
将初始实现的地址设置为部署者账户，部署者账户作为可以升级信标的所有者。

#### implementation() → address
公开#
返回当前实现的地址。

#### upgradeTo(address newImplementation)
公开#
将信标升级为新的实现。

发出*升级*事件。

要求：
* msg.sender必须是合约的所有者。

* newImplementation必须是一个合约。

#### Upgraded(address implementation)
事件#
当由beacon返回的实现发生变化时发出。

## Minimal Clones

### Clones
[EIP 1167](https://eips.ethereum.org/EIPS/eip-1167)是用于部署最小代理合约的标准，也被称为“克隆”。

    为了以不可变的方式简单且廉价地克隆合约功能，该标准指定了一个最小的字节码实现，将所有调用委托给一个已知的、固定的地址。

该库包括使用create（传统部署）或create2（带盐确定性部署）部署代理的函数。它还包括使用确定性方法部署的克隆的地址预测函数。

*自v3.4起可用。*

**FUNCTIONS**
clone(master)

cloneDeterministic(master, salt)

predictDeterministicAddress(master, salt, deployer)

predictDeterministicAddress(master, salt)

#### clone(address master) → address instance
内部#
部署并返回一个模仿主合约行为的克隆合约的地址。

该函数使用了create操作码，这个操作码不应该会引发回滚。

#### cloneDeterministic(address master, bytes32 salt) → address instance
内部#
部署并返回一个模仿主合约行为的克隆合约的地址。

该函数使用create2操作码和一个salt来确定性地部署克隆合约。如果多次使用相同的主合约和salt，将会回滚，因为克隆合约不能在相同的地址上部署两次。

#### predictDeterministicAddress(address master, bytes32 salt, address deployer) → address predicted
内部#
使用*Clones.cloneDeterministic*部署的克隆体的地址计算。

#### predictDeterministicAddress(address master, bytes32 salt) → address predicted
内部#
使用*Clones.cloneDeterministic*部署的克隆体的地址计算。

## Utilities

### Initializable
这是一个基础合约，用于帮助编写可升级的合约，或者任何将部署在代理之后的合约。由于代理合约不能有构造函数，通常将构造逻辑移动到一个外部的初始化函数中，通常称为*initializ*e。因此，需要保护这个初始化函数，使其只能被调用一次。这个合约提供的初始化器修饰符将具有此效果。

> TIP
为了避免使代理处于未初始化状态，初始化函数应尽早调用，通过将编码的函数调用作为_data参数提供给*UpgradeableProxy.constructor*。

> CAUTION
在使用继承时，必须手动注意不要两次调用父级初始化函数，或者确保所有的初始化函数都是幂等的。这不会像Solidity的构造函数那样自动进行验证。

**MODIFIERS**
initializer()

#### initializer()
修饰符#

### ProxyAdmin
这是一个辅助合约，用于将其分配为*TransparentUpgradeableProxy*的管理员。要了解为什么要使用此合约，请参阅*TransparentUpgradeableProxy*的文档。

**FUNCTIONS**
getProxyImplementation(proxy)

getProxyAdmin(proxy)

changeProxyAdmin(proxy, newAdmin)

upgrade(proxy, implementation)

upgradeAndCall(proxy, implementation, data)

OWNABLE
constructor()

owner()

renounceOwnership()

transferOwnership(newOwner)

**EVENTS**
OWNABLE
OwnershipTransferred(previousOwner, newOwner)

#### getProxyImplementation(contract TransparentUpgradeableProxy proxy) → address
公开#
返回代理的当前实现。

要求：
* 该合约必须是代理的管理员。

#### getProxyAdmin(contract TransparentUpgradeableProxy proxy) → address
公开#
返回代理的当前管理员。

要求：
* 此合约必须是代理的管理员。

#### changeProxyAdmin(contract TransparentUpgradeableProxy proxy, address newAdmin)
公开#
将proxy的管理员更改为newAdmin。

要求：
* 此合约必须是proxy的当前管理员。

#### upgrade(contract TransparentUpgradeableProxy proxy, address implementation)
公开#
将代理升级为实现。参见*TransparentUpgradeableProxy.upgradeTo*。

要求：
* 该合约必须是代理的管理员。

#### upgradeAndCall(contract TransparentUpgradeableProxy proxy, address implementation, bytes data)
公开#
将代理升级为实现，并在新实现上调用一个函数。参见*TransparentUpgradeableProxy.upgradeToAndCall*。

要求：
* 此合约必须是代理的管理员。

