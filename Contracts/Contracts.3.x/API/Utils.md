# Utilities
杂项合同和库包含了一些实用函数，可以用来提高安全性、处理新的数据类型或安全地使用低级别的基元。

安全工具包括：

* *Pausable*：提供了一种简单的方式来暂停合同中的活动（通常是为了应对外部威胁）。

* *ReentrancyGuard*：保护您免受[重入调用](https://blog.openzeppelin.com/reentrancy-after-istanbul/)的影响。

*Address*、*Arrays*和*Strings*库提供了与这些原生数据类型相关的更多操作，而*SafeCast*则提供了安全地在不同的有符号和无符号数值类型之间进行转换的方法。

对于新的数据类型：
* *Counters*：提供了一个只能递增或递减的计数器。非常适用于ID生成、计算合同活动等。

* *EnumerableMap*：类似于Solidity的[映射类型](https://solidity.readthedocs.io/en/latest/types.html#mapping-types)，但具有键值枚举功能：这将让您知道映射中有多少条目，并对它们进行迭代（这在映射中是不可能的）。

* *EnumerableSet*：类似于*EnumerableMap*，但用于[集合](https://en.wikipedia.org/wiki/Set_(abstract_data_type))。可以用来存储特权账户、发行的ID等。

> NOTE
由于Solidity不支持泛型类型，*EnumerableMap*和*EnumerableSet*被专门设计为适用于有限数量的键值类型。
从v3.0开始，*EnumerableMap*支持uint256 → address（UintToAddressMap），而*EnumerableSet*支持address和uint256（AddressSet和UintSet）。

最后，*Create2*包含了使用[CREATE2 EVM操作码](https://blog.openzeppelin.com/getting-the-most-out-of-create2/)所需的所有实用工具，而无需处理底层汇编。

## Contracts

### Pausable
合同模块，允许子合同实现由授权账户触发的紧急停止机制。

通过继承使用此模块。它将提供whenNotPaused和whenPaused修饰符，可以应用于您合同的函数。请注意，仅通过包含此模块，不会使函数可暂停，只有在放置修饰符之后才能暂停。

**MODIFIERS**
whenNotPaused()

whenPaused()

**FUNCTIONS**
constructor()

paused()

_pause()

_unpause()

**EVENTS**
Paused(account)

Unpaused(account)

#### whenNotPaused()
修饰符#
修改使函数仅在合约未暂停时可调用。

要求：
* 合约不能被暂停。

#### whenPaused()
修饰符#
只有当合约被暂停时才能调用的修饰符。

要求：
* 合同必须暂停。

#### constructor()
内部#
在未暂停状态下初始化合约。

#### paused() → bool
公开#
如果合同暂停，则返回true，否则返回false。

#### _pause()
内部#
触发停止状态。

要求：
* 合同不能被暂停。

#### _unpause()
内部#
返回正常状态。

要求：
* 合同必须暂停。

#### Paused(address account)
事件#
当账户触发暂停时发出。

#### Unpaused(address account)
事件#
当账户解除暂停时发出的信号。

### ReentrancyGuard
防止对函数进行重入调用的合约模块。

继承自ReentrancyGuard将使*nonReentrant*修饰符可用，可以应用于函数以确保没有嵌套（重入）调用它们。

请注意，由于只有一个nonReentrant保护程序，标记为nonReentrant的函数可能不能互相调用。可以通过将这些函数设置为私有函数，然后为它们添加外部的nonReentrant入口来解决这个问题。

> TIP
如果您想了解更多关于重入性及其替代保护方法的信息，请查阅我们的博客文章[Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

**MODIFIERS**
nonReentrant()

**FUNCTIONS**
constructor()

#### nonReentrant()
修饰符#
禁止合约直接或间接调用自身的功能。不支持从另一个非重入函数调用非重入函数。可以通过将非重入函数设置为external，并使其调用一个执行实际工作的私有函数来防止这种情况发生。

#### constructor()
内部#

## Libraries

### Address
与地址类型相关的函数集合

**FUNCTIONS**
isContract(account)

sendValue(recipient, amount)

functionCall(target, data)

functionCall(target, data, errorMessage)

functionCallWithValue(target, data, value)

functionCallWithValue(target, data, value, errorMessage)

functionStaticCall(target, data)

functionStaticCall(target, data, errorMessage)

functionDelegateCall(target, data)

functionDelegateCall(target, data, errorMessage)

#### isContract(address account) → bool
内部#
如果账户是一个合约，则返回true。

> IMPORTANT
不能确定该函数返回false的地址是否是一个外部拥有账户（EOA）而不是一个合约，这样做是不安全的。
除其他情况外，isContract将对以下类型的地址返回false：
* 一个外部拥有账户
* 正在构建的合约
* 将创建合约的地址
* 曾经存在合约但已被销毁的地址

#### sendValue(address payable recipient, uint256 amount)
内部#
Solidity中transfer的替代方法：将amount wei发送给recipient，转发所有可用的gas，并在出错时回滚。

[EIP1884](https://eips.ethereum.org/EIPS/eip-1884)增加了某些操作码的gas成本，可能会使合约超过transfer所施加的2300 gas限制，导致无法通过transfer接收资金。*sendValue*消除了这个限制。

[了解更多](https://diligence.consensys.net/posts/2019/09/stop-using-soliditys-transfer-now/)。

> IMPORTANT
由于控制权转移到了recipient，必须注意不要创建重入漏洞。可以考虑使用*ReentrancyGuard*或[checks-effects-interactions模式](https://solidity.readthedocs.io/en/v0.5.11/security-considerations.html#use-the-checks-effects-interactions-pattern)。

#### functionCall(address target, bytes data) → bytes
内部#
使用低级别的调用执行Solidity函数调用。普通的 call 是对函数调用的不安全替代：请改用这个函数。

如果目标使用回退原因进行回退，此函数会将其上升（类似于常规的Solidity函数调用）。

返回原始返回数据。要转换为预期的返回值，请使用[abi.decode](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html?highlight=abi.decode#abi-encoding-and-decoding-functions)。

要求：

* 目标必须是一个合约。

* 使用数据调用目标不能回滚。

*自v3.1起可用。*

#### functionCall(address target, bytes data, string errorMessage) → bytes
内部#
与*functionCall*相同，但在目标函数回退时使用errorMessage作为回退原因的回退。

*自v3.1起可用。*

#### functionCallWithValue(address target, bytes data, uint256 value) → bytes
内部#
与functionCall相同，但还将wei值转移到目标。

要求：
* 调用合同的ETH余额必须至少为value。
* 被调用的Solidity函数必须可支付。

*自v3.1起可用。*

#### functionCallWithValue(address target, bytes data, uint256 value, string errorMessage) → bytes
内部#
与*functionCallWithValue*相同，但在目标函数回退时使用errorMessage作为回退原因的回退。

*从v3.1开始可用。*

#### functionStaticCall(address target, bytes data) → bytes
内部#
与*functionCall*相同，但执行静态调用。

*自v3.3起可用。*

#### functionStaticCall(address target, bytes data, string errorMessage) → bytes
内部#
与*functionCall*相同，但执行静态调用。

*自v3.3起可用。*

#### functionDelegateCall(address target, bytes data) → bytes
内部#
与*functionCall*相同，但执行委托调用。

*自v3.4版本起可用。*

#### functionDelegateCall(address target, bytes data, string errorMessage) → bytes
内部#
与*functionCall*相同，但执行委托调用。

*自v3.4起可用。*

### Arrays
与数组类型相关的函数集合。

**FUNCTIONS**
findUpperBound(array, element)

#### findUpperBound(uint256[] array, uint256 element) → uint256
内部#
搜索一个已排序的数组，并返回第一个大于或等于给定元素的索引。如果不存在这样的索引（即数组中的所有值都严格小于给定元素），则返回数组的长度。时间复杂度为O(log n)。

数组被期望以升序排序，并且不包含重复元素。

### Counters
提供了一种只能递增或递减一个单位的计数器。这可以用于跟踪映射中的元素数量、发放ERC721 ID或计算请求ID。

在使用Counters for Counters.Counter;时，可以省略*SafeMath*溢出检查，因为递增一个单位不可能导致256位整数溢出，从而节省了燃气。然而，这需要正确的使用方式，即不能直接访问底层的_value。

**FUNCTIONS**
current(counter)

increment(counter)

decrement(counter)

#### current(struct Counters.Counter counter) → uint256
内部#

#### increment(struct Counters.Counter counter)
内部#

#### decrement(struct Counters.Counter counter)
内部#

### Create2
CREATE2 EVM操作码的帮助程序，使使用更加简便和安全。CREATE2可用于提前计算智能合约部署的地址，这可以实现一些有趣的新机制，称为“反事实交互”。

有关更多信息，请参阅[EIP](https://eips.ethereum.org/EIPS/eip-1014#motivation)。

**FUNCTIONS**
deploy(amount, salt, bytecode)

computeAddress(salt, bytecodeHash)

computeAddress(salt, bytecodeHash, deployer)

#### deploy(uint256 amount, bytes32 salt, bytes bytecode) → address
内部#
使用CREATE2部署合约。合约将被部署到的地址可以通过*computeAddress*提前知道。

合约的字节码可以通过Solidity的type(contractName).creationCode获取。

要求：
* 字节码不能为空。

* salt在之前没有被用于字节码。

* 工厂账户的余额必须至少为amount。

* 如果amount不为零，字节码必须具有可支付的构造函数。

#### computeAddress(bytes32 salt, bytes32 bytecodeHash) → address
内部#
如果通过部署操作*部署*合约，将返回合约存储的地址。如果bytecodeHash或salt发生变化，将导致生成一个新的目标地址。

#### computeAddress(bytes32 salt, bytes32 bytecodeHash, address deployer) → address
内部#
如果通过位于deployer的合约*部署*合约，则返回合约将存储的地址。如果deployer是该合约的地址，则返回与*computeAddress*相同的值。

### EnumerableMap
管理Solidity的可枚举映射类型的库。

映射具有以下特性：
* 添加、删除和检查条目的存在都在常量时间（O(1)）内完成。

* 条目的枚举时间复杂度为O(n)。对于排序不做任何保证。
```
contract Example {
    // Add the library methods
    using EnumerableMap for EnumerableMap.UintToAddressMap;

    // Declare a set state variable
    EnumerableMap.UintToAddressMap private myMap;
}
```
截至v3.0.0，只支持uint256 → address类型的映射（UintToAddressMap）。

**FUNCTIONS**
set(map, key, value)

remove(map, key)

contains(map, key)

length(map)

at(map, index)

tryGet(map, key)

get(map, key)

get(map, key, errorMessage)

#### set(struct EnumerableMap.UintToAddressMap map, uint256 key, address value) → bool
内部#
向映射中添加一个键值对，或者更新现有键的值。O(1)。

如果键被添加到映射中（即之前不存在），则返回true。

#### remove(struct EnumerableMap.UintToAddressMap map, uint256 key) → bool
内部#
从集合中删除一个值。O(1)。

如果键从映射中删除，即存在，则返回true。

#### contains(struct EnumerableMap.UintToAddressMap map, uint256 key) → bool
内部#
如果键存在于映射中，则返回true。O(1)。

#### length(struct EnumerableMap.UintToAddressMap map) → uint256
内部#
返回映射中的元素数量。O(1)。

#### at(struct EnumerableMap.UintToAddressMap map, uint256 index) → uint256, address
内部#
在集合中返回存储在位置索引处的元素。O(1)。请注意，数组内的值的排序没有任何保证，并且当添加或删除更多值时，它可能会发生变化。

要求：
* 索引必须严格小于长度。

#### tryGet(struct EnumerableMap.UintToAddressMap map, uint256 key) → bool, address
内部#
尝试返回与键关联的值。O(1)时间复杂度。如果键不在映射中，则不会回滚。

*自v3.4版本起可用。*

#### get(struct EnumerableMap.UintToAddressMap map, uint256 key) → address
内部#
返回与键相关联的值。O(1)。

要求：
* 键必须存在于映射中。

#### get(struct EnumerableMap.UintToAddressMap map, uint256 key, string errorMessage) → address
内部#
与*get*相同，当键不在映射中时具有自定义错误消息。

> CAUTION
此函数已弃用，因为它不必要地需要为错误消息分配内存。要使用自定义的回滚原因，请使用*tryGet*。

### EnumerableSet
用于管理原始类型集的库。

集合具有以下属性：

* 元素的添加、移除和存在检查在常数时间（O(1)）内完成。

* 元素的枚举在O(n)内完成。对排序不做任何保证。

```
contract Example {
    // Add the library methods
    using EnumerableSet for EnumerableSet.AddressSet;

    // Declare a set state variable
    EnumerableSet.AddressSet private mySet;
}
```

从v3.3.0版本开始，支持bytes32类型的集合（Bytes32Set）、address类型的集合（AddressSet）和uint256类型的集合（UintSet）。

**FUNCTIONS**
add(set, value)

remove(set, value)

contains(set, value)

length(set)

at(set, index)

add(set, value)

remove(set, value)

contains(set, value)

length(set)

at(set, index)

add(set, value)

remove(set, value)

contains(set, value)

length(set)

at(set, index)

#### add(struct EnumerableSet.Bytes32Set set, bytes32 value) → bool
内部#
向集合中添加一个值。O(1)。

如果该值被添加到集合中（即之前不存在），则返回true。

#### remove(struct EnumerableSet.Bytes32Set set, bytes32 value) → bool
内部#
从集合中删除一个值。 O(1)。

如果该值从集合中被删除，即存在，则返回true。

#### contains(struct EnumerableSet.Bytes32Set set, bytes32 value) → bool
内部#
如果值在集合中，则返回true。O(1)。

#### length(struct EnumerableSet.Bytes32Set set) → uint256
内部#
返回集合中的值的数量。O(1)。

#### at(struct EnumerableSet.Bytes32Set set, uint256 index) → bytes32
内部#
在集合中返回存储在索引位置的值。O(1)。

请注意，数组内的值的顺序没有保证，并且当添加或删除更多值时，它可能会更改。

要求：
* 索引必须严格小于*length*。

#### add(struct EnumerableSet.AddressSet set, address value) → bool
内部#
向集合中添加一个值。O(1)。

如果该值已经存在于集合中，则返回false；如果该值成功添加到集合中，则返回true。

#### remove(struct EnumerableSet.AddressSet set, address value) → bool
内部#
从集合中删除一个值。O(1)。

如果该值从集合中被删除，即存在于集合中，则返回true。

#### contains(struct EnumerableSet.AddressSet set, address value) → bool
内部#
如果值在集合中，则返回true。O(1)。

#### length(struct EnumerableSet.AddressSet set) → uint256
内部# 
返回集合中的值的数量。O(1)。

#### at(struct EnumerableSet.AddressSet set, uint256 index) → address
内部#
在集合中返回存储在索引位置的值。 O(1)。

请注意，数组内部值的排序没有保证，并且当添加或删除更多值时，它可能会发生变化。

要求：
* 索引必须严格小于* length*。

#### add(struct EnumerableSet.UintSet set, uint256 value) → bool
内部#
将一个值添加到集合中。O(1)。

如果该值尚未存在于集合中，则返回true，表示已将其添加到集合中。

#### remove(struct EnumerableSet.UintSet set, uint256 value) → bool
内部#
从集合中移除一个值。O(1)。

如果值被从集合中移除，则返回true，即如果该值存在。

#### contains(struct EnumerableSet.UintSet set, uint256 value) → bool
内部# 
如果值在集合中，则返回true。O(1)。

#### length(struct EnumerableSet.UintSet set) → uint256
内部#
返回集合中的值的数量。O(1)。

#### at(struct EnumerableSet.UintSet set, uint256 index) → uint256
内部#
在集合中返回存储在位置索引处的值。O(1)。

请注意，数组内部的值的排序没有保证，在添加或删除更多值时可能会发生变化。

要求：
* 索引必须严格小于*length*。

### SafeCast
在Solidity中，对uintXX/intXX类型进行包装，添加了溢出检查。

在Solidity中，从uint256/int256类型进行向下转换不会在溢出时回滚。这很容易导致意外的利用或错误，因为开发人员通常假设溢出会引发错误。通过在此类操作溢出时回滚事务，SafeCast恢复了这种直觉。

与未经检查的操作相比，使用此库可以消除整个类别的错误，因此建议始终使用它。

可以与*SafeMath*和*SignedSafeMath*结合使用，通过在uint256和int256上执行所有数学运算，然后进行向下转换，将其扩展到较小的类型。

*FUNCTIONS*
toUint128(value)

toUint64(value)

toUint32(value)

toUint16(value)

toUint8(value)

toUint256(value)

toInt128(value)

toInt64(value)

toInt32(value)

toInt16(value)

toInt8(value)

toInt256(value)

#### toUint128(uint256 value) → uint128
内部#
将uint256强制转换为uint128，并在溢出时恢复（当输入大于最大的uint128时）。

与Solidity的uint128运算符相对应。

要求：
* 输入必须适合128位。

#### toUint64(uint256 value) → uint64
内部#
将uint256从uint64下转换为，如果输入大于最大的uint64，则返回revert。

与Solidity的uint64运算符相对应。

要求：
* 输入必须适合64位。

#### toUint32(uint256 value) → uint32
内部#
将uint256从uint32下转换，当溢出时返回（当输入大于最大的uint32时）。

与Solidity的uint32操作符相对应。

要求：
* 输入必须适合32位。

#### toUint16(uint256 value) → uint16
内部#
将uint256转换为uint16，并在溢出时回滚（当输入大于最大的uint16时）。

这是Solidity中uint16操作符的对应物。

要求：
* 输入必须适合16位。

#### toUint8(uint256 value) → uint8
内部#
将uint256转换为uint8并返回结果，当输入大于最大的uint8时，会回滚（revert）。

与Solidity的uint8操作符相对应。

要求：
* 输入必须适合8位。

#### toUint256(int256 value) → uint256
内部#
将有符号的int256转换为无符号的uint256。

要求：
* 输入必须大于或等于0。

#### toInt128(int256 value) → int128
内部#
将int256转换为int128，如果溢出则回滚（当输入小于最小int128或大于最大int128时）。

与Solidity的int128操作符相对应。

要求：
* 输入必须适合128位

*自v3.1起可用。*

#### toInt64(int256 value) → int64
内部#
从int256返回向下转换的int64，当输入小于最小int64或大于最大int64时会回滚（溢出）。

与Solidity的int64运算符相对应。

要求：
* 输入必须适合64位

*从v3.1开始可用。*

#### toInt32(int256 value) → int32
内部#
将int256向下转换为int32，如果溢出则回滚（当输入小于最小int32或大于最大int32时）。

与Solidity的int32操作符相对应。

要求：
* 输入必须适合32位

*自v3.1起可用。*

#### toInt16(int256 value) → int16
内部#
将int256转换为int16的结果，如果溢出则会回退（即输入小于最小int16或大于最大int16）。

与Solidity的int16操作符相对应。

要求：
* 输入必须适合16位。

*自v3.1起可用。*

#### toInt8(int256 value) → int8
内部#

从int256返回向下转型的int8，在溢出时回滚（当输入小于最小的int8或大于最大的int8时）。

与Solidity的int8运算符相对应。

要求：
* 输入必须适合8位。

*从v3.1开始可用。*

#### toInt256(uint256 value) → int256
内部#
将一个无符号的uint256转换成有符号的int256。

要求：
* 输入必须小于或等于maxInt256。

### Strings

字符串操作

**FUNCTIONS**
toString(value)

#### toString(uint256 value) → string
内部#
将一个uint256转换为它的ASCII字符串表示。