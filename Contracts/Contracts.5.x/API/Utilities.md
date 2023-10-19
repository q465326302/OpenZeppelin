# Utilities
这些是一些杂项合约和库，包含了一些实用的功能，你可以使用它们来提高安全性，处理新的数据类型，或者安全地使用低级原语。

* *ReentrancyGuard*: 这是一个修饰符，可以在某些函数中防止重入。

* *Pausable*: 这是一个常见的应急响应机制，可以在等待补救措施期间暂停功能。

* *SafeCast*: 这是一种经过检查的向下转型函数，可以避免静默截断。

* *Math*, *SignedMath*: 这是各种算术函数的实现。

* *Multicall*: 这是一种简单的方式，可以在一个外部调用中批量处理多个调用。

* *Create2*: 这是围绕[CREATE2 EVM操作码](https://blog.openzeppelin.com/getting-the-most-out-of-create2/)的包装器，可以安全地使用，无需处理低级汇编。

*EnumerableMap*: 这是一种类似于Solidity的[映射类型](https://solidity.readthedocs.io/en/latest/types.html#mapping-types)，但具有键值枚举：这将让你知道一个映射有多少个条目，并迭代它们（这是映射无法做到的）。

*EnumerableSet*: 类似于*EnumerableMap*，但用于[集合](https://en.wikipedia.org/wiki/Set_(abstract_data_type))。可以用来存储特权账户，发出的ID等。

> NOTE
因为Solidity不支持通用类型，所以*EnumerableMap*和*EnumerableSet*专门用于有限的键值类型。

## Math

### [Math](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/math/Math.sol)
```
import "@openzeppelin/contracts/utils/math/Math.sol";
```

这是在Solidity语言中缺失的标准数学工具。

**FUNCTIONS**
tryAdd(a, b)

trySub(a, b)

tryMul(a, b)

tryDiv(a, b)

tryMod(a, b)

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

unsignedRoundsUp(rounding)

**ERRORS**
MathOverflowedMulDiv()

#### tryAdd(uint256 a, uint256 b) → bool, uint256
internal#
返回两个无符号整数的加法，带有溢出标志。

#### trySub(uint256 a, uint256 b) → bool, uint256
internal#
返回两个无符号整数的减法，带有溢出标志。

#### tryMul(uint256 a, uint256 b) → bool, uint256
internal#
返回两个无符号整数的乘积，带有溢出标志。

#### tryDiv(uint256 a, uint256 b) → bool, uint256
internal#
返回两个无符号整数的除法，带有零除标志。

#### tryMod(uint256 a, uint256 b) → bool, uint256
internal#
返回两个无符号整数除法的余数，带有零除标志。

#### max(uint256 a, uint256 b) → uint256
internal#
返回两个数字中的最大值。

#### min(uint256 a, uint256 b) → uint256
internal#
返回两个数字中的最小值。

#### average(uint256 a, uint256 b) → uint256
internal#
返回两个数字的平均值。结果向零舍入。

#### ceilDiv(uint256 a, uint256 b) → uint256
internal#
返回两个数字除法的上限。

这与使用/的标准除法不同，它是向无穷大方向取整，而不是向零方向取整。

#### mulDiv(uint256 x, uint256 y, uint256 denominator) → uint256 result
internal#
原创版权归Remco Bloemen所有，遵循MIT许可证（https://xn—​2-umb.com/21/muldiv），Uniswap Labs进行了进一步的编辑，也遵循MIT许可证。

#### mulDiv(uint256 x, uint256 y, uint256 denominator, enum Math.Rounding rounding) → uint256
internal#

#### sqrt(uint256 a) → uint256
internal#
返回一个数的平方根。如果这个数不是完全平方数，那么值将向零舍入。

灵感来源于Henry S. Warren, Jr.的《Hacker’s Delight》（第11章）。

#### sqrt(uint256 a, enum Math.Rounding rounding) → uint256
internal#

#### log2(uint256 value) → uint256
internal#
返回以2为底的正值的对数，向零取整。如果给定的值为0，则返回0。

#### log2(uint256 value, enum Math.Rounding rounding) → uint256
internal#
返回正值的以2为底的对数，按照选择的舍入方向进行舍入。如果给定的值为0，则返回0。

#### log10(uint256 value) → uint256
internal#
返回向零舍入的正值的10为底的对数。如果给定0，则返回0。

#### log10(uint256 value, enum Math.Rounding rounding) → uint256
internal#
返回正值的以10为底的对数，按照选定的舍入方向进行舍入。如果给定的值为0，则返回0。

#### log256(uint256 value) → uint256
internal#
返回以256为底的正值对数，并向零舍入。如果给定的值为0，则返回0。

将结果加一，得到的是表示值为十六进制字符串所需的十六进制符号对的数量。

#### log256(uint256 value, enum Math.Rounding rounding) → uint256
internal#
返回给定正值的以256为底的对数，根据选择的舍入方向进行舍入。如果给定的值为0，则返回0。

#### unsignedRoundsUp(enum Math.Rounding rounding) → bool
internal#
返回提供的舍入模式是否被认为是对无符号整数进行向上舍入。

#### MathOverflowedMulDiv()
error#
乘除运算溢出。

### SignedMath
```
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/math/SignedMath.sol
```

标准的有符号数学工具在Solidity语言中缺失。

**FUNCTIONS**
max(a, b)

min(a, b)

average(a, b)

abs(n)

#### max(int256 a, int256 b) → int256
internal#
返回两个有符号数字中的最大值。

#### min(int256 a, int256 b) → int256
internal#
返回两个有符号数字中的最小值。

#### average(int256 a, int256 b) → int256
internal#
返回两个有符号数字的平均值，不会溢出。结果向零舍入。

#### abs(int256 n) → uint256
internal#
返回有符号值的绝对无符号值。

### [SafeCast](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/math/SafeCast.sol)
```
import "@openzeppelin/contracts/utils/math/SafeCast.sol";
```

在Solidity中，从uint256/int256进行下转型并不会在溢出时反转。这很容易导致不期望的利用或错误，因为开发人员通常假设溢出会引发错误。SafeCast通过在此类操作溢出时反转交易来恢复这种直觉。

使用这个库而不是未经检查的操作可以消除一整类的错误，因此建议始终使用它。

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

**ERRORS**
SafeCastOverflowedUintDowncast(bits, value)

SafeCastOverflowedIntToUint(value)

SafeCastOverflowedIntDowncast(bits, value)

SafeCastOverflowedUintToInt(value)

#### toUint248(uint256 value) → uint248
internal#
返回从uint256降级的uint248，当输入大于最大的uint248时溢出（撤销）。

与Solidity的uint248运算符相对应。

要求：
* 输入必须适应248位

#### toUint240(uint256 value) → uint240
internal#
返回从uint256降级到uint240的值，如果溢出（当输入大于最大的uint240时）则恢复。

这是Solidity的uint240运算符的对应部分。

要求：
* 输入必须适合240位。

#### toUint232(uint256 value) → uint232
internal#
返回从uint256降级的uint232，当输入大于最大的uint232时溢出（回滚）。

这与Solidity的uint232运算符相对应。

要求：
* 输入必须适应232位。

#### toUint224(uint256 value) → uint224
internal#
返回从uint256降级的uint224，当输入大于最大的uint224时，会发生溢出。

这是Solidity的uint224运算符的对应部分。

要求：
* 输入必须适合224位。

#### toUint216(uint256 value) → uint216
internal#
返回从uint256降级的uint216，当输入大于最大的uint216时溢出（撤销）。

与Solidity的uint216运算符相对应。

要求：
* 输入必须适合216位。

#### toUint208(uint256 value) → uint208
internal#
返回从uint256降级到uint208的值，当输入大于最大的uint208时会发生溢出。

这与Solidity的uint208运算符相对应。

要求：
* 输入必须适应208位。

#### toUint200(uint256 value) → uint200
internal#
将uint256降级为uint200，如果溢出（当输入大于最大的uint200时）则返回。

这是Solidity的uint200运算符的对应部分。

要求：
* 输入必须适应200位

#### toUint192(uint256 value) → uint192
internal#
将uint256降级为uint192，如果溢出（当输入大于最大的uint192时）则返回。

这是Solidity的uint192运算符的对应部分。

要求：
* 输入必须适合192位。

#### toUint184(uint256 value) → uint184
internal#
将uint256降级为uint184，如果溢出（当输入大于最大的uint184时）则返回。

这是Solidity的uint184运算符的对应部分。

要求：
* 输入必须适合184位。

#### toUint176(uint256 value) → uint184
internal#
将uint256降级为uint176，如果溢出（当输入大于最大的uint176时）则返回。

这是Solidity的uint176运算符的对应部分。

要求：
* 输入必须适合176位。

#### toUint168(uint256 value) → uint184
internal#
将uint256降级为uint168，如果溢出（当输入大于最大的uint168时）则返回。

这是Solidity的uint168运算符的对应部分。

要求：
* 输入必须适合168位。

#### toUint160(uint256 value) → uint184
internal#
将uint256降级为uint160，如果溢出（当输入大于最大的uint160时）则返回。

这是Solidity的uint160运算符的对应部分。

要求：
* 输入必须适合160位。

#### toUint152(uint256 value) → uint184
internal#
将uint256降级为uint152，如果溢出（当输入大于最大的uint152时）则返回。

这是Solidity的uint152运算符的对应部分。

要求：
* 输入必须适合152位。

#### toUint144(uint256 value) → uint184
internal#
将uint256降级为uint144，如果溢出（当输入大于最大的uint144时）则返回。

这是Solidity的uint144运算符的对应部分。

要求：
* 输入必须适合144位。

#### toUint136(uint256 value) → uint184
internal#
将uint256降级为uint136，如果溢出（当输入大于最大的uint136时）则返回。

这是Solidity的uint136运算符的对应部分。

要求：
* 输入必须适合136位。

#### toUint128(uint256 value) → uint184
internal#
将uint256降级为uint128，如果溢出（当输入大于最大的uint128时）则返回。

这是Solidity的uint128运算符的对应部分。

要求：
* 输入必须适合128位。

#### toUint120(uint256 value) → uint184
internal#
将uint256降级为uint120，如果溢出（当输入大于最大的uint120时）则返回。

这是Solidity的uint120运算符的对应部分。

要求：
* 输入必须适合120位。

#### toUint112(uint256 value) → uint184
internal#
将uint256降级为uint112，如果溢出（当输入大于最大的uint112时）则返回。

这是Solidity的uint112运算符的对应部分。

要求：
* 输入必须适合112位。

#### toUint104(uint256 value) → uint184
internal#
将uint256降级为uint104，如果溢出（当输入大于最大的uint104时）则返回。

这是Solidity的uint104运算符的对应部分。

要求：
* 输入必须适合104位。

#### toUint96(uint256 value) → uint184
internal#
将uint256降级为uint96，如果溢出（当输入大于最大的uint96时）则返回。

这是Solidity的uint96运算符的对应部分。

要求：
* 输入必须适合96位。

#### toUint88(uint256 value) → uint184
internal#
将uint256降级为uint88，如果溢出（当输入大于最大的uint88时）则返回。

这是Solidity的uint88运算符的对应部分。

要求：
* 输入必须适合88位。

#### toUint80(uint256 value) → uint184
internal#
将uint256降级为uint80，如果溢出（当输入大于最大的uint80时）则返回。

这是Solidity的uint80运算符的对应部分。

要求：
* 输入必须适合80位。

#### toUint72(uint256 value) → uint184
internal#
将uint256降级为uint72，如果溢出（当输入大于最大的uint72时）则返回。

这是Solidity的uint72运算符的对应部分。

要求：
* 输入必须适合72位。

#### toUint64(uint256 value) → uint184
internal#
将uint256降级为uint64，如果溢出（当输入大于最大的uint64时）则返回。

这是Solidity的uint64运算符的对应部分。

要求：
* 输入必须适合64位。

#### toUint56(uint256 value) → uint184
internal#
将uint256降级为uint56，如果溢出（当输入大于最大的uint56时）则返回。

这是Solidity的uint56运算符的对应部分。

要求：
* 输入必须适合56位。

#### toUint48(uint256 value) → uint184
internal#
将uint256降级为uint48，如果溢出（当输入大于最大的uint48时）则返回。

这是Solidity的uint48运算符的对应部分。

要求：
* 输入必须适合48位。

#### toUint40(uint256 value) → uint184
internal#
将uint256降级为uint40，如果溢出（当输入大于最大的uint40时）则返回。

这是Solidity的uint40运算符的对应部分。

要求：
* 输入必须适合40位。

#### toUint32(uint256 value) → uint184
internal#
将uint256降级为uint32，如果溢出（当输入大于最大的uint32时）则返回。

这是Solidity的uint32运算符的对应部分。

要求：
* 输入必须适合32位。

#### toUint24(uint256 value) → uint184
internal#
将uint256降级为uint24，如果溢出（当输入大于最大的uint24时）则返回。

这是Solidity的uint24运算符的对应部分。

要求：
* 输入必须适合24位。

#### toUint16(uint256 value) → uint184
internal#
将uint256降级为uint16，如果溢出（当输入大于最大的uint16时）则返回。

这是Solidity的uint16运算符的对应部分。

要求：
* 输入必须适合16位。

#### toUint8(uint256 value) → uint184
internal#
将uint256降级为uint8，如果溢出（当输入大于最大的uint8时）则返回。

这是Solidity的uint8运算符的对应部分。

要求：
* 输入必须适合8位。

#### toUint256(int256 value) → uint256
internal#
将有符号的int256转换为无符号的uint256。

要求：
* 输入必须大于或等于0。

#### toInt248(int256 value) → int248 downcasted
internal#
将int256转换为int248，如果溢出（当输入小于最小的int248或大于最大的int248）则返回。这是Solidity的int248运算符的对应物。

要求：
* 输入必须适合248位

#### toInt240(int256 value) → int240 downcasted
internal#
返回从int256降级的int240，当输入小于最小的int240或大于最大的int240时，将会溢出并恢复。

这是Solidity的int240运算符的对应部分。

要求：
* 输入必须适应240位。

#### toInt232(int256 value) → int232 downcasted
internal#
返回从int256降级的int232，当输入小于最小的int232或大于最大的int232时，将会溢出并恢复。

这是Solidity的int232运算符的对应部分。

要求：
* 输入必须适应232位。

#### toInt224(int256 value) → int224 downcasted
internal#
返回从int256降级的int224，当输入小于最小的int224或大于最大的int224时，将会溢出并恢复。

这是Solidity的int224运算符的对应部分。

要求：
* 输入必须适应224位。

#### toInt216(int256 value) → int216 downcasted
internal#
返回从int256降级的int216，当输入小于最小的int216或大于最大的int216时，将会溢出并恢复。

这是Solidity的int216运算符的对应部分。

要求：
* 输入必须适应216位。

#### toInt208(int256 value) → int208 downcasted
internal#
返回从int256降级的int208，当输入小于最小的int208或大于最大的int208时，将会溢出并恢复。

这是Solidity的int208运算符的对应部分。

要求：
* 输入必须适应208位。

#### toInt200(int256 value) → int200 downcasted
internal#
返回从int256降级的int200，当输入小于最小的int200或大于最大的int200时，将会溢出并恢复。

这是Solidity的int200运算符的对应部分。

要求：
* 输入必须适应200位。

#### toInt192(int256 value) → int192 downcasted
internal#
返回从int256降级的int192，当输入小于最小的int192或大于最大的int192时，将会溢出并恢复。

这是Solidity的int192运算符的对应部分。

要求：
* 输入必须适应192位。

#### toInt184(int256 value) → int184 downcasted
internal#
返回从int256降级的int184，当输入小于最小的int184或大于最大的int184时，将会溢出并恢复。

这是Solidity的int184运算符的对应部分。

要求：
* 输入必须适应184位。

#### toInt176(int256 value) → int176 downcasted
internal#
返回从int256降级的int176，当输入小于最小的int176或大于最大的int176时，将会溢出并恢复。

这是Solidity的int176运算符的对应部分。

要求：
* 输入必须适应176位。

#### toInt168(int256 value) → int168 downcasted
internal#
返回从int256降级的int168，当输入小于最小的int168或大于最大的int168时，将会溢出并恢复。

这是Solidity的int168运算符的对应部分。

要求：
* 输入必须适应168位。

#### toInt160(int256 value) → int160 downcasted
internal#
返回从int256降级的int160，当输入小于最小的int160或大于最大的int160时，将会溢出并恢复。

这是Solidity的int160运算符的对应部分。

要求：
* 输入必须适应160位。

#### toInt152(int256 value) → int152 downcasted
internal#
返回从int256降级的int152，当输入小于最小的int152或大于最大的int152时，将会溢出并恢复。

这是Solidity的int152运算符的对应部分。

要求：
* 输入必须适应152位。

#### toInt144(int256 value) → int144 downcasted
internal#
返回从int256降级的int144，当输入小于最小的int144或大于最大的int144时，将会溢出并恢复。

这是Solidity的int144运算符的对应部分。

要求：
* 输入必须适应144位。

#### toInt136(int256 value) → int136 downcasted
internal#
返回从int256降级的int136，当输入小于最小的int136或大于最大的int136时，将会溢出并恢复。

这是Solidity的int136运算符的对应部分。

要求：
* 输入必须适应136位。

#### toInt128(int256 value) → int128 downcasted
internal#
返回从int256降级的int128，当输入小于最小的int128或大于最大的int128时，将会溢出并恢复。

这是Solidity的int128运算符的对应部分。

要求：
* 输入必须适应128位。

#### toInt120(int256 value) → int120 downcasted
internal#
返回从int256降级的int120，当输入小于最小的int120或大于最大的int120时，将会溢出并恢复。

这是Solidity的int120运算符的对应部分。

要求：
* 输入必须适应120位。

#### toInt112(int256 value) → int112 downcasted
internal#
返回从int256降级的int112，当输入小于最小的int112或大于最大的int112时，将会溢出并恢复。

这是Solidity的int112运算符的对应部分。

要求：
* 输入必须适应112位。

#### toInt104(int256 value) → int104 downcasted
internal#
返回从int256降级的int104，当输入小于最小的int104或大于最大的int104时，将会溢出并恢复。

这是Solidity的int104运算符的对应部分。

要求：
* 输入必须适应104位。

#### toInt96(int256 value) → int96 downcasted
internal#
返回从int256降级的int96，当输入小于最小的int96或大于最大的int96时，将会溢出并恢复。

这是Solidity的int96运算符的对应部分。

要求：
* 输入必须适应96位。

#### toInt88(int256 value) → int88 downcasted
internal#
返回从int256降级的int88，当输入小于最小的int88或大于最大的int88时，将会溢出并恢复。

这是Solidity的int88运算符的对应部分。

要求：
* 输入必须适应88位。

#### toInt80(int256 value) → int80 downcasted
internal#
返回从int256降级的int80，当输入小于最小的int80或大于最大的int80时，将会溢出并恢复。

这是Solidity的int80运算符的对应部分。

要求：
* 输入必须适应80位。

#### toInt72(int256 value) → int72 downcasted
internal#
返回从int256降级的int72，当输入小于最小的int72或大于最大的int72时，将会溢出并恢复。

这是Solidity的int72运算符的对应部分。

要求：
* 输入必须适应72位。

#### toInt64(int256 value) → int64 downcasted
internal#
返回从int256降级的int64，当输入小于最小的int64或大于最大的int64时，将会溢出并恢复。

这是Solidity的int64运算符的对应部分。

要求：
* 输入必须适应64位。

#### toInt56(int256 value) → int56 downcasted
internal#
返回从int256降级的int56，当输入小于最小的int56或大于最大的int56时，将会溢出并恢复。

这是Solidity的int56运算符的对应部分。

要求：
* 输入必须适应56位。

#### toInt48(int256 value) → int48 downcasted
internal#
返回从int256降级的int48，当输入小于最小的int48或大于最大的int48时，将会溢出并恢复。

这是Solidity的int48运算符的对应部分。

要求：
* 输入必须适应48位。

#### toInt40(int256 value) → int40 downcasted
internal#
返回从int256降级的int40，当输入小于最小的int40或大于最大的int40时，将会溢出并恢复。

这是Solidity的int40运算符的对应部分。

要求：
* 输入必须适应40位。

#### toInt32(int256 value) → int32 downcasted
internal#
返回从int256降级的int32，当输入小于最小的int32或大于最大的int32时，将会溢出并恢复。

这是Solidity的int32运算符的对应部分。

要求：
* 输入必须适应32位。

#### toInt24(int256 value) → int24 downcasted
internal#
返回从int256降级的int24，当输入小于最小的int24或大于最大的int24时，将会溢出并恢复。

这是Solidity的int24运算符的对应部分。

要求：
* 输入必须适应24位。

#### toInt16(int256 value) → int16 downcasted
internal#
返回从int256降级的int16，当输入小于最小的int16或大于最大的int16时，将会溢出并恢复。

这是Solidity的int16运算符的对应部分。

要求：
* 输入必须适应16位。

#### toInt8(int256 value) → int8 downcasted
internal#
返回从int256降级的int8，当输入小于最小的int8或大于最大的int8时，将会溢出并恢复。

这是Solidity的int8运算符的对应部分。

要求：
* 输入必须适应8位。

#### toInt256(uint256 value) → int256
internal#
将无符号uint256转换为有符号int256。

要求：
* 输入必须小于或等于maxInt256。

#### SafeCastOverflowedUintDowncast(uint8 bits, uint256 value)
error#
值不适合在位大小的uint中。

#### SafeCastOverflowedIntToUint(int256 value)
error#
一个int值不能适应位大小的uint。

#### SafeCastOverflowedIntDowncast(uint8 bits, int256 value)
error#
值不适合位大小的整数。

#### SafeCastOverflowedUintToInt(uint256 value)
error#
一个无符号整数值不适合在位大小的整数中。

## Cryptography

### [ECDSA](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/cryptography/ECDSA.sol)
```
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
```

椭圆曲线数字签名算法ECDSA操作。

这些功能可以用来验证消息是否由给定地址的私钥持有者签名。

**FUNCTIONS**
tryRecover(hash, signature)

recover(hash, signature)

tryRecover(hash, r, vs)

recover(hash, r, vs)

tryRecover(hash, v, r, s)

recover(hash, v, r, s)

**ERRORS**
ECDSAInvalidSignature()

ECDSAInvalidSignatureLength(length)

ECDSAInvalidSignatureS(s)

#### tryRecover(bytes32 hash, bytes signature) → address, enum ECDSA.RecoverError, bytes32
internal#
返回签署了带有签名的哈希消息（哈希）的地址，或者返回一个错误。如果没有错误，那么这个地址可以用于验证目的。

这个函数通过要求s值在较低的半序中，以及v值为27或28，来拒绝ecrecover EVM预编译允许的易变（非唯一）的签名。

> IMPORTANT
哈希必须是哈希操作的结果，验证才能安全：对于非哈希数据，可以制作出恢复到任意地址的签名。确保这一点的安全方式是接收原始消息的哈希（否则可能太长），然后在其上调用*MessageHashUtils.toEthSignedMessageHash*。

签名生成的文档：- 使用[Web3.js](https://web3js.readthedocs.io/en/v1.3.4/web3-eth-accounts.html#sign) - 使用[ethers](https://docs.ethers.io/v5/api/signer/#Signer-signMessage)。

#### recover(bytes32 hash, bytes signature) → address
internal#
返回签署了哈希消息（哈希）的地址。然后，可以用这个地址进行验证。

ecrecover EVM 预编译允许可变形（非唯一）的签名：此函数通过要求 s 值在较低的半序中，以及 v 值为27或28来拒绝它们。

> IMPORTANT
哈希必须是哈希操作的结果，验证才能安全：对于非哈希数据，可以制作出恢复到任意地址的签名。确保这一点的安全方式是接收原始消息的哈希（否则可能太长），然后在其上调用*MessageHashUtils.toEthSignedMessageHash*。

#### tryRecover(bytes32 hash, bytes32 r, bytes32 vs) → address, enum ECDSA.RecoverError, bytes32
internal#
*ECDSA.tryRecover*的重载，分别接收r和vs短签名字段。

参见[EIP-2098短签名](https://eips.ethereum.org/EIPS/eip-2098)

#### recover(bytes32 hash, bytes32 r, bytes32 vs) → address
internal#
重载*ECDSA.recover*，分别接收r和`vs短签名字段。

#### recover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) → address
internal#
对*ECDSA.recover*的重载，分别接收v，r和s签名字段。

#### ECDSAInvalidSignature()
error#
签名导出地址(0)。

#### ECDSAInvalidSignatureLength(uint256 length)
error#
签名的长度无效。

#### ECDSAInvalidSignatureS(bytes32 s)
error#
签名中的S值位于上半顺序中。

### [MessageHashUtils](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/cryptography/MessageHashUtils.sol)
```
import "@openzeppelin/contracts/utils/cryptography/MessageHashUtils.sol";
```
用于生成由ECDSA恢复或签名消费的摘要的签名消息哈希实用程序。
此库提供了生成符合[EIP 191](https://eips.ethereum.org/EIPS/eip-191)和[EIP 712](https://eips.ethereum.org/EIPS/eip-712)规范的消息哈希的方法，用于生产由ECDSA恢复或签名消费的摘要。

**FUNCTIONS**
toEthSignedMessageHash(messageHash)

toEthSignedMessageHash(message)

toDataWithIntendedValidatorHash(validator, data)

toTypedDataHash(domainSeparator, structHash)

#### toEthSignedMessageHash(bytes32 messageHash) → bytes32 digest
internal#
返回版本为0x45（personal_sign消息）的EIP-191签名数据的keccak256摘要。

通过在bytes32 messageHash前加上"\x19Ethereum Signed Message:\n32"并对结果进行哈希运算来计算摘要。它与使用[eth_sign](https://eth.wiki/json-rpc/API#eth_sign) JSON-RPC方法签名的哈希相对应。

> NOTE
messageHash参数旨在是用keccak256对原始消息进行哈希运算的结果，尽管可以安全地使用任何bytes32值，因为最终的摘要将被重新哈希。
参见*ECDSA.recover*。

#### toEthSignedMessageHash(bytes message) → bytes32
internal#
返回EIP-191签名数据的keccak256摘要，版本为0x45（personal_sign消息）。

通过在任意消息前加上"\x19Ethereum Signed Message:\n" + len(message)并对结果进行哈希来计算摘要。它对应于使用[eth_sign](https://eth.wiki/json-rpc/API#eth_sign) JSON-RPC方法签名的哈希。

参见*ECDSA.recover*。

#### toDataWithIntendedValidatorHash(address validator, bytes data) → bytes32
internal#
返回EIP-191签名数据的keccak256摘要，版本为0x00（带有预期验证器的数据）。

通过在任意数据前加上"\x19\x00"和预期的验证器地址，然后对结果进行哈希，来计算摘要。

参见*ECDSA.recover*。

#### toTypedDataHash(bytes32 domainSeparator, bytes32 structHash) → bytes32 digest
internal#
返回EIP-712类型数据的keccak256摘要（EIP-191版本0x01）。

该摘要是通过在domainSeparator和structHash前加上\x19\x01并对结果进行哈希计算得到的。它对应于作为EIP-712一部分的[eth_signTypedData](https://eips.ethereum.org/EIPS/eip-712) JSON-RPC方法签名的哈希。

参见*ECDSA.recover*。

### [SignatureChecker](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/cryptography/SignatureChecker.sol)
```
import "@openzeppelin/contracts/utils/cryptography/SignatureChecker.sol";
```

签名验证助手，可以替代ECDSA.recover，无缝支持来自外部拥有账户（EOAs）的ECDSA签名以及来自像Argent和Safe Wallet（以前的Gnosis Safe）这样的智能合约钱包的ERC1271签名。

**FUNCTIONS**
isValidSignatureNow(signer, hash, signature)

isValidERC1271SignatureNow(signer, hash, signature)

#### isValidSignatureNow(address signer, bytes32 hash, bytes signature) → bool
internal#
检查签名是否对给定的签名者和数据哈希有效。如果签名者是一个智能合约，那么使用ERC1271对该签名进行验证，否则使用ECDSA.recover进行验证。

> NOTE
与ECDSA签名不同，合约签名是可以撤销的，因此这个函数的结果可能会随着时间的推移而改变。它可能在区块N返回true，而在区块N+1返回false（或者反过来）。

#### isValidERC1271SignatureNow(address signer, bytes32 hash, bytes signature) → bool
internal#
检查签名是否对给定的签名者和数据哈希有效。该签名通过ERC1271对签名者智能合约进行验证。

> NOTE
与ECDSA签名不同，合约签名是可撤销的，因此该函数的结果可能会随时间变化。它可能在区块N返回true，在区块N+1返回false（或相反）。

#### [MerkleProof](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/cryptography/MerkleProof.sol)
```
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";
```

这些函数处理Merkle树证明的验证。

树和证明可以使用我们的[JavaScript库](https://github.com/OpenZeppelin/merkle-tree)生成。你可以在readme中找到一个快速入门指南。

> WARNING
你应该避免使用长度为64字节的叶值进行哈希，或者使用除keccak256之外的哈希函数进行叶的哈希。这是因为在Merkle树中，一个排序对的内部节点的连接可能被重新解释为一个叶值。OpenZeppelin的JavaScript库生成的Merkle树默认是安全的，不会受到这种攻击。

**FUNCTIONS**
verify(proof, root, leaf)

verifyCalldata(proof, root, leaf)

processProof(proof, leaf)

processProofCalldata(proof, leaf)

multiProofVerify(proof, proofFlags, root, leaves)

multiProofVerifyCalldata(proof, proofFlags, root, leaves)

processMultiProof(proof, proofFlags, leaves)

processMultiProofCalldata(proof, proofFlags, leaves)

**ERRORS**
MerkleProofInvalidMultiproof()

#### verify(bytes32[] proof, bytes32 root, bytes32 leaf) → bool
internal#
如果可以证明一个叶子是由根定义的Merkle树的一部分，则返回真。为此，必须提供一个证明，包含从叶子到树根的分支上的兄弟哈希。假定每对叶子和每对预映像都是排序的。

#### verifyCalldata(bytes32[] proof, bytes32 root, bytes32 leaf) → bool
internal#
*验证*调用数据版本

#### processProof(bytes32[] proof, bytes32 leaf) → bytes32
internal#
返回通过使用证明从叶子向上遍历Merkle树获得的重建哈希。只有当重建的哈希与树的根匹配时，证明才有效。在处理证明时，假定叶子和预映像的对是排序的。

#### processProofCalldata(bytes32[] proof, bytes32 leaf) → bytes32
internal#
*处理证明*的调用数据版本

#### multiProofVerify(bytes32[] proof, bool[] proofFlags, bytes32 root, bytes32[] leaves) → bool
internal#
如果根据*processMultiProof*中描述的证明和证明标志，叶子可以同时被证明是由根定义的Merkle树的一部分，则返回true。

> CAUTION
并非所有的Merkle树都接受多重证明。详情请参阅*processMultiProof*。

#### multiProofVerifyCalldata(bytes32[] proof, bool[] proofFlags, bytes32 root, bytes32[] leaves) → bool
internal#
*验证多重证明*的调用数据版本。

> CAUTION
并非所有的Merkle树都支持多重证明。具体详情请参见*processMultiProof*。

#### processMultiProof(bytes32[] proof, bool[] proofFlags, bytes32[] leaves) → bytes32 merkleRoot
internal#
返回从证明中的叶子和兄弟节点重构的树的根。重构通过逐步重构所有内部节点来进行，通过将叶子/内部节点与另一个叶子/内部节点或证明兄弟节点结合，这取决于每个proofFlags项是真还是假。

> CAUTION
并非所有的默克尔树都承认多重证明。要使用多重证明，只需确保：1）树是完整的（但不一定是完美的），2）要证明的叶子是在树中的相反顺序（即，从最深层的右到左开始，然后继续在下一层）。

#### processMultiProofCalldata(bytes32[] proof, bool[] proofFlags, bytes32[] leaves) → bytes32 merkleRoot
internal#
调用数据版本的*processMultiProof*。

并非所有的Merkle树都支持多重证明。详情请参阅*processMultiProof*。

#### MerkleProofInvalidMultiproof()
error#
提供的多重证明是无效的。

### [EIP712](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/cryptography/EIP712.sol)
```
import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";
```

[EIP 712](https://eips.ethereum.org/EIPS/eip-712)是一种用于哈希和签名类型化结构化数据的标准。

EIP中指定的编码方案需要一个域分隔符和类型化结构化数据的哈希，其编码非常通用，因此在Solidity中无法实现，因此该合约并未实现编码本身。协议需要实现他们需要的类型特定编码，以便使用abi.encode和keccak256的组合产生他们的类型化数据的哈希。

该合约实现了EIP 712域分隔符(*_domainSeparatorV4*)，该分隔符用作编码方案的一部分，以及获取消息摘要的编码的最后步骤，然后通过ECDSA进行签名(*_hashTypedDataV4*)。

域分隔符的实现旨在尽可能高效，同时仍然正确地更新链id，以防止对链的可能分叉的重放攻击。

> NOTE
此合约实现了被称为"v4"的编码版本，该版本由[MetaMask中的JSON RPC方法eth_signTypedDataV4](https://docs.metamask.io/guide/signing-data.html)实现。

> NOTE
在这个可升级版本的合约中，缓存的值将对应于实施合约的地址和域分隔符。这将导致*_domainSeparatorV4*函数始终从不可变的值中重建分隔符，这比访问冷存储中的缓存版本更便宜。

**FUNCTIONS**
constructor(name, version)

_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
IERC5267
EIP712DomainChanged()

#### constructor(string name, string version)
internal#
初始化域分隔符和参数缓存。

名称和版本的含义在[EIP 712](https://eips.ethereum.org/EIPS/eip-712#definition-of-domainseparator)中有所规定：

* 名称：签名域的用户可读名称，即DApp或协议的名称。

* 版本：签名域的当前主要版本。

> NOTE
这些参数除非通过*智能合约升级*，否则无法更改。

#### _domainSeparatorV4() → bytes32
internal#
返回当前链的域分隔符。

#### _hashTypedDataV4(bytes32 structHash) → bytes32
internal#
这个函数返回一个已经[哈希过的结构](https://eips.ethereum.org/EIPS/eip-712#definition-of-hashstruct)完全编码的EIP712消息的哈希。

这个哈希可以和*ECDSA.recover*一起使用，以获取消息的签名者。例如:
```
bytes32 digest = _hashTypedDataV4(keccak256(abi.encode(
    keccak256("Mail(address to,string contents)"),
    mailTo,
    keccak256(bytes(mailContents))
)));
address signer = ECDSA.recover(digest, signature);
```

#### eip712Domain() → bytes1 fields, string name, string version, uint256 chainId, address verifyingContract, bytes32 salt, uint256[] extensions
public#
查阅 {IERC-5267}.

#### _EIP712Name() → string
internal#
EIP712域的名称参数。

> NOTE
默认情况下，此函数读取的是_name，这是一个不可变的值。只有在必要的时候（例如，值太大，无法放入ShortString中）才会从存储中读取。

#### _EIP712Version() → string
internal#
EIP712域的版本参数。

> NOTE
默认情况下，此函数读取的是不可变值_version。只有在必要时（例如，值太大，无法放入ShortString中）才从存储中读取。

## Security

### [ReentrancyGuard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/ReentrancyGuard.sol)
```
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";
```

该合约模块有助于防止对函数的重入调用。

继承ReentrancyGuard将使*nonReentrant*修饰符可用，可以将其应用于函数，以确保它们没有嵌套（重入）调用。

请注意，由于只有一个nonReentrant保护，因此标记为nonReentrant的函数可能无法相互调用。可以通过将这些函数设为私有，然后为它们添加外部nonReentrant入口点来解决这个问题。

> TIP
如果你想了解更多关于重入性以及防止重入的替代方法，可以查看我们的博客文章[《伊斯坦布尔后的重入性》](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

**MODIFIERS**
nonReentrant()

**FUNCTIONS**
constructor()

_reentrancyGuardEntered()

**ERRORS**
ReentrancyGuardReentrantCall()

#### nonReentrant()
modifier#
防止合约直接或间接地调用自身。不支持从另一个非重入函数调用非重入函数。可以通过将非重入函数设为外部函数，并让它调用一个执行实际工作的私有函数来防止这种情况发生。

#### constructor()
internal#

#### _reentrancyGuardEntered() → bool
internal#
如果重入保护当前设置为"entered"，表示调用堆栈中存在一个nonReentrant函数，则返回true。

#### ReentrancyGuardReentrantCall()
error#
未经授权的重入调用。

### [Pausable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/Pausable.sol)
```
import "@openzeppelin/contracts/utils/Pausable.sol";
```

这个合约模块允许子合约实现一个紧急停止机制，该机制可以由授权账户触发。

这个模块是通过继承来使用的。它将提供whenNotPaused和whenPaused修饰符，这些修饰符可以应用到你的合约函数上。请注意，仅仅包含这个模块并不能使它们可暂停，只有当修饰符被放置后才能暂停。

**MODIFIERS**
whenNotPaused()

whenPaused()

**FUNCTIONS**
constructor()

paused()

_requireNotPaused()

_requirePaused()

_pause()

_unpause()

**EVENTS**
Paused(account)

Unpaused(account)

**ERRORS**
EnforcedPause()

ExpectedPause()

#### whenNotPaused()
modifier#
修饰符用于使函数仅在合约未暂停时可调用。

要求：
* 合约必须未处于暂停状态。

#### whenPaused()
modifier#
修饰符用于使函数仅在合约未暂停时可调用。

要求：
* 合约必须未处于暂停状态。

#### constructor()
internal#
将合约初始化为未暂停状态。

#### paused() → bool
public#
如果合约被暂停，则返回真，否则返回假。

#### _requireNotPaused()
internal#
如果合约被暂停，则抛出异常。

#### _requirePaused()
internal#
如果合约没有暂停，就会抛出异常。

#### _pause()
internal#
触发停止状态。

要求：
* 合约不能被暂停。

#### _unpause()
返回到正常状态。

要求：
* 合约必须被暂停。

#### Paused(address account)
event#
当账户触发暂停时发出。

#### Unpaused(address account)
event#
当账户解除暂停时发出。

#### EnforcedPause()
error#
操作失败，因为合约已暂停。

#### ExpectedPause()
error#
操作失败，因为合约并未暂停。

## Introspection
这套接口和合约处理合约的类型自省，也就是检查可以在合约上调用哪些函数。这通常被称为合约的接口。

以太坊合约没有接口的原生概念，所以应用程序通常只能信任它们没有进行错误的调用。对于可信的设置，这不是问题，但是经常需要与未知的和不可信的第三方地址进行交互。甚至可能没有对它们的直接调用！（例如，ERC20代币可能被发送到一个没有转移它们的方式的合约，永远锁定它们）。在这些情况下，一个声明其接口的合约可以非常有效地防止错误。

### [IERC165](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/introspection/IERC165.sol)
```
import "@openzeppelin/contracts/utils/introspection/IERC165.sol";
```

ERC165标准的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-165)中定义的那样。

实施者可以声明支持合约接口，然后可以由其他人（*ERC165Checker*）查询。

有关实施，请参阅*ERC165*。

**FUNCTIONS**
supportsInterface(interfaceId)

#### supportsInterface(bytes4 interfaceId) → bool
external#
如果此合约实现了由interfaceId定义的接口，则返回true。请参阅相应的[EIP部分](https://eips.ethereum.org/EIPS/eip-165#how-interfaces-are-identified)，了解更多关于如何创建这些id的信息。

此函数调用必须使用少于30,000个gas。

### [ERC165](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/introspection/ERC165.sol)
```
import "@openzeppelin/contracts/utils/introspection/ERC165.sol";
```

想要实现*ERC165*接口的合约应该继承自这个合约，并重写*supportsInterface*以检查将被支持的额外接口id。例如:
```
function supportsInterface(bytes4 interfaceId) public view virtual override returns (bool) {
    return interfaceId == type(MyInterface).interfaceId || super.supportsInterface(interfaceId);
}
```

**FUNCTIONS**
supportsInterface(interfaceId)

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查阅 *IERC165.supportsInterface*.

### [ERC165Checker](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/introspection/ERC165Checker.sol)
```
import "@openzeppelin/contracts/utils/introspection/ERC165Checker.sol";
```

用于查询通过*IERC165*声明的接口支持的库。

请注意，这些函数返回查询的实际结果：如果不支持接口，它们不会恢复。这取决于调用者在这些情况下该做什么。

**FUNCTIONS**
supportsERC165(account)

supportsInterface(account, interfaceId)

getSupportedInterfaces(account, interfaceIds)

supportsAllInterfaces(account, interfaceIds)

supportsERC165InterfaceUnchecked(account, interfaceId)

#### supportsERC165(address account) → bool
internal#
如果账户支持*IERC165*接口，则返回真。

#### supportsInterface(address account, bytes4 interfaceId) → bool
internal#
如果账户支持由interfaceId定义的接口，则返回true。*IERC165*本身的支持会自动查询。

参见*IERC165.supportsInterface*。

#### getSupportedInterfaces(address account, bytes4[] interfaceIds) → bool[]
internal#
返回一个布尔数组，其中每个值对应于传入的接口以及是否支持它们。这允许你批量检查合约的接口，你的期望是可能有些接口不被支持。

参见*IERC165.supportsInterface*。

#### supportsAllInterfaces(address account, bytes4[] interfaceIds) → bool
internal#
如果账户支持interfaceIds中定义的所有接口，则返回true。*IERC165*本身的支持会自动查询。

批量查询可以通过跳过对*IERC165*支持的重复检查来节省gas。

参见*IERC165.supportsInterface*。

#### supportsERC165InterfaceUnchecked(address account, bytes4 interfaceId) → bool
internal#
假设账户包含支持ERC165的合约，否则此方法的行为是未定义的。可以通过*supportsERC165*来检查这个前提条件。

一些预编译的合约会错误地表示支持给定的接口，因此在使用此函数时应谨慎。

接口识别在ERC-165中有规定。

## Data Structures

### [BitMaps](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/structs/BitMaps.sol)
```
import "@openzeppelin/contracts/utils/structs/BitMaps.sol";
```

这是一个用于以紧凑和高效的方式管理uint256到bool映射的库，前提是键是顺序的。这在很大程度上受到Uniswap的merkle-distributor的启发。

BitMaps将256个布尔值打包到单个256位的uint256类型的插槽的每一位中。因此，对应于256个顺序索引的布尔值只会消耗一个插槽，而不像常规的bool，它会为单个值消耗整个插槽。

这以两种方式节省了gas：

每256次只有一次将零值设置为非零

每256个顺序索引都访问同一个热插槽

**FUNCTIONS**
get(bitmap, index)

setTo(bitmap, index, value)

set(bitmap, index)

unset(bitmap, index)

#### get(struct BitMaps.BitMap bitmap, uint256 index) → bool
internal#
返回索引处的位是否设置。

#### setTo(struct BitMaps.BitMap bitmap, uint256 index, bool value)
internal#
将索引处的位设置为布尔值。

#### set(struct BitMaps.BitMap bitmap, uint256 index)
internal#
设置索引处的位。

#### unset(struct BitMaps.BitMap bitmap, uint256 index)
internal#
取消索引处的位。

### [EnumerableMap](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/structs/EnumerableMap.sol)
```
import "@openzeppelin/contracts/utils/structs/EnumerableMap.sol";
```

用于管理Solidity的[映射类型](https://solidity.readthedocs.io/en/latest/types.html#mapping-types)的可枚举变体的库。

映射具有以下属性：

*  在常数时间（O(1)）内添加、删除和检查条目的存在。

*  在O(n)中枚举条目。不对排序做任何保证。

```
contract Example {
    // Add the library methods
    using EnumerableMap for EnumerableMap.UintToAddressMap;

    // Declare a set state variable
    EnumerableMap.UintToAddressMap private myMap;
}
```
支持以下映射类型：

* uint256 → address (UintToAddressMap) 自v3.0.0起

* address → uint256 (AddressToUintMap) 自v4.6.0起

* bytes32 → bytes32 (Bytes32ToBytes32Map) 自v4.6.0起

* uint256 → uint256 (UintToUintMap) 自v4.7.0起

* bytes32 → uint256 (Bytes32ToUintMap) 自v4.7.0起

> WARNING
尝试从存储中删除这样的结构可能会导致数据损坏，使结构无法使用。更多信息请参见[ethereum/solidity#11843](https://github.com/ethereum/solidity/pull/11843)。
为了清理一个EnumerableMap，你可以逐一删除所有元素，或者使用EnumerableMap的数组创建一个新的实例。

**FUNCTIONS**
set(map, key, value)

remove(map, key)

contains(map, key)

length(map)

at(map, index)

tryGet(map, key)

get(map, key)

keys(map)

set(map, key, value)

remove(map, key)

contains(map, key)

length(map)

at(map, index)

tryGet(map, key)

get(map, key)

keys(map)

set(map, key, value)

remove(map, key)

contains(map, key)

length(map)

at(map, index)

tryGet(map, key)

get(map, key)

keys(map)

set(map, key, value)

remove(map, key)

contains(map, key)

length(map)

at(map, index)

tryGet(map, key)

get(map, key)

keys(map)

set(map, key, value)

remove(map, key)

contains(map, key)

length(map)

at(map, index)

tryGet(map, key)

get(map, key)

keys(map)

**ERRORS**
EnumerableMapNonexistentKey(key)

#### set(struct EnumerableMap.Bytes32ToBytes32Map map, bytes32 key, bytes32 value) → bool
internal#
将键值对添加到映射中，或更新现有键的值。O(1)。

如果键被添加到映射中，即它之前不存在，则返回true。

#### remove(struct EnumerableMap.Bytes32ToBytes32Map map, bytes32 key) → bool
internal#
从映射中删除一个键值对。时间复杂度为O(1)。

如果键已从映射中删除，即如果它存在，则返回true。

#### contains(struct EnumerableMap.Bytes32ToBytes32Map map, bytes32 key) → bool
internal#
如果键在映射中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableMap.Bytes32ToBytes32Map map) → uint256
internal#
返回映射中键值对的数量。时间复杂度为O(1)。

#### at(struct EnumerableMap.Bytes32ToBytes32Map map, uint256 index) → bytes32, bytes32
internal#
返回在映射中位置索引处存储的键值对。O(1)。

请注意，数组内条目的排序没有任何保证，并且在添加或删除更多条目时可能会改变。

要求：
* 索引必须严格小于*length*。

#### tryGet(struct EnumerableMap.Bytes32ToBytes32Map map, bytes32 key) → bool, bytes32
internal#
尝试返回与键关联的值。时间复杂度为O(1)。如果键不在映射中，不会恢复。

#### get(struct EnumerableMap.Bytes32ToBytes32Map map, bytes32 key) → bytes32
internal#
返回与键关联的值。O(1)。

要求：
* 键必须在映射中。

#### keys(struct EnumerableMap.Bytes32ToBytes32Map map) → bytes32[]
internal#
返回一个包含所有键的数组

> WARNING
这个操作会将整个存储复制到内存中，这可能相当昂贵。这主要是为了被视图访问器使用，这些访问器在没有任何gas费用的情况下被查询。开发者应该记住，这个函数的成本是无限的，如果映射增长到一个点，复制到内存消耗太多的gas以至于无法适应一个块，那么使用它作为一个状态改变的函数可能会使函数无法调用。

#### set(struct EnumerableMap.UintToUintMap map, uint256 key, uint256 value) → bool
internal#
将键值对添加到映射中，或更新现有键的值。 O(1)。

如果键被添加到映射中，即如果它之前不存在，则返回true。

#### remove(struct EnumerableMap.UintToUintMap map, uint256 key) → bool
internal#
从映射中移除一个值。时间复杂度为O(1)。

如果键已从映射中移除，即它原本存在，那么返回真。

#### contains(struct EnumerableMap.UintToUintMap map, uint256 key) → bool
internal#
如果键在映射中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableMap.UintToUintMap map) → uint256
internal#
返回映射中元素的数量。时间复杂度为O(1)。

#### at(struct EnumerableMap.UintToUintMap map, uint256 index) → uint256, uint256
internal#
返回存储在映射中位置索引的元素。 O(1)。请注意，数组内部值的排序没有任何保证，并且在添加或删除更多值时可能会改变。

要求：
* 索引必须严格小于 *length*。

#### tryGet(struct EnumerableMap.UintToUintMap map, uint256 key) → bool, uint256
internal#
尝试返回与键关联的值。时间复杂度为O(1)。如果键不在映射中，不会恢复。

#### get(struct EnumerableMap.UintToUintMap map, uint256 key) → uint256
internal#
返回与键关联的值。O(1)。

要求：
* 键必须在映射中。

#### keys(struct EnumerableMap.UintToUintMap map) → uint256[]
internal#
返回一个包含所有键的数组

> WARNING
这个操作会将整个存储复制到内存中，这可能相当昂贵。这主要是为了被查询时不需要任何gas费用的视图访问器设计的。开发者应该记住，这个函数的成本是无限的，如果映射增长到一个点，复制到内存消耗太多的gas以至于不能适应一个块，那么作为状态改变函数的一部分使用它可能会使函数无法调用。

#### set(struct EnumerableMap.UintToAddressMap map, uint256 key, address value) → bool
internal#
将键值对添加到映射中，或更新现有键的值。O(1)。

如果键被添加到映射中，即它之前不存在，则返回true。

#### remove(struct EnumerableMap.UintToAddressMap map, uint256 key) → bool
internal#
从映射中移除一个值。时间复杂度为O(1)。

如果键从映射中被移除，即它原本存在，那么返回真。

#### contains(struct EnumerableMap.UintToAddressMap map, uint256 key) → bool
internal#
如果键在映射中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableMap.UintToAddressMap map) → uint256
internal#
返回映射中元素的数量。时间复杂度为O(1)。

#### at(struct EnumerableMap.UintToAddressMap map, uint256 index) → uint256, address
internal#
返回存储在映射中位置索引的元素。O(1)。请注意，数组内值的排序没有任何保证，并且在添加或删除更多值时可能会改变。

要求：
索引必须严格小于*length*。

#### tryGet(struct EnumerableMap.UintToAddressMap map, uint256 key) → bool, address
internal#
尝试返回与键关联的值。时间复杂度为O(1)。如果键不在映射中，不会还原。

#### get(struct EnumerableMap.UintToAddressMap map, uint256 key) → address
internal#
返回与键关联的值。O(1)。

要求：
* 键必须在映射中。

#### keys(struct EnumerableMap.UintToAddressMap map) → uint256[]
internal#
返回一个包含所有键的数组

> WARNING
这个操作将把整个存储复制到内存中，这可能相当昂贵。这主要是为了被无需任何gas费用的视图访问器查询而设计的。开发者应该记住，这个函数的成本是无限的，如果映射增长到一个点，复制到内存消耗的gas太多以至于无法适应一个块，那么使用它作为一个状态改变函数可能会使函数无法调用。

#### set(struct EnumerableMap.AddressToUintMap map, address key, uint256 value) → bool
internal#
向映射中添加一个键值对，或更新现有键的值。时间复杂度为O(1)。

如果键被添加到映射中，即它之前并未存在，则返回真。

#### remove(struct EnumerableMap.AddressToUintMap map, address key) → bool
internal#
从映射中移除一个值。时间复杂度为O(1)。

如果键已从映射中移除，即如果它存在，则返回true。

#### contains(struct EnumerableMap.AddressToUintMap map, address key) → bool
internal#
如果键在映射中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableMap.AddressToUintMap map) → uint256
internal#
返回映射中元素的数量。时间复杂度为O(1)。

#### at(struct EnumerableMap.AddressToUintMap map, uint256 index) → address, uint256
internal#
返回存储在映射中位置索引的元素。 O(1)。请注意，数组内部值的排序没有任何保证，并且在添加或删除更多值时可能会改变。

要求：
* 索引必须严格小于*length*。

#### tryGet(struct EnumerableMap.UintToAddressMap map, uint256 key) → bool, address
internal#
尝试返回与键关联的值。时间复杂度为O(1)。如果键不在映射中，不会还原。

#### get(struct EnumerableMap.UintToAddressMap map, uint256 key) → address
internal#
返回与键关联的值。O(1)。

要求：
* 键必须在映射中。

#### keys(struct EnumerableMap.UintToAddressMap map) → uint256[]
internal#
返回一个包含所有键的数组

> WARNING
这个操作会将整个存储复制到内存中，这可能相当昂贵。这主要是为了被无需任何gas费用的视图访问器查询而设计的。开发者应该记住，这个函数的成本是无限的，如果映射增长到一个点，复制到内存消耗的gas太多，无法适应一个块，那么作为状态改变函数的一部分使用它可能会使函数无法调用。

#### set(struct EnumerableMap.AddressToUintMap map, address key, uint256 value) → bool
internal#
向映射中添加一个键值对，或更新现有键的值。 O(1)。

如果键被添加到映射中，即它之前不存在，则返回true。

#### remove(struct EnumerableMap.AddressToUintMap map, address key) → bool
internal#
从映射中移除一个值。时间复杂度为O(1)。

如果键已从映射中移除，即它原本存在，那么返回真。

#### contains(struct EnumerableMap.AddressToUintMap map, address key) → bool
internal#
如果键在映射中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableMap.AddressToUintMap map) → uint256
internal#
返回映射中元素的数量。时间复杂度为O(1)。

#### at(struct EnumerableMap.AddressToUintMap map, uint256 index) → address, uint256
internal#
返回存储在映射中位置索引的元素。 O(1)。请注意，数组内部值的排序没有保证，并且在添加或删除更多值时可能会改变。

要求：
* 索引必须严格小于*长度*。

#### tryGet(struct EnumerableMap.AddressToUintMap map, address key) → bool, uint256
internal#
尝试返回与键关联的值。时间复杂度为O(1)。如果键不在映射中，不会恢复。

#### get(struct EnumerableMap.AddressToUintMap map, address key) → uint256
internal#
返回与键关联的值。O(1)。

要求：
* 键必须在映射中。

#### keys(struct EnumerableMap.AddressToUintMap map) → address[]
internal#
返回一个包含所有键的数组

> WARNING
这个操作会将整个存储复制到内存中，这可能相当昂贵。这主要是为了在没有任何gas费用的情况下查询视图访问器而设计的。开发者应该记住，这个函数的成本是无限的，如果映射增长到复制到内存消耗太多gas以至于无法适应一个块，那么使用它作为一个改变状态的函数可能会使函数无法调用。

#### set(struct EnumerableMap.Bytes32ToUintMap map, bytes32 key, uint256 value) → bool
internal#
向映射添加一个键值对，或更新现有键的值。O(1)。

如果键被添加到映射中，即如果它之前不存在，则返回true。

#### remove(struct EnumerableMap.Bytes32ToUintMap map, bytes32 key) → bool
internal#
从映射中删除一个值。时间复杂度为O(1)。

如果键已从映射中删除，即如果它存在，则返回true。

#### contains(struct EnumerableMap.Bytes32ToUintMap map, bytes32 key) → bool
internal#
如果键在映射中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableMap.Bytes32ToUintMap map) → uint256
internal#
返回映射中的元素数量。时间复杂度为O(1)。

#### at(struct EnumerableMap.Bytes32ToUintMap map, uint256 index) → bytes32, uint256
internal#
返回存储在映射中位置索引的元素。O(1)。请注意，数组内值的排序没有任何保证，并且在添加或删除更多值时可能会改变。

要求：
* 索引必须严格小于长度。

#### tryGet(struct EnumerableMap.Bytes32ToUintMap map, bytes32 key) → bool, uint256
internal#
尝试返回与键关联的值。时间复杂度为O(1)。如果键不在映射中，不会恢复。

#### get(struct EnumerableMap.Bytes32ToUintMap map, bytes32 key) → uint256
internal#
返回与键关联的值。O(1)。

要求：
* 键必须在映射中。

#### keys(struct EnumerableMap.Bytes32ToUintMap map) → bytes32[]
internal#
返回一个包含所有键的数组

> WARNING
此操作将把整个存储复制到内存中，这可能相当昂贵。这主要是为了被无需任何gas费用的视图访问器查询而设计的。开发人员应该记住，这个函数的成本是无限的，如果映射增长到一个点，复制到内存消耗太多的gas以至于无法适应一个块，那么作为状态改变函数的一部分使用它可能会使函数无法调用。

#### EnumerableMapNonexistentKey(bytes32 key)
error#
查询一个不存在的映射键。

### [EnumerableSet](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/structs/EnumerableSet.sol)
```
import "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";
```

用于管理基本类型[集合](https://en.wikipedia.org/wiki/Set_(abstract_data_type))的库。

集合具有以下属性：
* 元素的添加、删除和存在性检查都是常数时间 (O(1))。

* 元素的枚举是O(n)。不对排序做任何保证。

```
contract Example {
    // Add the library methods
    using EnumerableSet for EnumerableSet.AddressSet;

    // Declare a set state variable
    EnumerableSet.AddressSet private mySet;
}
```

从v3.3.0开始，支持bytes32（Bytes32Set）、address（AddressSet）和uint256（UintSet）类型的集合。

> WARNING
尝试从存储中删除这样的结构可能会导致数据损坏，使结构无法使用。更多信息请参见[ethereum/solidity#11843](https://github.com/ethereum/solidity/pull/11843)。
为了清理一个EnumerableSet，你可以逐一删除所有元素，或者使用EnumerableSet数组创建一个新的实例。

**FUNCTIONS**
add(set, value)

remove(set, value)

contains(set, value)

length(set)

at(set, index)

values(set)

add(set, value)

remove(set, value)

contains(set, value)

length(set)

at(set, index)

values(set)

add(set, value)

remove(set, value)

contains(set, value)

length(set)

at(set, index)

values(set)

#### add(struct EnumerableSet.Bytes32Set set, bytes32 value) → bool
internal#
向集合添加一个值。时间复杂度为O(1)。

如果该值被添加到集合中，即如果它之前并未存在，则返回真。

#### remove(struct EnumerableSet.Bytes32Set set, bytes32 value) → bool
internal#
从集合中移除一个值。时间复杂度为O(1)。

如果该值已从集合中移除，即如果该值存在，则返回true。

#### contains(struct EnumerableSet.Bytes32Set set, bytes32 value) → bool
internal#
如果值在集合中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableSet.Bytes32Set set) → uint256
internal#
返回集合中的值的数量。时间复杂度为O(1)。

#### at(struct EnumerableSet.Bytes32Set set, uint256 index) → bytes32
internal#
返回集合中位置索引处存储的值。O(1)。

请注意，数组内部值的排序没有任何保证，并且在添加或删除更多值时可能会改变。

要求：

索引必须严格小于 *length*。

#### values(struct EnumerableSet.Bytes32Set set) → bytes32[]
internal#
将整个集合返回为数组

> WARNING
此操作将把整个存储复制到内存中，这可能相当昂贵。这主要是为了被无需任何gas费用的视图访问器查询而设计的。开发者应该记住，这个函数的成本是无限的，如果集合增长到复制到内存消耗太多gas以至于不能适应一个区块，那么作为状态改变函数的一部分使用它可能会使函数无法调用。

#### add(struct EnumerableSet.AddressSet set, address value) → bool
internal#
向集合添加一个值。时间复杂度为O(1)。

如果该值被添加到集合中，即它之前并未存在，则返回真。

#### remove(struct EnumerableSet.AddressSet set, address value) → bool
internal#
从集合中移除一个值。时间复杂度为O(1)。

如果该值从集合中被移除，即它原本就在集合中，则返回真。

#### contains(struct EnumerableSet.AddressSet set, address value) → bool
internal#
如果值在集合中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableSet.AddressSet set) → uint256
internal#
返回集合中的值数量。时间复杂度为O(1)。

#### at(struct EnumerableSet.AddressSet set, uint256 index) → address
internal#
返回集合中索引位置存储的值。O(1)。

请注意，数组内值的排序没有任何保证，并且在添加或删除更多值时可能会改变。

要求：
* 索引必须严格小于长度。

#### values(struct EnumerableSet.AddressSet set) → address[]
internal#
将整个集合返回为数组

> WARNING
此操作将把整个存储复制到内存中，这可能相当昂贵。这主要是为了在没有任何gas费用的情况下查询的视图访问器而设计的。开发者应该记住，这个函数的成本是无限的，如果集合增长到一个点，复制到内存消耗的gas太多以至于无法适应一个区块，那么作为状态改变函数的一部分使用它可能会使函数无法调用。

#### add(struct EnumerableSet.UintSet set, uint256 value) → bool
internal#
向集合添加一个值。时间复杂度为O(1)。

如果该值被添加到集合中，即它之前并未存在于集合中，则返回真。

#### remove(struct EnumerableSet.UintSet set, uint256 value) → bool
internal#
从集合中移除一个值。时间复杂度为O(1)。

如果该值被从集合中移除，即它原本就存在于集合中，则返回真。

#### contains(struct EnumerableSet.UintSet set, uint256 value) → bool
internal#
如果值在集合中，则返回真。时间复杂度为O(1)。

#### length(struct EnumerableSet.UintSet set) → uint256
internal#
返回集合中的值的数量。时间复杂度为O(1)。

#### at(struct EnumerableSet.UintSet set, uint256 index) → uint256
internal#
返回集合中索引位置存储的值。O(1)。

请注意，数组内值的排序没有任何保证，当添加或删除更多值时，它可能会改变。

要求：
* 索引必须严格小于*length*。

#### values(struct EnumerableSet.UintSet set) → uint256[]
internal#
将整个集合返回为数组

> WARNING
此操作将把整个存储复制到内存中，这可能相当昂贵。这主要是为了被无需任何gas费的视图访问器查询而设计的。开发者应该记住，这个函数的成本是无限的，如果集合增长到一个点，复制到内存消耗的gas太多以至于无法适应一个区块，那么作为状态改变函数的一部分使用它可能会使函数无法调用。

### [DoubleEndedQueue](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/structs/DoubleEndedQueue.sol)
```
import "@openzeppelin/contracts/utils/structs/DoubleEndedQueue.sol";
```

这是一个项目序列，具有在序列的两端（称为前端和后端）有效地推送和弹出项目（即插入和移除）的能力。除了其他访问模式，它可以用来实现有效的LIFO和FIFO队列。存储使用已优化，所有操作都是O(1)常数时间。这包括[clear](https://docs.openzeppelin.com/contracts/5.x/api/utils#DoubleEndedQueue-clear-struct-DoubleEndedQueue-Bytes32Deque-)，前提是现有队列内容保留在存储中。

这个结构被称为Bytes32Deque。其他类型可以被转换为bytes32。这种数据结构只能在存储中使用，不能在内存中使用。
```
DoubleEndedQueue.Bytes32Deque queue;
```

**FUNCTIONS**
pushBack(deque, value)

popBack(deque)

pushFront(deque, value)

popFront(deque)

front(deque)

back(deque)

at(deque, index)

clear(deque)

length(deque)

empty(deque)

**ERRORS**
QueueEmpty()

QueueFull()

QueueOutOfBounds()

#### pushBack(struct DoubleEndedQueue.Bytes32Deque deque, bytes32 value)
internal#
在队列的末尾插入一个项目。

如果队列已满，则返回*QueueFull*。

#### popBack(struct DoubleEndedQueue.Bytes32Deque deque) → bytes32 value
internal#
删除队列末尾的项目并返回它。

如果队列为空，则以*QueueEmpty*为原因进行回滚。

#### pushFront(struct DoubleEndedQueue.Bytes32Deque deque, bytes32 value)
internal#
在队列的开始插入一个项目。

如果队列已满，则返回*QueueFull*。

#### popFront(struct DoubleEndedQueue.Bytes32Deque deque) → bytes32 value
internal#
从队列的开始处移除项目并返回它。

如果队列为空，则以QueueEmpty为错误进行回退。

#### front(struct DoubleEndedQueue.Bytes32Deque deque) → bytes32 value
internal#
返回队列开始的项目。

如果队列为空，则返回QueueEmpty。

#### back(struct DoubleEndedQueue.Bytes32Deque deque) → bytes32 value
internal#
返回队列末尾的项目。

如果队列为空，则返回QueueEmpty。

#### at(struct DoubleEndedQueue.Bytes32Deque deque, uint256 index) → bytes32 value
internal#
返回队列中由索引给出位置的项目，第一个项目在0，最后一个项目在length(deque) - 1。

如果索引超出范围，则返回QueueOutOfBounds。

#### clear(struct DoubleEndedQueue.Bytes32Deque deque)
internal#
将队列重置为空。

> NOTE
当前的项目会被留在存储中。这不会影响队列的功能，但会错过潜在的gas退款。

#### length(struct DoubleEndedQueue.Bytes32Deque deque) → uint256
internal#
返回队列中的项目数量。

#### empty(struct DoubleEndedQueue.Bytes32Deque deque) → bool
internal#
如果队列为空，则返回真。

#### QueueEmpty()
error#
由于队列为空，无法完成某项操作（例如，前端操作）。

#### QueueFull()
error#
由于队列已满，无法完成推送操作。

#### QueueOutOfBounds()
error#
由于索引超出范围，无法完成某项操作（例如*at*）。

### [Checkpoints](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/structs/Checkpoints.sol)
```
import "@openzeppelin/contracts/utils/structs/Checkpoints.sol";
```

这个库定义了Trace*结构体，用于在不同时间点检查值的变化，并在以后通过区块号查找过去的值。请参阅*Votes*作为一个例子。

要创建一个历史检查点，你需要在你的合约中定义一个Checkpoints.Trace*类型的变量，并使用*push*函数为当前交易块存储一个新的检查点。

**FUNCTIONS**
push(self, key, value)

lowerLookup(self, key)

upperLookup(self, key)

upperLookupRecent(self, key)

latest(self)

latestCheckpoint(self)

length(self)

at(self, pos)

push(self, key, value)

lowerLookup(self, key)

upperLookup(self, key)

upperLookupRecent(self, key)

latest(self)

latestCheckpoint(self)

length(self)

at(self, pos)

push(self, key, value)

lowerLookup(self, key)

upperLookup(self, key)

upperLookupRecent(self, key)

latest(self)

latestCheckpoint(self)

length(self)

at(self, pos)

**ERRORS**
CheckpointUnorderedInsertion()

#### push(struct Checkpoints.Trace224 self, uint32 key, uint224 value) → uint224, uint224
internal#
将一个(key, value)对推入Trace224，以便将其存储为检查点。

返回前一个值和新值。

> IMPORTANT
永远不要接受键作为用户输入，因为任意类型(uint32).max键集将禁用库。

#### lowerLookup(struct Checkpoints.Trace224 self, uint32 key) → uint224
internal#
返回第一个（最旧的）检查点中大于或等于搜索键的值，如果没有，则返回零。

#### upperLookup(struct Checkpoints.Trace224 self, uint32 key) → uint224
internal#
返回关键字小于或等于搜索关键字的最近（最后）检查点的值，如果没有则返回零。

#### upperLookupRecent(struct Checkpoints.Trace224 self, uint32 key) → uint224
internal#
返回键值小于或等于搜索键的最近（最新）检查点中的值，如果没有则返回零。

> NOTE
这是*upperLookup*的一个变体，它被优化用于查找"最近"的检查点（具有高键值的检查点）。

#### latest(struct Checkpoints.Trace224 self) → uint224
internal#
返回最近检查点的值，如果没有检查点，则返回零。

#### latestCheckpoint(struct Checkpoints.Trace224 self) → bool exists, uint32 _key, uint224 _value
internal#
返回结构中是否有检查点（即它不为空），如果有，则返回最近检查点中的键和值。

#### length(struct Checkpoints.Trace224 self) → uint256
internal#
返回检查点的数量。

#### at(struct Checkpoints.Trace224 self, uint32 pos) → struct Checkpoints.Checkpoint224
internal#
返回给定位置的检查点。

#### push(struct Checkpoints.Trace208 self, uint48 key, uint208 value) → uint208, uint208
internal#
将一个(key, value)对推入Trace208，使其作为检查点存储。

返回前一个值和新的值。

> IMPORTANT
永远不要接受键作为用户输入，因为一个任意类型(uint48).max键集将禁用库。

#### lowerLookup(struct Checkpoints.Trace208 self, uint48 key) → uint208
internal#
返回第一个（最旧的）检查点中大于或等于搜索键的值，如果没有则返回零。

#### upperLookup(struct Checkpoints.Trace208 self, uint48 key) → uint208
internal#
返回关键字低于或等于搜索关键字的最近（最后）检查点的值，如果没有，则返回零。

#### upperLookupRecent(struct Checkpoints.Trace208 self, uint48 key) → uint208
internal#
返回键值小于或等于搜索键的最近（最后）检查点中的值，如果没有则返回零。

> NOTE
这是*upperLookup*的一个变体，它经过优化以找到"最近"的检查点（具有高键的检查点）。

#### latest(struct Checkpoints.Trace208 self) → uint208
internal#
返回最近检查点的值，如果没有检查点，则返回零。

#### latestCheckpoint(struct Checkpoints.Trace208 self) → bool exists, uint48 _key, uint208 _value
internal#
返回结构中是否有检查点（即它不为空），如果有，则返回最近检查点中的键和值。

#### length(struct Checkpoints.Trace208 self) → uint256
internal#
返回检查点的数量。

#### at(struct Checkpoints.Trace208 self, uint32 pos) → struct Checkpoints.Checkpoint208
internal#
返回给定位置的检查点。

#### push(struct Checkpoints.Trace160 self, uint96 key, uint160 value) → uint160, uint160
internal#
将一个(key, value)对推入Trace160，以便将其存储为检查点。

返回前一个值和新的值。

> IMPORTANT
永远不要接受键作为用户输入，因为设置一个任意类型(uint96).max键将禁用库。

#### lowerLookup(struct Checkpoints.Trace160 self, uint96 key) → uint160
internal#
返回第一个（最旧的）检查点中键大于或等于搜索键的值，如果没有，则返回零。

#### upperLookup(struct Checkpoints.Trace160 self, uint96 key) → uint160
internal#
返回关键字低于或等于搜索关键字的最后（最近）检查点的值，如果没有则返回零。

#### upperLookupRecent(struct Checkpoints.Trace160 self, uint96 key) → uint160
internal#
返回关键字低于或等于搜索关键字的最近（最新）检查点的值，如果没有则返回零。

> NOTE
这是*upperLookup*的一个变体，优化了查找“最近”检查点（具有高关键字的检查点）。

#### latest(struct Checkpoints.Trace160 self) → uint160
internal#
返回最近检查点的值，如果没有检查点，则返回零。

#### latestCheckpoint(struct Checkpoints.Trace160 self) → bool exists, uint96 _key, uint160 _value
internal#
返回结构中是否有检查点（即它不为空），如果有，则返回最近检查点中的键和值。

#### length(struct Checkpoints.Trace160 self) → uint256
internal#
返回检查点的数量。

#### at(struct Checkpoints.Trace160 self, uint32 pos) → struct Checkpoints.Checkpoint160
internal#
返回给定位置的检查点。

#### CheckpointUnorderedInsertion()
error#
尝试在过去的检查点插入一个值。

## Libraries

### [Create2](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/Create2.sol)
```
import "@openzeppelin/contracts/utils/Create2.sol";
```

辅助使CREATE2 EVM操作码的使用更加简单和安全。CREATE2可以用于提前计算出将部署智能合约的地址，这允许实现被称为"反事实交互"的有趣新机制。

更多信息请参阅[EIP](https://eips.ethereum.org/EIPS/eip-1014#motivation)。

**FUNCTIONS**
deploy(amount, salt, bytecode)

computeAddress(salt, bytecodeHash)

computeAddress(salt, bytecodeHash, deployer)

**ERRORS**
Create2InsufficientBalance(balance, needed)

Create2EmptyBytecode()

Create2FailedDeployment()

#### deploy(uint256 amount, bytes32 salt, bytes bytecode) → address addr
internal#
使用CREATE2部署合约。合约将被部署的地址可以通过*computeAddress*提前知道。

合约的字节码可以通过Solidity的type(contractName).creationCode获取。

要求：
* 字节码不能为空。

* salt不能已经被字节码使用过。

* 工厂必须至少有amount的余额。

* 如果amount不为零，字节码必须有一个可支付的构造函数。

#### computeAddress(bytes32 salt, bytes32 bytecodeHash) → address
internal#
返回如果通过部署进行*部署*，合约将被存储的地址。任何在字节码哈希或salt中的改变都将导致一个新的目标地址。

#### computeAddress(bytes32 salt, bytes32 bytecodeHash, address deployer) → address addr
internal#
如果通过位于deployer的合约*部署*，返回将存储合约的地址。如果deployer是此合约的地址，返回与*computeAddress*相同的值。

#### Create2InsufficientBalance(uint256 balance, uint256 needed)
error#
没有足够的余额来执行CREATE2部署。

#### Create2EmptyBytecode()
error#
没有代码可以部署。

#### Create2FailedDeployment()
error#
部署失败。

### [Address](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/utils/Address.sol)
```
import "@openzeppelin/contracts/utils/Address.sol";
```

地址类型相关函数的集合

**FUNCTIONS**
sendValue(recipient, amount)

functionCall(target, data)

functionCallWithValue(target, data, value)

functionStaticCall(target, data)

functionDelegateCall(target, data)

verifyCallResultFromTarget(target, success, returndata)

verifyCallResult(success, returndata)

**ERRORS**
AddressInsufficientBalance(account)

AddressEmptyCode(target)

FailedInnerCall()

#### sendValue(address payable recipient, uint256 amount)
internal#
替代Solidity的transfer：将金额wei发送给收款人，转发所有可用的gas并在错误时回滚。

[EIP1884](https://eips.ethereum.org/EIPS/eip-1884)增加了某些操作码的gas成本，可能使合约超过transfer强加的2300 gas限制，使它们无法通过transfer接收资金。*sendValue*消除了这个限制。

[了解更多](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/)。

> IMPORTANT
因为控制权被转移到收款人，必须小心不要创建重入漏洞。考虑使用*ReentrancyGuard*或者[检查-效果-交互模式](https://solidity.readthedocs.io/en/v0.8.20/security-considerations.html#use-the-checks-effects-interactions-pattern)。

#### functionCall(address target, bytes data) → bytes
internal#
执行一个使用低级调用的Solidity函数调用。普通调用是函数调用的不安全替代品：请使用此函数代替。

如果目标以恢复原因或自定义错误恢复，则此函数会冒泡上来（就像常规Solidity函数调用一样）。然而，如果调用没有返回原因就恢复，那么这个函数就会因为*FailedInnerCall*错误而恢复。

返回原始返回的数据。要转换为预期的返回值，请使用[abi.decode](https://solidity.readthedocs.io/en/latest/units-and-global-variables.html?highlight=abi.decode#abi-encoding-and-decoding-functions)。

要求：
* 目标必须是一个合约。

* 调用目标与数据不得恢复。

#### functionCallWithValue(address target, bytes data, uint256 value) → bytes
internal#
与*functionCall*相同，但也将价值wei转移到目标。

要求：
* 调用合约必须至少拥有价值的ETH余额。

* 被调用的Solidity函数必须是可支付的。

#### functionStaticCall(address target, bytes data) → bytes
internal#
与*functionCall*相同，但执行静态调用。

#### functionDelegateCall(address target, bytes data) → bytes
internal#
与*functionCall*相同，但执行委托调用。

#### verifyCallResultFromTarget(address target, bool success, bytes returndata) → bytes
internal#
工具用于验证对智能合约的低级调用是否成功，并在目标不是合约或者因调用不成功而产生的还原原因（回退到*FailedInnerCall*）时进行还原。

#### verifyCallResult(bool success, bytes returndata) → bytes
internal#
工具用于验证低级调用是否成功，并在不成功时进行回滚，通过冒泡回滚原因或默认的*FailedInnerCall*错误。

#### AddressInsufficientBalance(address account)
error#
账户的ETH余额不足以执行此操作。

#### AddressEmptyCode(address target)
error#
在目标处没有代码（它不是合约）。

#### FailedInnerCall()
error#
在目标处没有代码（它不是合约）。