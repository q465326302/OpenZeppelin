# Math
这些是与数学相关的实用工具。

## Libraries

### SafeMath
对Solidity的算术操作进行包装，添加了溢出检查。

Solidity中的算术操作会在溢出时进行包装。这很容易导致错误，因为程序员通常假设溢出会引发错误，这是高级编程语言的标准行为。SafeMath通过在操作溢出时回滚事务来恢复这种直觉。

使用这个库而不是未经检查的操作可以消除一整类错误，因此建议始终使用它。

**FUNCTIONS**
add(a, b)

sub(a, b)

sub(a, b, errorMessage)

mul(a, b)

div(a, b)

div(a, b, errorMessage)

mod(a, b)

mod(a, b, errorMessage)

#### add(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的加法结果，如果溢出则返回反转。

与Solidity的+运算符相对应。

要求：- 加法不能溢出。

#### sub(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的减法结果，在溢出时返回反向结果（即结果为负数）。

与Solidity的-运算符相对应。

要求：- 减法不能溢出。

#### sub(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数的减法结果，并在溢出时返回自定义消息（当结果为负数时）。

与Solidity的-运算符相对应。

要求：减法不能溢出。

*自v2.4.0起可用。*

#### mul(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的乘积，如果溢出则返回反转。

与Solidity的*运算符相对应。

要求：-乘法不能溢出。

#### div(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数的整数除法结果。在除以零时会回滚。结果向零舍入。

与Solidity的/运算符相对应。注意：此函数使用回滚操作码（保留剩余的gas），而Solidity使用无效操作码来回滚（消耗所有剩余的gas）。

要求：- 除数不能为零。

#### div(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数的整数除法结果。在除以零时返回自定义消息并回滚。结果向零舍入。

这是 Solidity 中 / 运算符的对应函数。注意：该函数使用 revert 操作码（保留剩余的 gas），而 Solidity 使用无效操作码来回滚（消耗所有剩余的 gas）。

要求：- 除数不能为零。

*从 v2.4.0 版本开始可用。*

#### mod(uint256 a, uint256 b) → uint256
内部#
返回两个无符号整数相除的余数（无符号整数取模），当除以零时回滚。

这个函数是 Solidity 中 % 运算符的对应函数。该函数使用 revert 操作码（保留剩余的 gas），而 Solidity 使用无效操作码来回滚（消耗所有剩余的 gas）。

要求：- 除数不能为零。

#### mod(uint256 a, uint256 b, string errorMessage) → uint256
内部#
返回两个无符号整数相除的余数（无符号整数取模），在除以零时返回自定义消息。

与Solidity的%运算符相对应。此函数使用revert操作码（保留剩余的gas），而Solidity使用无效操作码来回滚（消耗所有剩余的gas）。

要求：-除数不能为零。

*自v2.4.0起可用。*

### Math
Solidity语言中缺少的标准数学工具。

**FUNCTIONS**
max(a, b)

min(a, b)

average(a, b)

#### max(uint256 a, uint256 b) → uint256
内部#
返回两个数字中较大的数字。

#### min(uint256 a, uint256 b) → uint256
内部#
返回两个数字中较小的数字。

#### average(uint256 a, uint256 b) → uint256
内部#
返回两个数字的平均值。结果向零舍入。