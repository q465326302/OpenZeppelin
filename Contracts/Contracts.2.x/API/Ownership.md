# Ownership
简单授权和访问控制机制的合约模块。

> TIP
对于更复杂的需求，请参见 [Access](./Access.md)。

## Contracts

### Ownable
提供了一个基本的访问控制机制的合约模块，其中有一个账户（所有者）可以被授予对特定函数的独占访问权限。

该模块通过继承来使用。它将提供修饰符onlyOwner，可以应用于你的函数，以限制它们只能由所有者使用。

**MODIFIERS**
[onlyOwner()](#onlyowner)

**FUNCTIONS**
[constructor()](#constructor)

[owner()](#owner-→-address)

[isOwner()](#isowner-→-bool)

[renounceOwnership()](#renounceownership)

[transferOwnership(newOwner)](#transferownershipaddress-newowner)

[_transferOwnership(newOwner)](#_transferownershipaddress-newowner)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[OwnershipTransferred(previousOwner, newOwner)](#ownershiptransferredaddress-previousowner-address-newowner)

#### onlyOwner()
modifier#
如果被除了所有者之外的任何账户调用，则抛出异常。

#### constructor()
internal#
初始化合约，将部署者设为初始所有者。

#### owner() → address
public#
返回当前所有者的地址。

#### isOwner() → bool
public#
如果调用者是当前所有者，则返回true。

#### renounceOwnership()
public#

离开合约而没有所有者。将不再能够调用只有所有者的功能。只能由当前所有者调用。

> NOTE
放弃所有权将使合约失去所有者，从而删除只有所有者才能使用的任何功能。

#### transferOwnership(address newOwner)
public#
将合约的所有权转移到一个新账户（newOwner）。只能由当前所有者调用。

#### _transferOwnership(address newOwner)
internal#
将合约的所有权转移到一个新账户（newOwner）。

#### OwnershipTransferred(address previousOwner, address newOwner)
事件#

### Secondary
一个次要合约只能由其主要账户（创建它的账户）使用。

**MODIFIERS**
[onlyPrimary()](#onlyprimary)

**FUNCTIONS**
[constructor()](#constructor-1)

[primary()](#primary-→-address)

[transferPrimary(recipient)](#transferprimaryaddress-recipient)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[PrimaryTransferred(recipient)](#primarytransferredaddress-recipient)

#### onlyPrimary()
modifier#

#### constructor()
internal#
将主要账户设置为创建次级合约的账户。

#### primary() → address
public#

#### transferPrimary(address recipient)
public#
将合约转让给新的主要方。

#### PrimaryTransferred(address recipient)
事件#
当主要合约发生变化时发出。