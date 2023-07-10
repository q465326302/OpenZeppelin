# Math
这些是与数学相关的实用程序。

## 库

### SafeMath
对Solidity的算术操作进行包装，并添加了溢出检查。

Solidity中的算术操作在溢出时会进行包装。这很容易导致错误，因为程序员通常会假设溢出会引发错误，这是高级编程语言中的标准行为。SafeMath通过在操作溢出时回滚交易来恢复这种直觉。

使用这个库而不是不经检查的操作可以消除一整类错误，因此建议始终使用它。

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
返回两个无符号整数的加法结果，并带有溢出标志。

*自v3.4版本起可用。*

#### trySub(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数的减法结果，并附带一个溢出标志。

*自v3.4版本开始可用。*

#### tryMul(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数的乘积，并带有溢出标志。

*自v3.4起可用。*

#### tryDiv(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数的除法结果，并附带除以零的标志。

*自v3.4版本起可用。*

#### tryMod(uint256 a, uint256 b) → bool, uint256
内部#
返回两个无符号整数相除的余数，并附带除以零的标志。

*自v3.4版本开始提供。*

#### add(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的加法结果，当发生溢出时返回反向结果。

与Solidity的+操作符相对应。

要求：
* 加法不能溢出。

#### sub(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的减法结果，当结果为负数时会溢出。

这是 Solidity 中 - 运算符的对应函数。

要求：
* 减法不能溢出。

#### mul(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的乘积，如果溢出则返回反转。

与Solidity的*操作符相对应。

要求：
* 乘法不能溢出。

#### div(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的整数除法，如果除以零则返回错误。结果向零舍入。

这是 Solidity 中 / 运算符的对应函数。注意：该函数使用 revert 操作码（保留剩余的 gas），而 Solidity 使用无效操作码来回滚（消耗所有剩余的 gas）。

要求：
* 除数不能为零。

#### mod(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数相除的余数（无符号整数取模），在除以零时返回。

与Solidity的%运算符相对应。该函数使用revert操作码（保持剩余的gas不变），而Solidity使用无效操作码来回滚（消耗所有剩余的gas）。

要求：
* 除数不能为零。

#### sub(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数的减法结果，并在溢出时使用自定义消息进行回退（当结果为负数时）。

> CAUTION
该函数已被弃用，因为它不必要地需要为错误消息分配内存。对于自定义的回退原因，请使用*trySub*。

与Solidity的-运算符相对应。

要求：
* 减法不能溢出。

#### div(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数的整数除法，如果除以零则返回自定义消息。结果向零舍入。

> CAUTION
该函数已弃用，因为它不必要地需要分配内存来存储错误消息。如果需要自定义回滚原因，请使用tryDiv。

与Solidity的/运算符相对应。注意：该函数使用了一个回滚操作码（保留了剩余的gas），而Solidity使用了一个无效的操作码来回滚（消耗了所有剩余的gas）。

要求：
* 除数不能为零。

#### mod(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数相除的余数（无符号整数模数），在除以零时返回自定义消息。

该函数已弃用，因为它不必要地需要为错误消息分配内存。对于自定义的还原原因，请使用*tryMod*。

与Solidity的%运算符相对应。此函数使用revert操作码（保持剩余的gas不变），而Solidity使用无效操作码来还原（消耗所有剩余的gas）。

要求：
* 除数不能为零。

### SignedSafeMath
带有安全检查的有错误回滚功能的带符号数学运算。

**FUNCTIONS**
mul(a, b)

div(a, b)

sub(a, b)

add(a, b)

#### mul(int256 a, int256 b) → int256
内部#
返回两个有符号整数的乘法结果，如果溢出则返回反转。

与Solidity的*运算符相对应。

要求：
* 乘法不能溢出。

#### div(int256 a, int256 b) → int256
内部#
返回两个有符号整数的整数除法结果。在除以零时会回滚。结果向零舍入。

这是 Solidity 中 / 运算符的对应函数。注意：该函数使用回滚操作码（不消耗剩余的 gas），而 Solidity 使用无效操作码回滚（消耗所有剩余的 gas）。

要求：

* 除数不能为零。

#### sub(int256 a, int256 b) → int256
内部#
返回两个有符号整数的减法结果，如果溢出则返回反转结果。

与Solidity的-运算符相对应。

要求：
* 减法不能溢出。

#### add(int256 a, int256 b) → int256
内部#
返回两个有符号整数的加法结果，如果溢出则返回反转结果。

与Solidity的+运算符相对应。

要求：
* 加法不能溢出。

### 数学
Solidity语言中缺少标准数学工具。

**FUNCTIONS**
max(a, b)

min(a, b)

average(a, b)

#### max(uint256 a, uint256 b) → uint256
内部#
返回两个数字中的最大值。

#### min(uint256 a, uint256 b) → uint256
内部#
返回两个数字中较小的数字。

#### average(uint256 a, uint256 b) → uint256
内部#
返回两个数字的平均值。结果向零四舍五入。