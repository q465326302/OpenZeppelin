# Utilities
杂项合约包含实用函数，通常与处理不同数据类型有关。

## Contracts

### Address
与地址类型相关的函数集合

**FUNCTIONS**
[isContract(account)](#iscontractaddress-account-→-bool)

[toPayable(account)](#topayableaddress-account-→-address-payable)

[sendValue(recipient, amount)](#sendvalueaddress-payable-recipient-uint256-amount)

#### isContract(address account) → bool
internal#
如果账户是合约，则返回true。

> IMPORTANT
不能假设此函数返回false的地址是外部拥有账户（EOA）而不是合约，这是不安全的。
除其他情况外，isContract函数将返回false的地址包括：
* 外部拥有账户
* 正在构建中的合约
* 将要创建合约的地址
* 曾经存在合约但已被销毁的地址

#### toPayable(address account) → address payable
internal#
将地址转换为可支付地址。请注意，这只是一种类型转换：实际的底层值并没有改变。

*自v2.4.0起可用。*

#### sendValue(address payable recipient, uint256 amount)
internal#
Solidity的transfer的替代方法：将amount wei发送给收款人，并将所有可用的gas转发，并在出现错误时回滚。

[EIP1884](https://eips.ethereum.org/EIPS/eip-1884)增加了某些操作码的gas成本，可能使合约超过transfer所施加的2300 gas限制，从而使它们无法通过transfer接收资金。[sendValue](#sendvalueaddress-payable-recipient-uint256-amount)消除了这个限制。

[了解更多信息](https://diligence.consensys.net/posts/2019/09/stop-using-soliditys-transfer-now/)。

> IMPORTANT
由于控制权转移到收款人，必须小心不要创建重入漏洞。考虑使用[ReentrancyGuard](#reentrancyguard)或[checks-effects-interactions模式。](https://solidity.readthedocs.io/en/v0.5.11/security-considerations.html#use-the-checks-effects-interactions-pattern)

*自v2.4.0起可用。*

### SafeCast
在Solidity中，对uintXX类型的转换操作进行包装，添加了溢出检查。

在Solidity中，从uint256类型向下转换不会在溢出时回滚。这很容易导致不希望的利用或错误，因为开发人员通常会假设溢出会引发错误。SafeCast通过在此类操作溢出时回滚事务来恢复这种直觉。

使用此库而不是不安全的操作可以消除一整类错误，因此建议始终使用它。

可以与[SafeMath](./Math.md#safemath)结合使用，通过在uint256上执行所有数学运算，然后进行向下转换，将其扩展到较小的类型。

*从v2.5.0版本开始提供。*

**FUNCTIONS**
[toUint128(value)](#touint128uint256-value-→-uint128)

[toUint64(value)](#touint64uint256-value-→-uint64)

[toUint32(value)](#touint32uint256-value-→-uint32)

[toUint16(value)](#touint16uint256-value-→-uint16)

[toUint8(value)](#touint8uint256-value-→-uint8)

#### toUint128(uint256 value) → uint128
internal#
将uint256向下转换为uint128，并在溢出时进行回滚（当输入大于最大的uint128时）。

与Solidity的uint128操作符相对应。

要求：
* 输入必须适应128位。

#### toUint64(uint256 value) → uint64
internal#
将uint256强制转换为uint64，如果溢出（即输入大于最大的uint64），则回滚。

与Solidity的uint64操作符相对应。

要求：
* 输入必须适应64位。

#### toUint32(uint256 value) → uint32
internal#
返回从uint256到uint32的向下转换，当溢出时回退（当输入大于最大的uint32时）。

与Solidity的uint32运算符相对应。

要求：

* 输入必须适合32位。

#### toUint16(uint256 value) → uint16
internal#
将uint256转换为downcasted uint16，如果溢出（当输入大于最大的uint16时）则回滚。

与Solidity的uint16操作符相对应。

要求：
* 输入必须适合16位

#### toUint8(uint256 value) → uint8
internal#
将uint256转换为uint8，并在溢出时回滚（当输入大于最大的uint8时）。

这是Solidity中uint8操作符的对应功能。

要求：
* 输入必须适合8位。

### Arrays
与数组类型相关的函数集合。

**FUNCTIONS**
[findUpperBound(array, element)](#findupperboundvar-typeuint256-array-uint256-element-→-uint256)

#### findUpperBound([.var-type]#uint256[# array, uint256 element) → uint256]
internal# 
搜索一个已排序的数组，并返回第一个大于或等于给定元素的索引。如果不存在这样的索引（即数组中的所有值都严格小于给定元素），则返回数组的长度。时间复杂度为O(log n)。

数组应按升序排序，并且不包含重复元素。

### EnumerableSet
用于管理原始类型[集合](https://en.wikipedia.org/wiki/Set_(abstract_data_type))的库。

集合具有以下特性：

* 元素的添加、删除和存在性检查的时间复杂度为常数时间（O(1)）。

* 元素的枚举时间复杂度为O(n)。不保证顺序。

从v2.5.0开始，仅支持地址集。

使用EnumerableSet.AddressSet进行包含。

*自v2.5.0以来可用。*

**FUNCTIONS**
[add(set, value)](#addstruct-enumerablesetaddressset-set-address-value-→-bool)

[remove(set, value)](#removestruct-enumerablesetaddressset-set-address-value-→-bool)

[contains(set, value)](#containsstruct-enumerablesetaddressset-set-address-value-→-bool)

[enumerate(set)](#enumeratestruct-enumerablesetaddressset-set-→-address)

[length(set)](#lengthstruct-enumerablesetaddressset-set-→-uint256)

[get(set, index)](#getstruct-enumerablesetaddressset-set-uint256-index-→-address)

#### add(struct EnumerableSet.AddressSet set, address value) → bool
internal#
向集合中添加一个值。O(1)。如果该值已经在集合中，则返回false。

#### remove(struct EnumerableSet.AddressSet set, address value) → bool
internal#
从集合中删除一个值。O(1)。如果该值不存在于集合中，则返回false。

#### contains(struct EnumerableSet.AddressSet set, address value) → bool
internal#
如果值在集合中，则返回true。O（1）。

#### enumerate(struct EnumerableSet.AddressSet set) → address
internal#
返回一个包含集合中所有值的数组。O(N)。注意，数组内的值的顺序没有保证，当添加或删除更多值时，它可能会发生变化。警告：对于大型集合，此函数可能会耗尽gas：在这些情况下，请使用[length](#lengthstruct-enumerablesetaddressset-set-→-uint256)和[get](#getstruct-enumerablesetaddressset-set-uint256-index-→-address)。

#### length(struct EnumerableSet.AddressSet set) → uint256
internal#
返回集合中的元素数量。O(1)。

#### get(struct EnumerableSet.AddressSet set, uint256 index) → address
internal#
返回集合中存储在位置索引处的元素。O(1)。请注意，数组内的值的排序没有任何保证，当添加或删除更多值时，它可能会发生变化。

要求：
* 索引必须严格小于[length](#lengthstruct-enumerablesetaddressset-set-→-uint256)。

### Create2
CREATE2 EVM操作码的帮助程序使得使用CREATE2更加简单和安全。CREATE2可以用于提前计算智能合约部署的地址，从而可以实现有趣的新机制，称为“反事实交互”。

更多信息请参阅[EIP](https://eips.ethereum.org/EIPS/eip-1014#motivation)。

*自v2.5.0版本开始提供。*

**FUNCTIONS**
[deploy(salt, bytecode)](#deploybytes32-salt-bytes-bytecode-→-address)

[computeAddress(salt, bytecode)](#computeaddressbytes32-salt-bytes-bytecode-→-address)

[computeAddress(salt, bytecodeHash, deployer)](#computeaddressbytes32-salt-bytes-bytecodehash-address-deployer-→-address)

#### deploy(bytes32 salt, bytes bytecode) → address
internal#
使用CREATE2部署合约。可以通过[computeAddress](#computeaddressbytes32-salt-bytes-bytecode-→-address)提前知道合约将被部署的地址。请注意，不能使用相同的salt两次部署合约。

#### computeAddress(bytes32 salt, bytes bytecode) → address
internal#
如果通过[deploy](#deploybytes32-salt-bytes-bytecode-→-address)部署，将返回合约存储的地址。字节码或盐的任何更改都将导致新的目标地址。

#### computeAddress(bytes32 salt, bytes bytecodeHash, address deployer) → address
internal#
如果通过位[deploy](#deploybytes32-salt-bytes-bytecode-→-address)的合约部署来部署合约，则返回合约存储的地址。如果deployer是此合约的地址，则返回与[computeAddress](#computeaddressbytes32-salt-bytes-bytecode-→-address)相同的值。

### ReentrancyGuard
防止对函数进行重入调用的合约模块。

继承自ReentrancyGuard将使[nonReentrant](#nonreentrant)修饰符可用，可以应用于函数上，以确保没有嵌套的（重入的）调用。

请注意，由于只有一个nonReentrant保护，标记为nonReentrant的函数可能不会相互调用。可以通过将这些函数设置为private，然后为它们添加外部的nonReentrant入口来解决这个问题。

> TIP
如果你想了解更多关于重入性及其替代方法的信息，请查阅我们的博客文章[ Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

*自v2.5.0版本起*：由于伊斯坦布尔硬分叉引入了净燃料计量变化，该模块现在更加高效。

**MODIFIERS**
[nonReentrant()](#nonreentrant)

**FUNCTIONS**
[constructor()]

#### nonReentrant()
modifier#
防止合约直接或间接调用自身的功能。不支持从另一个nonReentrant函数调用nonReentrant函数。可以通过将nonReentrant函数设置为external，并使其调用执行实际工作的private函数来防止这种情况发生。

#### constructor()
internal#