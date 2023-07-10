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