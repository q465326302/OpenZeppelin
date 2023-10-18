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