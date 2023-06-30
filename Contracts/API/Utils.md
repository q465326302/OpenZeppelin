# Utilities
杂项合约和库包含了实用函数，可以提高安全性、处理新的数据类型或安全地使用底层基元。

*Address*、*Arrays*、*Base64*和*Strings*库提供了与这些原生数据类型相关的更多操作，而*SafeCast*则提供了安全地在不同的有符号和无符号数值类型之间转换的方法。*Multicall*提供了一种将多个调用合并为单个外部调用的函数。

对于新的数据类型：

* *Counters*：一种简单的方式来获取只能递增、递减或重置的计数器。非常适用于ID生成、计算合约活动等。
* *EnumerableMap*：类似于Solidity的*mapping*类型，但具有键值枚举功能：这将让您知道一个映射有多少条目，并可以对它们进行迭代（这在mapping中是不可能的）。
* *EnumerableSet*：类似于*EnumerableMap*，但用于[集合](https://en.wikipedia.org/wiki/Set_(abstract_data_type))。可用于存储特权账户、发行的ID等。

> NOTE
由于Solidity不支持泛型类型，*EnumerableMap*和*EnumerableSet*专门针对有限数量的键值类型进行了特化。
从v3.0开始，*EnumerableMap*支持uint256 → address（UintToAddressMap），*EnumerableSet*支持address和uint256（AddressSet和UintSet）。

最后，*Create2*包含了所有必要的实用工具，可以安全地使用[CREATE2 EVM操作码](https://blog.openzeppelin.com/getting-the-most-out-of-create2/)，而无需处理底层汇编。

## Math

### Math
```
import "@openzeppelin/contracts/utils/math/Math.sol";
```
Solidity语言中缺少的标准数学工具。

**FUNCTIONS**
max(a, b)
min(a, b)
average(a, b)
ceilDiv(a, b)
mulDiv(x, y, denominator)
mulDiv(x, y, denominator, rounding)
sqrt(a)
sqrt(a, rounding)
log2(value)
log2(value, rounding)
log10(value)
log10(value, rounding)
log256(value)
log256(value, rounding)

#### max(uint256 a, uint256 b) → uint256
内部#
返回两个数中的较大值。

#### min(uint256 a, uint256 b) → uint256
内部#
返回两个数字中较小的数字。

#### average(uint256 a, uint256 b) → uint256
内部#
返回两个数字的平均值。结果向零舍入。

#### ceilDiv(uint256 a, uint256 b) → uint256
内部#
返回两个数相除的上取整结果。

与标准除法（/）不同的是，它向上舍入而不是向下舍入。

#### mulDiv(uint256 x, uint256 y, uint256 denominator) → uint256 result
内部#
原始信用归Remco Bloemen使用MIT许可证（https://xn—​2-umb.com/21/muldiv），由Uniswap Labs进行进一步编辑，同样使用MIT许可证。

#### mulDiv(uint256 x, uint256 y, uint256 denominator, enum Math.Rounding rounding) → uint256
内部#

#### sqrt(uint256 a) → uint256
内部#
返回一个数的平方根。如果这个数不是一个完全平方数，则取下整数部分。

受到Henry S. Warren, Jr.的《Hacker’s Delight》（第11章）的启发。

#### sqrt(uint256 a, enum Math.Rounding rounding) → uint256
内部#

#### log2(uint256 value) → uint256
内部#
返回一个正数的以2为底的对数的向下取整值。如果给定0，则返回0。

#### log2(uint256 value, enum Math.Rounding rounding) → uint256
内部#
返回以所选舍入方式为基数2的对数，如果给定0，则返回0。

#### log10(uint256 value) → uint256
内部#
返回一个正数的以10为底的对数，向下取整。如果给定0，则返回0。

#### log10(uint256 value, enum Math.Rounding rounding) → uint256
内部#
返回正数的以10为底的对数，根据选择的舍入方向。如果给定0，则返回0。

#### log256(uint256 value) → uint256
内部#
返回一个正数的以256为底的对数，向下取整。如果给定的是0，则返回0。

将结果加一即可得到以十六进制字符串表示该值所需的十六进制符号对数。

#### log256(uint256 value, enum Math.Rounding rounding) → uint256
内部#
返回以256为底的对数，按照选择的舍入方向，对一个正值进行计算。如果给定0，则返回0。

### SignedMath
```
import "@openzeppelin/contracts/utils/math/SignedMath.sol";
```
Solidity语言中缺少标准的有符号数学工具。

**FUNCTIONS**
max(a, b)
min(a, b)
average(a, b)
abs(n)

#### max(int256 a, int256 b) → int256
内部#
返回两个有符号数中的最大值。

#### min(int256 a, int256 b) → int256
内部#
返回两个有符号数字中的最小值。

#### average(int256 a, int256 b) → int256
内部#
返回两个有符号数的平均值，避免溢出。结果向零舍入。

#### abs(int256 n) → uint256
内部#
返回有符号值的绝对无符号值。

### SafeCast
```
import "@openzeppelin/contracts/utils/math/SafeCast.sol";
```
在Solidity中，对uintXX/intXX进行包装，添加了溢出检查。

在Solidity中，从uint256/int256进行向下转换不会在溢出时回滚。这很容易导致意外的利用或错误，因为开发人员通常假设溢出会引发错误。SafeCast通过在此类操作溢出时回滚事务来恢复这种直觉。

使用这个库而不是未检查的操作可以消除整个类别的错误，因此建议始终使用它。

可以与*SafeMath*和*SignedSafeMath*结合使用，通过在uint256和int256上执行所有数学运算，然后进行向下转换，将其扩展到较小的类型。

**FUNCTIONS**
toUint248(value)

toUint240(value)

toUint232(value)

toUint224(value)

toUint216(value)

toUint208(value)

toUint200(value)

toUint192(value)

toUint184(value)

toUint176(value)

toUint168(value)

toUint160(value)

toUint152(value)

toUint144(value)

toUint136(value)

toUint128(value)

toUint120(value)

toUint112(value)

toUint104(value)

toUint96(value)

toUint88(value)

toUint80(value)

toUint72(value)

toUint64(value)

toUint56(value)

toUint48(value)

toUint40(value)

toUint32(value)

toUint24(value)

toUint16(value)

toUint8(value)

toUint256(value)

toInt248(value)

toInt240(value)

toInt232(value)

toInt224(value)

toInt216(value)

toInt208(value)

toInt200(value)

toInt192(value)

toInt184(value)

toInt176(value)

toInt168(value)

toInt160(value)

toInt152(value)

toInt144(value)

toInt136(value)

toInt128(value)

toInt120(value)

toInt112(value)

toInt104(value)

toInt96(value)

toInt88(value)

toInt80(value)

toInt72(value)

toInt64(value)

toInt56(value)

toInt48(value)

toInt40(value)

toInt32(value)

toInt24(value)

toInt16(value)

toInt8(value)

toInt256(value)

#### toUint248(uint256 value) → uint248
内部#
将uint256强制转换为uint248，如果溢出（输入大于最大的uint248）则会回滚。

与Solidity的uint248操作符相对应。

要求：
* 输入必须适应248位

**从v4.7开始可用。**

#### toUint240(uint256 value) → uint240
内部#
将uint256从uint240降级为，当输入大于最大的uint240时回滚（发生溢出）。

与Solidity的uint240操作符相对应。

要求:
* 输入必须适合240位

**自v4.7起可用。**

#### toUint232(uint256 value) → uint232
内部#
将uint256转换为uint232，并在溢出时回滚（当输入大于最大的uint232时）。

与Solidity的uint232操作符相对应。

要求：
* 输入必须适合232位。

*从v4.7开始可用。*

#### toUint224(uint256 value) → uint224
内部#
将uint256向下转换为uint224，并在溢出时回滚（当输入大于最大的uint224时）。

与Solidity的uint224操作符相对应。

要求：
* 输入必须适合224位

*自v4.2起可用。*

#### toUint216(uint256 value) → uint216
内部#
将uint256转换为downcasted uint216，并在溢出时回滚（当输入大于最大的uint216时）。

对应Solidity的uint216运算符。

要求：
* 输入必须适应216位

*自v4.7起可用。*

#### toUint208(uint256 value) → uint208
内部#
将uint256转换为downcasted uint208，如果溢出（输入大于最大的uint208）则回滚。

与Solidity的uint208操作符相对应。

要求：
* 输入必须适合208位

*自v4.7起可用。*

#### toUint200(uint256 value) → uint200
内部#
从uint256返回下转型的uint200，当输入大于最大的uint200时会回滚（溢出）。

与Solidity的uint200操作符相对应。

要求：
* 输入必须适合200位

*自v4.7起可用。*

#### toUint192(uint256 value) → uint192
内部#
将uint256向下转换为uint192，并在溢出时回滚（当输入大于最大的uint192时）。

这是Solidity中uint192运算符的对应物。

要求：
* 输入必须适应192位

*从v4.7开始提供。*

#### toUint184(uint256 value) → uint184
内部#
将uint256转换为uint184，并在溢出时回滚（当输入大于最大的uint184时）。

与Solidity的uint184运算符相对应。

要求：
* 输入必须适合184位

*从v4.7开始可用。*

#### toUint176(uint256 value) → uint176
内部#
将uint256从uint176中向下转换，当溢出时返回（当输入大于最大的uint176时）。

与Solidity的uint176运算符相对应。

要求：
* 输入必须适合176位

*自v4.7起可用。*

#### toUint168(uint256 value) → uint168
内部#
将uint256从uint168向下转换，并在溢出时返回（当输入大于最大uint168时）。

与Solidity的uint168操作符相对应。

要求：
* 输入必须适合168位

*自v4.7起可用。*

#### toUint160(uint256 value) → uint160
内部#
将uint256从uint160降低，并在溢出时返回（当输入大于最大的uint160时）。

与Solidity的uint160操作符相对应。

要求：
* 输入必须适应160位

*自v4.7以来可用。*

#### toUint152(uint256 value) → uint152
内部#
将uint256类型的数字向下转换为uint152类型的数字，并在溢出时回滚（当输入大于最大的uint152时）。

这是Solidity中uint152运算符的对应函数。

要求：
* 输入必须适合152位

*从v4.7版本开始可用。*

#### toUint144(uint256 value) → uint144
内部#
将uint256从uint144返回为向下转型，溢出时回滚（当输入大于最大的uint144时）。

与Solidity的uint144操作符相对应。

要求：
* 输入必须适合144位

*自v4.7起可用。*

#### toUint136(uint256 value) → uint136
内部#
将uint256从uint136返回为降低，当溢出时回滚（当输入大于最大的uint136时）。

与Solidity的uint136操作符相对应。

要求：
* 输入必须适合136位

*自v4.7起可用。*

#### toUint128(uint256 value) → uint128
内部#
将uint256转换为uint128，并在溢出时引发错误（当输入大于最大的uint128时）。

与Solidity的uint128操作符相对应。

要求：
* 输入必须适合128位

*从v2.5开始提供。*

#### toUint120(uint256 value) → uint120
内部#
将uint256向下转换为uint120，并在溢出时返回（当输入大于最大的uint120时）。

与Solidity的uint120运算符相对应。

要求：
* 输入必须适合120位

*自v4.7以来可用。*

#### toUint112(uint256 value) → uint112
内部#
从uint256返回向下转换的uint112，当输入大于最大的uint112时，会发生溢出并导致程序回滚。

与Solidity的uint112运算符相对应。

要求：
* 输入必须适合112位

*自v4.7起可用。*

#### toUint104(uint256 value) → uint104
内部#
将uint256转换为downcasted uint104，并在溢出时进行回滚（当输入大于最大的uint104时）。

与Solidity的uint104运算符相对应。

要求：
* 输入必须适合104位

*自v4.7起可用。*

#### toUint96(uint256 value) → uint96
内部#
将uint256类型的值向下转换为uint96类型的值，当输入值大于最大的uint96时，将引发错误。

这是Solidity中uint96操作符的对应函数。

要求：
* 输入值必须适合96位

*从v4.2版本开始可用。*

#### toUint88(uint256 value) → uint88
内部#
将uint256转换为uint88，并在溢出时回滚（当输入大于最大的uint88时）。

与Solidity的uint88操作符相对应。

要求：
* 输入必须适合88位

*自v4.7起可用。*

#### toUint80(uint256 value) → uint80
内部#
将uint256向下转换为uint80，当输入大于最大uint80时会发生溢出，从而导致回滚。

要求：
* 输入必须适应80位

从v4.7开始提供。

#### toUint72(uint256 value) → uint72
内部#
将uint256转换为下投uint64，当输入大于最大的uint64时会回滚（即溢出）。

与Solidity的uint64操作符相对应。

要求：
* 输入必须适应64位

*从v2.5开始可用。*

#### toUint56(uint256 value) → uint56
内部#
将uint256转换为downcasted uint56，当输入大于最大的uint56时会回滚（溢出）。

与Solidity的uint56操作符相对应。

要求：
* 输入必须适合56位

*自v4.7起可用。*

#### toUint48(uint256 value) → uint48
内部#
将uint256转换为uint48，并在溢出时回滚（当输入大于最大的uint48时）。

与Solidity的uint48运算符相对应。

要求：
* 输入必须适合48位

*从v4.7开始可用。*

#### toUint40(uint256 value) → uint40
内部#
将uint256强制转换为uint40，并在溢出时回滚（当输入大于最大的uint40时）。

与Solidity的uint40操作符相对应。

要求：
* 输入必须适应40位

*自v4.7以来可用。*

#### toUint32(uint256 value) → uint32
内部#
从uint256返回downcasted uint32，当输入大于最大uint32时，返回溢出（Overflow）。

与Solidity的uint32操作符相对应。

要求：
*输入必须适合32位

*自v2.5起可用。*

#### toUint24(uint256 value) → uint24
内部#
从uint256返回downcasted uint24，在溢出时回滚（当输入大于最大uint24时）。

与Solidity的uint24操作符相对应。

要求：
* 输入必须适合24位

*自v4.7起可用。*

#### toUint16(uint256 value) → uint16
内部#
将uint256转换为uint16，并在溢出时返回（当输入大于最大的uint16时）。

与Solidity的uint16操作符相对应。

要求：
* 输入必须适合16位

*自v2.5起可用。*

#### toUint8(uint256 value) → uint8
内部#
将uint256类型下转型为uint8类型，在溢出时进行回滚（当输入大于最大的uint8时）。

与Solidity的uint8运算符相对应。

要求：
* 输入必须适合8位

*从v2.5版本开始提供。*

#### toUint256(int256 value) → uint256
内部#
将有符号的int256转换为无符号的uint256。

要求：
* 输入必须大于或等于0。

*从v3.0开始可用。*

#### toInt248(int256 value) → int248 downcasted
内部#
将int256转换为int248，并在溢出时进行还原（当输入小于最小的int248或大于最大的int248时）。

与Solidity的int248运算符相对应。

要求：
* 输入必须适合248位

*自v4.7起可用。*

#### toInt240(int256 value) → int240 downcasted
内部#
将int256转换为下降的int240，当溢出时回滚（当输入小于最小的int240或大于最大的int240时）。

与Solidity的int240运算符相对应。

要求：
* 输入必须适合240位

*自v4.7起可用。*

#### toInt232(int256 value) → int232 downcasted
内部#
将int256的值向下转换为int232，并在溢出时恢复（当输入小于最小int232或大于最大int232时）。

与Solidity的int232运算符相对应。

要求：
 * 输入必须适应232位

*自v4.7起可用。*

#### toInt224(int256 value) → int224 downcasted
内部#
将int256转换为下限为int224的整数，当溢出时恢复（当输入小于最小int224或大于最大int224时）。

与Solidity的int224运算符相对应。

要求：
* 输入必须适合224位

*自v4.7起可用。*

#### toInt216(int256 value) → int216 downcasted
内部#
将int256类型的数值转换为int216类型的数值，并在溢出时进行回滚（当输入小于最小的int216或大于最大的int216时）。

与Solidity的int216操作符相对应。

要求：
* 输入必须适合216位

*自v4.7起可用。*

#### toInt208(int256 value) → int208 downcasted
内部#
将int256转换为int208，并在溢出时进行回滚（当输入小于最小的int208或大于最大的int208时）。

与Solidity的int208操作符相对应。

要求：
* 输入必须适合208位

*从v4.7开始可用。*

#### toInt200(int256 value) → int200 downcasted
内部#
将int256强制转换为int200，并在溢出时回滚（当输入小于最小的int200或大于最大的int200时）。

与Solidity的int200运算符相对应。

要求：
*输入必须适合200位

*自v4.7起可用。*

#### toInt192(int256 value) → int192 downcasted
内部#
将int256转换为int192，并在溢出时返回，当输入小于最小int192或大于最大int192时会发生溢出。

与Solidity的int192运算符相对应。

要求：
* 输入必须适合192位

*自v4.7起可用。*

#### toInt184(int256 value) → int184 downcasted
内部#
将int256返回为downcasted int184，当溢出时进行回滚（当输入小于最小int184或大于最大int184时）。

与Solidity的int184运算符相对应。

要求：
* 输入必须适合184位

*自v4.7起可用。*

#### toInt176(int256 value) → int176 downcasted
内部#
返回从int256到int176的向下转换，当溢出时恢复（当输入小于最小int176或大于最大int176时）。

这是Solidity的int176运算符的对应物。

要求：
* 输入必须适合176位

**自v4.7起可用。**

#### toInt168(int256 value) → int168 downcasted
内部#
将int256转换为int168，并在溢出时进行回滚（当输入小于最小的int168或大于最大的int168时）。

与Solidity的int168运算符相对应。

要求：
* 输入必须适合168位

*自v4.7起可用。*

#### toInt160(int256 value) → int160 downcasted
内部#
将int256转换为int160，并在溢出时恢复（当输入小于最小int160或大于最大int160时）。

与Solidity的int160操作符相对应。

要求：
* 输入必须适合160位

*从v4.7版本开始提供。*

#### toInt152(int256 value) → int152 downcasted
内部#
将int256转换为int152，并在溢出时返回，当输入小于最小的int152或大于最大的int152时进行回滚。

与Solidity的int152操作符相对应。

要求：
* 输入必须适合152位

**自v4.7起可用。**

#### toInt144(int256 value) → int144 downcasted
内部#
将int256转换为int144，并在溢出时进行回滚（当输入小于最小的int144或大于最大的int144时）。

与Solidity的int144操作符相对应。

要求：
* 输入必须适合144位

*自v4.7起可用。*

#### toInt136(int256 value) → int136 downcasted
内部#
将int256转换为int136，如果溢出则返回错误（当输入小于最小的int136或大于最大的int136时）。

与Solidity的int136运算符相对应。

要求：
* 输入必须适合136位

*从v4.7开始可用。*

#### toInt128(int256 value) → int128 downcasted
内部#
将int256从int128反向转换为int128，当输入小于最小int128或大于最大int128时，返回溢出（revert）。

与Solidity的int128操作符相对应。

要求：
* 输入必须适合128位

*自v3.1起可用。*

#### toInt120(int256 value) → int120 downcasted
内部#
将int256转换为int120，并在溢出时回滚（当输入小于最小int120或大于最大int120时）。

与Solidity的int120运算符相对应。

要求：
* 输入必须适合120位

*自v4.7起可用。*

#### toInt112(int256 value) → int112 downcasted
内部#
将int256向下转换为int112，并在溢出时进行回滚（当输入小于最小的int112或大于最大的int112时）。

与Solidity的int112运算符相对应。

要求：
* 输入必须适合112位

*自v4.7起可用。*

#### toInt104(int256 value) → int104 downcasted
内部#
将int256转换为int104，并在溢出时恢复（当输入小于最小int104或大于最大int104时）。

与Solidity的int104运算符相对应。

要求：
* 输入必须适合104位

从v4.7开始提供。

#### toInt96(int256 value) → int96 downcasted
内部#
将int256转换为int96，并在溢出时返回，当输入小于最小int96或大于最大int96时将回滚。

与Solidity的int96运算符相对应。

要求：
* 输入必须适合96位

从v4.7开始提供。

#### toInt88(int256 value) → int88 downcasted
内部#
从int256返回向下转型的int88，当输入小于最小的int88或大于最大的int88时会回滚（溢出）。

与Solidity的int88操作符相对应。

要求：
* 输入必须适合88位

*自v4.7开始可用。*

#### toInt80(int256 value) → int80 downcasted
内部#
将int256强制转换为int80，并在溢出时回滚（当输入小于最小的int80或大于最大的int80时）。

与Solidity的int80运算符相对应。

要求：
* 输入必须适合80位

*自v4.7以来可用。*

#### toInt72(int256 value) → int72 downcasted
内部#
将int256强制转换为int72，并在溢出时回滚（当输入小于最小int72或大于最大int72时）。

与Solidity的int72操作符相对应。

要求：
* 输入必须适合72位。

**从v4.7开始可用。**

#### toInt64(int256 value) → int64 downcasted
内部#
将int256转换为int64，并在溢出时回滚（当输入小于最小的int64或大于最大的int64时）。

与Solidity的int64操作符相对应。

要求：
* 输入必须适合64位

*自v3.1起可用。*

#### toInt56(int256 value) → int56 downcasted
内部#
将int256转换为int56，当输入小于最小int56或大于最大int56时，将返回溢出（revert）。

与Solidity的int56操作符相对应。

要求：
* 输入必须适合56位

*从v4.7开始提供。*

#### toInt48(int256 value) → int48 downcasted
内部#
将int256转换为int48，并在溢出时进行回滚（当输入小于最小int48或大于最大int48时）。

与Solidity的int48运算符相对应。

要求：
* 输入必须适合48位

*自v4.7起可用。*

#### toInt40(int256 value) → int40 downcasted
内部#
将int256转换为downcasted int40，当溢出时返回（当输入小于最小int40或大于最大int40时）。

与Solidity的int40操作符相对应。

要求：
* 输入必须适合40位

*从v4.7开始可用。*

#### toInt32(int256 value) → int32 downcasted
内部#
将int256转换为int32，如果溢出则返回错误（当输入小于最小int32或大于最大int32时）。

与Solidity的int32操作符相对应。

要求：
* 输入必须适合32位

*自v3.1起可用。*

#### toInt24(int256 value) → int24 downcasted
内部#
将int256转换为int24，如果溢出则返回revert（当输入小于最小int24或大于最大int24时）。

与Solidity的int24运算符对应。

要求：
* 输入必须适合24位。

*自v4.7起可用。*

#### toInt16(int256 value) → int16 downcasted
内部#
将int256转换为int16，并在溢出时返回，即当输入小于最小int16或大于最大int16时返回。

与Solidity的int16操作符相对应。

要求：
* 输入必须适合16位

*自v3.1起可用。*

#### toInt16(int256 value) → int16 downcasted
内部#
将int256类型的值向下转换为int16类型的值，并在溢出时返回（当输入值小于最小int16或大于最大int16时）。

这是Solidity中int16运算符的对应方法。

要求：
* 输入值必须在16位范围内

*自v3.1版本起可用。*

#### toInt8(int256 value) → int8 downcasted
内部#
将int256转换为int8，并在溢出时进行回滚（当输入小于最小int8或大于最大int8时）。

与Solidity的int8操作符相对应。

要求：
* 输入必须适合8位

*自v3.1起可用。*

#### toInt256(uint256 value) → int256
内部#
将无符号的uint256转换为有符号的int256。

要求：
* 输入必须小于或等于maxInt256。

*自v3.0起可用。*

### SafeMath
```
import "@openzeppelin/contracts/utils/math/SafeMath.sol";
```

对Solidity算术操作的包装器。

> NOTE
从Solidity 0.8开始，通常不再需要SafeMath，因为编译器现在内置了溢出检查。

**FUNCTIONS**
tryAdd(a, b)

trySub(a, b)

tryMul(a, b)

tryDiv(a, b)

tryMod(a, b)

add(a, b)

sub(a, b)

mul(a, b)

div(a, b)

mod(a, b)

sub(a, b, errorMessage)

div(a, b, errorMessage)

mod(a, b, errorMessage)

#### tryAdd(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数的加法结果，同时返回溢出标志。

*自版本3.4起可用。*

#### trySub(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数的减法结果，并带有溢出标志。

*自v3.4起可用。*

#### tryMul(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数的乘积，并带有溢出标志。

*自 v3.4 版本起可用。*

#### tryDiv(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数的除法结果，并带有除零标志。

*自v3.4版本起可用。*

#### tryMod(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数相除的余数，并标记除以零的情况。

*从v3.4版本开始可用。*

#### add(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的加法结果，当溢出时返回反向结果。

与Solidity的+运算符相对应。

要求：
* 加法不能溢出。

#### sub(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的减法结果，在溢出时返回反转的结果（当结果为负数时）。

与Solidity的-运算符相对应。

要求：
* 减法不能溢出。

#### mul(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的乘积，如果溢出则返回零。

与Solidity的*操作符相对应。

要求：
* 乘法不能溢出。

#### div(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的整数除法，如果除以零则返回反转。结果向零舍入。

与Solidity的/运算符相对应。

要求：
* 除数不能为零。

#### mod(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数相除的余数（无符号整数取模），在除以零时回退。

这个函数是 Solidity 中 % 运算符的对应函数。该函数使用了一个 revert 指令（保留剩余的 gas），而 Solidity 使用了一个无效的指令来回退（消耗所有剩余的 gas）。

要求：
* 除数不能为零。

#### sub(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数的减法结果，在溢出时使用自定义消息进行回退（当结果为负数时）。

> CAUTION
该函数已弃用，因为它不必要地需要为错误消息分配内存。对于自定义的回退原因，请使用*trySub*。
Solidity的-运算符的对应物。

要求：
* 减法不能溢出。

#### div(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数的整数除法，如果除零则返回自定义消息。结果向零舍入。

这是Solidity中/操作符的对应函数。注意：此函数使用revert操作码（保留剩余的gas），而Solidity使用无效操作码来回滚（消耗所有剩余的gas）。

要求：
* 除数不能为零。

#### mod(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数相除的余数（无符号整数取模），在除以零时返回自定义消息。

> CAUTION
该函数已被弃用，因为它不必要地为错误消息分配内存。对于自定义的还原原因，请使用*tryMod*。
与Solidity的%运算符相对应。此函数使用revert操作码（保留剩余的gas），而Solidity使用无效操作码来还原（消耗所有剩余的gas）。

要求：
* 除数不能为零。

### SignedSafeMath
```
import "@openzeppelin/contracts/utils/math/SignedSafeMath.sol";
```
Solidity的算术操作封装。

> NOTE
从Solidity 0.8开始，不再需要SignedSafeMath。编译器现在具有内置的溢出检查功能。

**FUNCTIONS**
mul(a, b)

div(a, b)

sub(a, b)

add(a, b)

#### mul(int256 a, int256 b) → int256
内部#
返回两个有符号整数的乘积，如果溢出则返回相反数。

与Solidity的*运算符相对应。

要求：
* 乘法不能溢出。

#### div(int256 a, int256 b) → int256
内部#
返回两个有符号整数的整数除法结果。在除以零时会回滚。结果向零舍入。

这是 Solidity 中 / 运算符的对应操作。

要求：
* 被除数不能为零。

#### sub(int256 a, int256 b) → int256
内部#
返回两个有符号整数的减法结果，如果溢出则返回相反数。

与Solidity的-运算符相对应。

要求：
* 减法不能溢出。

#### add(int256 a, int256 b) → int256
内部#
返回两个有符号整数的加法结果，如果溢出则返回相反数。

与Solidity的+运算符相对应。

要求：
* 加法不能溢出。

## 加密

### ECDSA
```
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
```
椭圆曲线数字签名算法（ECDSA）操作。

这些函数可用于验证消息是否由给定地址的私钥持有者签名。

**FUNCTIONS**
tryRecover(hash, signature)

recover(hash, signature)

tryRecover(hash, r, vs)

recover(hash, r, vs)

tryRecover(hash, v, r, s)

recover(hash, v, r, s)

toEthSignedMessageHash(hash)

toEthSignedMessageHash(s)

toTypedDataHash(domainSeparator, structHash)

toDataWithIntendedValidatorHash(validator, data)

#### tryRecover(bytes32 hash, bytes signature) → address, enum ECDSA.RecoverError
内部#
返回使用签名或错误字符串对哈希消息（哈希）进行签名的地址。然后可以使用该地址进行验证。

ecrecover EVM 操作码允许生成可塑（非唯一）的签名：该函数通过要求 s 值位于低半序列中，并且 v 值为 27 或 28 来拒绝这些签名。

> IMPORTANT
为了确保验证的安全性，哈希必须是用于验证的哈希操作的结果：可以为非哈希数据制作恢复到任意地址的签名。确保此安全性的一种方法是接收原始消息的哈希（否则可能太长），然后对其调用 *toEthSignedMessageHash*。
签名生成的文档：- 使用 [Web3.js](https://web3js.readthedocs.io/en/v1.3.4/web3-eth-accounts.html#sign) - 使用 [ethers](https://docs.ethers.io/v5/api/signer/#Signer-signMessage)

*自 v4.3 起可用。*

#### recover(bytes32 hash, bytes signature) → address
内部#
返回使用签名对哈希消息（hash）进行签名的地址。可以将此地址用于验证目的。

ecrecover EVM操作码允许可塑（非唯一）签名：该函数通过要求s值在低半序中，并且v值为27或28来拒绝它们。

> IMPORTANT
为了确保验证安全，哈希必须是对原始消息进行哈希操作的结果：可以通过接收原始消息的哈希（否则可能过长），然后对其调用*toEthSignedMessageHash*来确保这一点。

#### tryRecover(bytes32 hash, bytes32 r, bytes32 vs) → address, enum ECDSA.RecoverError
内部#
*ECDSA.tryRecover*的重载函数，接收r和vs字段作为独立参数。

参考[EIP-2098的短签名](https://eips.ethereum.org/EIPS/eip-2098)。

*自v4.3版本开始可用。*

#### recover(bytes32 hash, bytes32 r, bytes32 vs) → address
内部#
*ECDSA.recover*的重载版本，可以分别接收r和vs字段的短签名。

*自v4.2版本起可用。*

#### tryRecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) → address, enum ECDSA.RecoverError
内部#
*ECDSA.tryRecover*的重载版本，接收单独的v、r和s签名字段。

*自v4.3版本开始提供。*

#### recover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) → address
内部#
*ECDSA.recover*的重载版本，接收单独的v、r和s签名字段。

#### toEthSignedMessageHash(bytes32 hash) → bytes32 message
内部#
返回一个以哈希值创建的以太坊已签名消息。这个方法会产生与使[用eth_sign](https://eth.wiki/json-rpc/API#eth_sign) JSON-RPC方法签名的哈希相对应的哈希值，作为EIP-191的一部分。

参见*recover*。

#### toEthSignedMessageHash(bytes s) → bytes32
内部#
返回一个以s创建的以太坊签名消息。这将产生与使用[eth_sign](https://eth.wiki/json-rpc/API#eth_sign) JSON-RPC方法签名的哈希相对应，作为EIP-191的一部分。

参见 *recove*r。

#### toTypedDataHash(bytes32 domainSeparator, bytes32 structHash) → bytes32 data
内部#
返回一个以域分隔符（domainSeparator）和结构哈希（structHash）创建的以太坊签名类型数据。这个方法生成的哈希与使用[eth_signTypedData](https://eips.ethereum.org/EIPS/eip-712) JSON-RPC方法作为EIP-712的一部分进行签名的哈希相对应。

参见*recover*。

#### toDataWithIntendedValidatorHash(address validator, bytes data) → bytes32
内部#
根据EIP-191版本0，从验证器和数据创建一个带有预期验证者的以太坊签名数据。

请参见*recover*。

### SignatureChecker
```
import "@openzeppelin/contracts/utils/cryptography/SignatureChecker.sol";
```
签名验证助手，可用于无缝支持来自外部拥有账户（EOAs）的ECDSA签名和来自智能合约钱包（如Argent和Gnosis Safe）的ERC1271签名。

*自v4.1版本开始提供。*

*FUNCTIONS*
isValidSignatureNow(signer, hash, signature)

isValidERC1271SignatureNow(signer, hash, signature)

#### isValidSignatureNow(address signer, bytes32 hash, bytes signature) → bool
内部#
检查给定签名者和数据哈希是否有效。如果签名者是智能合约，则使用ERC1271对该智能合约进行验证签名，否则使用ECDSA.recover进行验证。

> NOTE
与ECDSA签名不同，合约签名是可撤销的，因此该函数的结果可能随时间而变化。它可以在第N个区块返回true，在第N+1个区块返回false（或反之）。

#### isValidERC1271SignatureNow(address signer, bytes32 hash, bytes signature) → bool
检查给定签名者和数据哈希是否有效。该签名将使用ERC1271验证签名者智能合约。

> NOTE
与ECDSA签名不同，合约签名是可撤销的，因此此函数的结果可能会随时间变化。它可以在N块返回true，并在N+1块返回false（或相反）。

### MerkleProof
```
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
```
这些函数处理Merkle树证明的验证。

树和证明可以使用我们的[JavaScript库](https://github.com/OpenZeppelin/merkle-tree)生成。您可以在自述文件中找到快速入门指南。

> WARNING
在进行哈希之前，应避免使用长度为64字节的叶值，或者使用除keccak256之外的哈希函数进行哈希。这是因为Merkle树中内部节点的排序对的拼接可能会被重新解释为叶值。OpenZeppelin的JavaScript库可以直接生成抵御此攻击的Merkle树。

**FUNCTIONS**
verify(proof, root, leaf)

verifyCalldata(proof, root, leaf)

processProof(proof, leaf)

processProofCalldata(proof, leaf)

multiProofVerify(proof, proofFlags, root, leaves)

multiProofVerifyCalldata(proof, proofFlags, root, leaves)

processMultiProof(proof, proofFlags, leaves)

processMultiProofCalldata(proof, proofFlags, leaves)

#### verify(bytes32[] proof, bytes32 root, bytes32 leaf) → bool
内部#
如果可以通过提供包含从叶子到树根的分支上的兄弟哈希的证明来证明叶子是Merkle树的一部分，则返回true。假设每对叶子和每对原始图像都已排序。

#### verifyCalldata(bytes32[] proof, bytes32 root, bytes32 leaf) → bool
内部#
verify的Calldata版本

*从v4.7版本开始可用。*

#### processProof(bytes32[] proof, bytes32 leaf) → bytes32
内部#
从叶节点开始使用证明，通过遍历 Merkle 树返回重建的哈希值。只有当重建的哈希值与树的根节点匹配时，证明才有效。在处理证明时，假设叶节点和预映像的配对已经排序。

*自 v4.4 版本起可用。*

#### processProofCalldata(bytes32[] proof, bytes32 leaf) → bytes32
内部#
processProof的Calldata版本

*自v4.7版本开始可用。*

#### multiProofVerify(bytes32[] proof, bool[] proofFlags, bytes32 root, bytes32[] leaves) → bool
内部#
如果根据processMultiProof中的proof和proofFlags描述，可以同时*证明叶子节点是merkle树*的一部分，则返回true。

并非所有的merkle树都支持多证明。详情请参见*processMultiProof*。

*自v4.7版本开始可用。*

#### multiProofVerifyCalldata(bytes32[] proof, bool[] proofFlags, bytes32 root, bytes32[] leaves) → bool
内部#
*multiProofVerify*的Calldata版本

并非所有的Merkle树都支持多重验证。有关详细信息，请参阅*processMultiProof*。
**自v4.7起可用。**

#### processMultiProof(bytes32[] proof, bool[] proofFlags, bytes32[] leaves) → bytes32 merkleRoot
内部#
从叶子节点和兄弟节点的证明中返回重建的树的根节点。重建过程是通过将叶子节点/内部节点与另一个叶子节点/内部节点或证明兄弟节点组合来逐步重建所有内部节点，具体取决于每个proofFlags项是true还是false。

> CAUTION
并非所有的默克尔树都支持多证明。要使用多证明，只需确保：1）树是完整的（但不一定是完美的），2）要被证明的叶子节点的顺序与它们在树中的顺序相反（即，从最深的层开始从右向左看，然后继续下一层）。

*自v4.7起可用。*

#### processMultiProofCalldata(bytes32[] proof, bool[] proofFlags, bytes32[] leaves) → bytes32 merkleRoot
*processMultiProof*的calldata版本。

> CAUTION
并非所有的默克尔树都支持多重证明。有关详细信息，请参阅*processMultiProof*。
*自v4.7版本开始提供。*

### EIP712
```
import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";
```
[EIP 712](https://eips.ethereum.org/EIPS/eip-712)是一种用于对类型化结构化数据进行哈希和签名的标准。

EIP中指定的编码非常通用，Solidity中的这种通用实现是不可行的，因此该合约不实现编码本身。协议需要使用abi.encode和keccak256的组合来在其合约中实现所需的特定类型编码。

该合约实现了作为编码方案的一部分使用的EIP 712域分隔符（*_domainSeparatorV4*），以及获取消息摘要后通过ECDSA进行签名的编码的最后一步（*_hashTypedDataV4*）。

域分隔符的实现旨在尽可能高效，同时仍然正确地更新链ID，以防止在未来的链分叉上进行重放攻击。

> NOTE
该合约实现了被称为“v4”的编码版本，由[MetaMask中](https://docs.metamask.io/guide/signing-data.html)的JSON RPC方法[eth_signTypedDataV4](https://docs.metamask.io/guide/signing-data.html)实现。

> NOTE
在可升级版本的合约中，缓存的值将对应于实现合约的地址和域分隔符。这将导致_domainSeparatorV4函数始终从不可变值重新构建分隔符，这比访问冷存储中的缓存版本更便宜。

*自v3.4版本起可用。*

**FUNCTIONS**
constructor(name, version)

_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

**EVENTS**
IERC5267
EIP712DomainChanged()

#### constructor(string name, string version)
内部#
初始化域分隔符和参数缓存。

名称和版本的含义在[EIP 712](https://eips.ethereum.org/EIPS/eip-712#definition-of-domainseparator)中指定：
* 名称：签名域的用户可读名称，即DApp或协议的名称。
* 版本：签名域的当前主要版本。

> NOTE
除非通过*智能合约升级*，否则这些参数不能更改。

#### _domainSeparatorV4() → bytes32
内部#
返回当前链的域分隔符。

#### _hashTypedDataV4(bytes32 structHash) → bytes32
内部#
给定一个已经[哈希的结构体](https://eips.ethereum.org/EIPS/eip-712#definition-of-hashstruct)，这个函数返回完全编码的 EIP712 消息在该域中的哈希值。

这个哈希值可以与 *ECDSA.recover* 一起使用，以获得消息的签名者。例如：
```
bytes32 digest = _hashTypedDataV4(keccak256(abi.encode(
    keccak256("Mail(address to,string contents)"),
    mailTo,
    keccak256(bytes(mailContents))
)));
address signer = ECDSA.recover(digest, signature);
```

#### eip712Domain() → bytes1 fields, string name, string version, uint256 chainId, address verifyingContract, bytes32 salt, uint256[] extensions
公开#
请参阅{EIP-5267}。

*自v4.9版本起可用。*

## Escrow

### ConditionalEscrow
```
import "@openzeppelin/contracts/utils/escrow/ConditionalEscrow.sol";
```
基于抽象的托管只允许在满足条件时进行提款。预期用法：请参见*Escrow*。相同的用法指南适用于此处。

**FUNCTIONS**
withdrawalAllowed(payee)

withdraw(payee)

ESCROW
depositsOf(payee)

deposit(payee)

OWNABLE
owner()

_checkOwner()

renounceOwnership()

transferOwnership(newOwner)

_transferOwnership(newOwner)

**EVENTS**
ESCROW
Deposited(payee, weiAmount)

Withdrawn(payee, weiAmount)

OWNABLE
OwnershipTransferred(previousOwner, newOwner)

#### withdrawalAllowed(address payee) → bool
公开#
返回地址是否被允许提取资金。由派生合约实现。

#### withdraw(address payable payee)
公开#
将累积余额提取给收款人，并将所有燃气转发给收款人。

> WARNING
将所有燃气转发给收款人会打开重入漏洞的大门。请确保您信任收款人，或者遵循检查-效果-交互模式或使用*ReentrancyGuard*。

### Escrow
```
import "@openzeppelin/contracts/utils/escrow/Escrow.sol";
```
基本的托管合约，将款项保留给收款人，直到他们提取这些款项。

预期使用方式：该合约（以及派生的托管合约）应该是一个独立的合约，只与实例化它的合约进行交互。这样，可以确保所有以太币都按照托管规则进行处理，而不需要在继承树中检查可支付的函数或转账。使用托管作为付款方式的合约应该是其所有者，并提供公共方法重定向到托管的存款和提款。

**FUNCTIONS**
depositsOf(payee)

deposit(payee)

withdraw(payee)

OWNABLE
owner()

_checkOwner()

renounceOwnership()

transferOwnership(newOwner)

_transferOwnership(newOwner)

**EVENTS**
Deposited(payee, weiAmount)

Withdrawn(payee, weiAmount)

OWNABLE
OwnershipTransferred(previousOwner, newOwner)

#### depositsOf(address payee) → uint256
公开#

#### deposit(address payee)
公开#
将发送的金额存为信用以便提取。