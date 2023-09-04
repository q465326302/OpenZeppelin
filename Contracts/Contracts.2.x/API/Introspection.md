# Introspection
这组接口和合约用于处理合约的类型内省[（type introspection）](https://en.wikipedia.org/wiki/Type_introspection)，即检查可以在其上调用的函数。这通常被称为合约的接口。

以太坊合约没有原生的接口概念，因此应用程序通常必须简单地相信它们没有进行错误的调用。对于可信任的设置，这不是问题，但通常需要与未知和不受信任的第三方地址进行交互。甚至可能没有直接调用它们的方式！（例如，ERC20代币可能被发送到一个缺乏将其转移出去的方法的合约中，从而永久锁定它们）。在这些情况下，声明其接口的合约可以非常有助于防止错误。

有两种主要的方法来解决这个问题。

* 本地方式，其中一个合约实现了IERC165并声明了一个接口，然后第二个合约通过ERC165Checker直接查询它。

* 全局方式，其中使用一个全局且唯一的注册表（IERC1820Registry）来注册某个接口的实现者（IERC1820Implementer）。然后查询注册表，这允许更复杂的设置，如合约实现外部拥有的账户的接口。

请注意，在所有情况下，账户只是声明其接口，但不需要实际实现它们。因此，该机制既可以用于防止错误，又可以用于允许复杂的交互（参见ERC777），但不能依赖它来进行安全性。

## Local

### IERC165
ERC165标准的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-165)中定义的。

实现者可以声明对合约接口的支持，然后其他人可以查询（使用[ERC165Checker](#erc165checker)）。

有关实现，请参见[ERC165](#erc165)。

**FUNCTIONS**
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)

#### supportsInterface(bytes4 interfaceId) → bool
内部#
如果此合约实现了由interfaceId定义的接口，则返回true。有关如何创建这些id的详细信息，请参阅相应的[EIP部分](https://eips.ethereum.org/EIPS/eip-165#how-interfaces-are-identified)。

此函数调用必须使用少于30,000 gas。

### ERC165
实现[IERC165](#ierc165)接口。

合约可以继承该接口，并调用[_registerInterface](#_registerinterfacebytes4-interfaceid)来声明它们支持的接口。

**FUNCTIONS**
[constructor()](#constructor)

[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-e28692-bool-1)

[_registerInterface(interfaceId)](#_registerinterfacebytes4-interfaceid)

#### constructor()
内部#

#### supportsInterface(bytes4 interfaceId) → bool
外部#
查看 [IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-→-bool)。

时间复杂度为 O(1)，保证始终使用少于 30,000 gas。

#### _registerInterface(bytes4 interfaceId)
内部#
将合约注册为接口Id定义的接口的实现者。对于实际的ERC165接口的支持是自动的，不需要注册其接口Id。

请参阅[IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-e28692-bool-1)。

要求：

* interfaceId不能是ERC165无效接口（0xffffffff）。

### ERC165Checker
用于查询通过[IERC165](#ierc165)声明的接口支持的库。

请注意，这些函数返回查询的实际结果：如果不支持接口，它们不会回滚。由调用者决定在这些情况下要做什么。

**FUNCTIONS**
[_supportsERC165(account)](#_supportserc165address-account-→-bool)

[_supportsInterface(account, interfaceId)](#_supportsinterfaceaddress-account-bytes4-interfaceid-→-bool)

[_supportsAllInterfaces(account, interfaceIds)](#_supportsallinterfacesaddress-account-var-typebytes4-interfaceids-→-bool)

#### _supportsERC165(address account) → bool
内部#
如果账户支持[IERC165](#ierc165)接口，则返回真。

#### _supportsInterface(address account, bytes4 interfaceId) → bool
内部#
如果帐户支持由interfaceId定义的接口，则返回true。对于[IERC165](#ierc165)本身的支持会自动查询。

请参阅[IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-→-bool)。

#### _supportsAllInterfaces(address account, [.var-type]#bytes4[# interfaceIds) → bool]
内部#
如果账户支持interfaceIds中定义的所有接口，则返回true。自动查询对[IERC165](#ierc165)的支持。

通过批量查询可以节省gas费用，跳过对[IERC165](#ierc165)支持的重复检查。

参见[IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-→-bool)。

## Global

### IERC1820Registry
全球ERC1820注册表的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-1820)中定义的。账户可以在此注册表中注册接口的实现者，并查询支持。

实现者可以被多个账户共享，并且每个账户还可以实现多个接口。合约可以为自己实现接口，但外部拥有的账户（EOA）必须将此委托给合约。

还可以通过注册表查询[IERC165](#ierc165)接口。

有关详细说明和源代码分析，请参阅EIP文本。

**FUNCTIONS**
[setManager(account, newManager)](#setmanageraddress-account-address-newmanager)

[getManager(account)](#getmanageraddress-account-→-address)

[setInterfaceImplementer(account, interfaceHash, implementer)](#setinterfaceimplementeraddress-account-bytes32-interfacehash-address-implementer)

[getInterfaceImplementer(account, interfaceHash)](#getinterfaceimplementeraddress-account-bytes32-interfacehash-→-address)

[interfaceHash(interfaceName)](#interfacehashstring-interfacename-→-bytes32)

[updateERC165Cache(account, interfaceId)](#updateerc165cacheaddress-account-bytes4-interfaceid)

[implementsERC165Interface(account, interfaceId)](#implementserc165interfaceaddress-account-bytes4-interfaceid-→-bool)

[implementsERC165InterfaceNoCache(account, interfaceId)](#implementserc165interfacenocacheaddress-account-bytes4-interfaceid-→-bool)

**EVENTS**
[InterfaceImplementerSet(account, interfaceHash, implementer)](#interfaceimplementersetaddress-account-bytes32-interfacehash-address-implementer)

[ManagerChanged(account, newManager)](#managerchangedaddress-account-address-newmanager)

#### setManager(address account, address newManager)
外部#
将newManager设置为account的经理。账户的经理可以为其设置接口实现者。

默认情况下，每个账户都是自己的经理。将0x0的值传递给newManager将重置经理为此初始状态。

发出[ManagerChanged](#managerchangedaddress-account-address-newmanager)事件。

要求：
* 调用者必须是account的当前经理。

#### getManager(address account) → address
外部#
返回账户的经理。

参见[setManager](#setmanageraddress-account-address-newmanager)。

#### setInterfaceImplementer(address account, bytes32 interfaceHash, address implementer)
外部#
将实施者合约设置为`interfaceHash`的账户实施者。

账户为零地址是调用者地址的别名。零地址也可以在实施者中使用，以删除旧的实施者。

请查看[interfaceHash](#interfacehashstring-interfacename-→-bytes32)以了解如何创建它们。

发出一个[InterfaceImplementerSet](#interfaceimplementersetaddress-account-bytes32-interfacehash-address-implementer)事件。

要求：

* 调用者必须是账户的当前管理者。

* `interfaceHash`不能是一个[IERC165](#ierc165)接口ID（即它不能以28个零结尾）。

* 当查询支持时，实施者必须实现IERC1820Implementer并返回true，除非实施者是调用者。请参阅[IERC1820Implementer.canImplementInterfaceForAddress](#canimplementinterfaceforaddressbytes32-interfacehash-address-account-→-bytes32)。

#### getInterfaceImplementer(address account, bytes32 interfaceHash) → address
外部#
返回account对于interfaceHash接口的实现者。如果没有注册这样的实现者，则返回零地址。

如果interfaceHash是一个[IERC165](#ierc165)接口id（即以28个零结尾），将查询account是否支持它。

account为零地址是调用者地址的别名。

#### interfaceHash(string interfaceName) → bytes32
外部#
返回接口名称的接口哈希，如[EIP的相应部分](https://eips.ethereum.org/EIPS/eip-1820#interface-name)中所定义的。

#### updateERC165Cache(address account, bytes4 interfaceId)
外部#

#### implementsERC165Interface(address account, bytes4 interfaceId) → bool
外部#

#### implementsERC165InterfaceNoCache(address account, bytes4 interfaceId) → bool
外部#

#### InterfaceImplementerSet(address account, bytes32 interfaceHash, address implementer)
事件#

#### ManagerChanged(address account, address newManager)
事件#

### IERC1820Implementer
ERC1820实现者的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-1820#interface-implementation-erc1820implementerinterface)中所定义。被注册为IERC1820Registry中的实现者的合约使用。

该接口用于定义合约，这些合约将被注册为[IERC1820Registry](#ierc1820registry)中的实现者。

**FUNCTIONS**
[canImplementInterfaceForAddress(interfaceHash, account)](#canimplementinterfaceforaddressbytes32-interfacehash-address-account-e28692-bytes32-1)

#### canImplementInterfaceForAddress(bytes32 interfaceHash, address account) → bytes32
外部#
如果此合约为帐户实现了interfaceHash，则返回特殊值（ERC1820_ACCEPT_MAGIC）。

请参阅[IERC1820Registry.setInterfaceImplementer](#setinterfaceimplementeraddress-account-bytes32-interfacehash-address-implementer)。

### ERC1820Implementer
[IERC1820Implementer](#ierc1820implementer)接口的实现。

合约可以继承此接口，并调用[_registerInterfaceForAddress](#_registerinterfaceforaddressbytes32-interfacehash-address-account)来声明它们愿意成为实现者。然后，应调用[IERC1820Registry.setInterfaceImplementer](#setinterfaceimplementeraddress-account-bytes32-interfacehash-address-implementer)来完成注册。

**FUNCTIONS**
[canImplementInterfaceForAddress(interfaceHash, account)](#canimplementinterfaceforaddressbytes32-interfacehash-address-account-e28692-bytes32-1)

[_registerInterfaceForAddress(interfaceHash, account)](#_registerinterfaceforaddressbytes32-interfacehash-address-account)

#### canImplementInterfaceForAddress(bytes32 interfaceHash, address account) → bytes32
外部#

#### _registerInterfaceForAddress(bytes32 interfaceHash, address account)
内部#
声明该合约愿意成为账户的interfaceHash的实现者。

参考 [IERC1820Registry.setInterfaceImplementer](#setinterfaceimplementeraddress-account-bytes32-interfacehash-address-implementer) 和 [IERC1820Registry.interfaceHash](#interfacehashstring-interfacename-→-bytes32)。