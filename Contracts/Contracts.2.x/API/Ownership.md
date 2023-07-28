# Ownership
简单授权和访问控制机制的合同模块。

> TIP
对于更复杂的需求，请参见访问。

## Contracts

### Ownable
提供了一个基本的访问控制机制的合约模块，其中有一个账户（所有者）可以被授予对特定函数的独占访问权限。

该模块通过继承来使用。它将提供修饰符onlyOwner，可以应用于您的函数，以限制它们只能由所有者使用。

**MODIFIERS**
onlyOwner()

**FUNCTIONS**
constructor()

owner()

isOwner()

renounceOwnership()

transferOwnership(newOwner)

_transferOwnership(newOwner)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
OwnershipTransferred(previousOwner, newOwner)

#### onlyOwner()
修饰符#
如果被除了所有者之外的任何账户调用，则抛出异常。

#### constructor()
内部#
初始化合约，将部署者设为初始所有者。

#### owner() → address
公开#
返回当前所有者的地址。

#### isOwner() → bool
公开#
如果调用者是当前所有者，则返回true。

#### renounceOwnership()
公开#

离开合同而没有所有者。将不再能够调用只有所有者的功能。只能由当前所有者调用。

> NOTE
放弃所有权将使合同失去所有者，从而删除只有所有者才能使用的任何功能。

#### transferOwnership(address newOwner)
公开#
将合同的所有权转移到一个新账户（newOwner）。只能由当前所有者调用。

#### _transferOwnership(address newOwner)
内部#
将合同的所有权转移到一个新账户（newOwner）。

#### OwnershipTransferred(address previousOwner, address newOwner)
事件#

### Secondary
一个次要合同只能由其主要账户（创建它的账户）使用。

**MODIFIERS**
onlyPrimary()

**FUNCTIONS**
constructor()

primary()

transferPrimary(recipient)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
PrimaryTransferred(recipient)

#### onlyPrimary()
修饰符#

#### constructor()
内部#
将主要账户设置为创建次级合同的账户。

#### primary() → address
公开#

#### transferPrimary(address recipient)
公开#
将合同转让给新的主要方。

#### PrimaryTransferred(address recipient)
事件#
当主要合同发生变化时发出。