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