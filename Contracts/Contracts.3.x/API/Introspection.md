# Introspection
这组接口和合约处理合约的[类型内省](https://en.wikipedia.org/wiki/Type_introspection)，也就是检查可以在其上调用的函数。这通常被称为合约的接口。

以太坊合约没有本地的接口概念，因此应用程序通常必须简单地相信它们没有进行错误的调用。对于可信任的设置，这不是问题，但经常需要与未知和不可信的第三方地址进行交互。甚至可能没有直接调用它们的方法！（例如，ERC20代币可以发送到一个没有从中转出的方法的合约中，从而永久锁定它们）。在这些情况下，声明其接口的合约可以非常有助于防止错误。

有两种主要的方法来解决这个问题。

* 本地方法中，一个合约实现IERC165并声明一个接口，第二个合约通过ERC165Checker直接查询它。

* 全局方法中，使用全局唯一的注册表（IERC1820Registry）来注册某个接口的实现者（IERC1820Implementer）。然后查询注册表，这允许更复杂的设置，比如为外部拥有的账户实现接口的合约。

请注意，在所有情况下，账户只声明其接口，但不要求实际实现它们。因此，这个机制既可以用于防止错误，又可以允许复杂的交互（参见ERC777），但不能依赖于它来保证安全。

## Local

### IERC165
ERC165标准的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-165)中定义的。

实现者可以声明对合约接口的支持，然后其他人可以查询（使用[ERC165Checker](#erc165checker)）。

有关实现，请参见[ERC165](#erc165)。

**FUNCTIONS**
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)

#### supportsInterface(bytes4 interfaceId) → bool
external#
如果此合约实现了interfaceId定义的接口，则返回true。请参阅相应的[EIP部分](https://eips.ethereum.org/EIPS/eip-165#how-interfaces-are-identified)，了解有关这些ID如何创建的更多信息。

此函数调用必须使用少于30,000 gas。

### ERC165
[IERC165](#ierc165)接口的实现。

合约可以继承自该接口，并调用[_registerInterface](#_registerinterfacebytes4-interfaceid)来声明其对某个接口的支持。

**FUNCTIONS**
[constructor()](#constructor)

[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-e28692-bool-1)

[_registerInterface(interfaceId)](#_registerinterfacebytes4-interfaceid)

#### constructor()
internal#

#### supportsInterface(bytes4 interfaceId) → bool
public#
参见 [IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-→-bool)。

时间复杂度为 O(1)，保证始终使用少于 30,000 gas。

#### _registerInterface(bytes4 interfaceId)
internal#
将合约注册为实现interfaceId定义的接口。对于实际的ERC165接口的支持是自动的，不需要注册其接口id。

请参阅 [IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-→-bool)。

要求：
* interfaceId不能是ERC165无效的接口（0xffffffff）。

### ERC165Checker
用于查询通过[IERC165](#ierc165)声明的接口支持的库。

请注意，这些函数返回查询的实际结果：如果不支持某个接口，它们不会回滚。由调用者决定在这些情况下要做什么。

**FUNCTIONS**
[supportsERC165(account)](#supportserc165address-account-→-bool)

[supportsInterface(account, interfaceId)](#supportsinterfaceaddress-account-bytes4-interfaceid-→-bool)

[getSupportedInterfaces(account, interfaceIds)](#getsupportedinterfacesaddress-account-bytes4-interfaceids-→-bool)

[supportsAllInterfaces(account, interfaceIds)](#supportsallinterfacesaddress-account-bytes4-interfaceids-→-bool)

#### supportsERC165(address account) → bool
internal#
如果账户支持[IERC165](#ierc165)接口，则返回true。

#### supportsInterface(address account, bytes4 interfaceId) → bool
internal#
如果账户支持interfaceId定义的接口，则返回true。对于[IERC165](#ierc165)本身的支持会自动查询。

请参见[IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-e28692-bool-1)。

#### getSupportedInterfaces(address account, bytes4[] interfaceIds) → bool[]
internal#
返回一个布尔数组，其中每个值对应于传入的接口是否被支持。这样可以批量检查合约的接口，因为你可能预期某些接口不被支持。

请参阅[IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-e28692-bool-1)。

*自v3.4起可用。*

#### supportsAllInterfaces(address account, bytes4[] interfaceIds) → bool
internal#
如果帐户支持interfaceIds中定义的所有接口，则返回true。自动查询对[IERC165](#ierc165)本身的支持。

批量查询可以通过跳过对[IERC165](#ierc165)支持的重复检查来节省gas。

请参阅[IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-e28692-bool-1)。

## Global

### IERC1820Registry
全球ERC1820注册表的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-1820)中所定义的。账户可以在此注册表中注册接口的实现者，以及查询支持。

实现者可以由多个账户共享，并且还可以为每个账户实现多个接口。合约可以为自身实现接口，但外部拥有的账户（EOA）必须将其委托给合约。

还可以通过注册表查询[IERC165](#ierc165)接口。

有关详细解释和源代码分析，请参阅EIP文本。

**FUNCTIONS**
[setManager(account, newManager)](#setmanageraddress-account-address-newmanager)

[getManager(account)](#getmanageraddress-account-→-address)

[setInterfaceImplementer(account, _interfaceHash, implementer)](#setinterfaceimplementeraddress-account-bytes32-_interfacehash-address-implementer)

[getInterfaceImplementer(account, _interfaceHash)](#getinterfaceimplementeraddress-account-bytes32-_interfacehash-→-address)

[interfaceHash(interfaceName)](#interfacehashstring-interfacename-→-bytes32)

[updateERC165Cache(account, interfaceId)](#updateerc165cacheaddress-account-bytes4-interfaceid)

[implementsERC165Interface(account, interfaceId)](#implementserc165interfaceaddress-account-bytes4-interfaceid-→-bool)

[implementsERC165InterfaceNoCache(account, interfaceId)](#implementserc165interfacenocacheaddress-account-bytes4-interfaceid-→-bool)

**EVENTS**
[InterfaceImplementerSet(account, interfaceHash, implementer)](#interfaceimplementersetaddress-account-bytes32-interfacehash-address-implementer)

[ManagerChanged(account, newManager)](#managerchangedaddress-account-address-newmanager)

#### setManager(address account, address newManager)
external#
将newManager设置为account的经理。账户的经理能够为其设置接口实现者。

默认情况下，每个账户都是其自己的经理。如果在newManager中传递一个值为0x0，将重置经理为初始状态。

发出[ManagerChanged](#managerchangedaddress-account-address-newmanager)事件。

要求：
* 调用者必须是account的当前经理。

#### getManager(address account) → address
external#
返回账户的管理者。

参见[setManager](#setmanageraddress-account-address-newmanager)。

#### setInterfaceImplementer(address account, bytes32 _interfaceHash, address implementer)
external#
将实现者合约设置为接口哈希的账户的实现者。

账户为零地址是调用者地址的别名。零地址也可以在实现者中使用，以移除旧的实现者。

请参阅[interfaceHash](#interfacehashstring-interfacename-→-bytes32)以了解如何创建这些。

发出[InterfaceImplementerSet](#interfaceimplementersetaddress-account-bytes32-interfacehash-address-implementer)事件。

要求：
* 调用者必须是账户的当前管理员。

* interfaceHash不能是[IERC165](#ierc165)接口ID（即它不能以28个零结尾）。

* 实现者必须实现[IERC1820Implementer](#ierc1820implementer)接口并在查询支持时返回true，除非实现者是调用者。请参阅[IERC1820Implementer.canImplementInterfaceForAddress](#canimplementinterfaceforaddressbytes32-interfacehash-address-account-→-bytes32)。

#### getInterfaceImplementer(address account, bytes32 _interfaceHash) → address
external#
返回接口哈希对应的账户的实现者。如果没有注册这样的实现者，则返回零地址。

如果interfaceHash是一个[IERC165](#ierc165)接口的id（即以28个零结尾），将查询账户是否支持它。

零地址账户是调用者地址的别名。

#### interfaceHash(string interfaceName) → bytes32
external#
返回接口名称的接口哈希，如在[EIP的相应部分中定义](https://eips.ethereum.org/EIPS/eip-1820#interface-name)的。

#### updateERC165Cache(address account, bytes4 interfaceId)
external#

#### implementsERC165Interface(address account, bytes4 interfaceId) → bool
external#

#### implementsERC165InterfaceNoCache(address account, bytes4 interfaceId) → bool
external#

#### InterfaceImplementerSet(address account, bytes32 interfaceHash, address implementer)
event#

#### ManagerChanged(address account, address newManager)
event#

### IERC1820Implementer

ERC1820实现者的接口，根据[EIP](https://eips.ethereum.org/EIPS/eip-1820#interface-implementation-erc1820implementerinterface)中定义。用于将合约注册为[IERC1820Registry](#ierc1820registry)中的实现者的合约使用。

**FUNCTIONS**
[canImplementInterfaceForAddress(interfaceHash, account)](#canimplementinterfaceforaddressbytes32-interfacehash-address-account-→-bytes32)

#### canImplementInterfaceForAddress(bytes32 interfaceHash, address account) → bytes32
external#
如果此合约为账户实现了interfaceHash，则返回特殊值(ERC1820_ACCEPT_MAGIC)。

请参见[IERC1820Registry.setInterfaceImplementer](#setinterfaceimplementeraddress-account-bytes32-_interfacehash-address-implementer)。

### ERC1820Implementer
[IERC1820Implementer](#ierc1820implementer)接口的实现。

合约可以继承这个接口，并调用[_registerInterfaceForAddress](#_registerinterfaceforaddressbytes32-interfacehash-address-account)来声明它们愿意成为实现者。然后，应调用[IERC1820Registry.setInterfaceImplementer](#setinterfaceimplementeraddress-account-bytes32-_interfacehash-address-implementer)来完成注册。

**FUNCTIONS**
[canImplementInterfaceForAddress(interfaceHash, account)](#canimplementinterfaceforaddressbytes32-interfacehash-address-account-e28692-bytes32-1)

[_registerInterfaceForAddress(interfaceHash, account)](#_registerinterfaceforaddressbytes32-interfacehash-address-account)

#### canImplementInterfaceForAddress(bytes32 interfaceHash, address account) → bytes32
public#

#### _registerInterfaceForAddress(bytes32 interfaceHash, address account)
internal#
将合约声明为愿意成为账户的interfaceHash的实现者。

请参考[IERC1820Registry.setInterfaceImplementer](#setinterfaceimplementeraddress-account-bytes32-_interfacehash-address-implementer)和[IERC1820Registry.interfaceHash](#interfacehashstring-interfacename-→-bytes32)。