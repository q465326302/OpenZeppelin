# ERC 20
这组接口、合约和实用工具都与[ERC20代币标准](https://eips.ethereum.org/EIPS/eip-20)相关。

>TIP
要了解ERC20代币的概述和如何创建代币合约的步骤，请阅读我们的*ERC20指南*。

有一些核心合约实现了EIP中指定的行为：

* *IERC20*：所有ERC20实现都应符合的接口。
* *IERC20Metadata*：扩展的ERC20接口，包括*名称*、*符号*和*小数*函数。
* ERC20：实现ERC20接口，包括*名称*、*符号*和*小数*可选标准扩展到基本接口。

此外，还有多个自定义扩展，包括：

* *ERC20Burnable*：销毁自己的代币。
* *ERC20Capped*：在铸造代币时强制执行总供应量上限。
* *ERC20Pausable*：暂停代币转移的能力。
* *ERC20Snapshot*：高效存储过去的代币余额，以便随时查询。
* *ERC20Permit*：代币的无需燃气的批准（标准化为ERC2612）。
* *ERC20FlashMint*：代币级别支持闪电贷款，通过铸造和销毁短暂的代币（标准化为ERC3156）。
* *ERC20Votes*：支持投票和投票委托。
* *ERC20VotesComp*：支持投票和投票委托（与Compound的代币兼容，具有uint96限制）。
* *ERC20Wrapper*：包装器，用于创建由另一个ERC20支持的ERC20，具有存款和取款方法。与*ERC20Votes*结合使用非常有用。
* *ERC4626*：代币化的保险库，管理由资产（另一个ERC20）支持的股份（表示为ERC20）。

最后，有一些与ERC20合约以各种方式交互的实用程序。

* *SafeERC20*：接口的包装器，消除了处理布尔返回值的需要。
* *TokenTimelock*：将代币保留给受益人，直到指定时间。

>NOTE这组核心合约旨在不具有偏见，允许开发人员访问ERC20中的内部函数（如*_mint*）并以他们喜欢的方式将它们公开为外部函数。另一方面，*ERC20预设*（如*ERC20PresetMinterPauser*）使用偏见模式设计，为开发人员提供可用于部署的合约。

## 核心

### IERC20
```
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```
ERC20标准的接口定义，如EIP中所述。

**FUNCTIONS**
totalSupply()
balanceOf(account)
transfer(to, amount)
allowance(owner, spender)
approve(spender, amount)
transferFrom(from, to, amount)

**EVENTS**
Transfer(from, to, value)
Approval(owner, spender, value)

#### totalSupply() → uint256
外部#
返回存在的令牌数量。

#### balanceOf(address account) → uint256
外部#
返回账户拥有的代币数量。

#### transfer(address to, uint256 amount) → bool
外部#
将数量代币从调用者的账户转移到to账户。
返回一个布尔值，指示操作是否成功。
发出一个*转账*事件。

#### allowance(address owner, address spender) → uint256
外部#
返回spender将被允许通过*transferFrom*代表owner花费的剩余代币数量。默认情况下为零。
当调用*approve*或*transferFrom*时，此值会发生变化。

#### approve(address spender, uint256 amount) → bool
外部#
将金额设置为调用者代币的支出。
返回一个布尔值，指示操作是否成功。
>IMPORTANT
请注意，使用此方法更改津贴存在的风险，因为某些人可能会通过不幸的交易顺序同时使用旧的和新的津贴。解决这种竞争条件的一种可能的方法是先将支出者的津贴减少到0，然后再设置所需的值：https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729

发出一个*Approval*事件。

#### transferFrom(address from, address to, uint256 amount) → bool
外部#
使用津贴机制，将一定数量的代币从一个账户移动到另一个账户。移动后，调用者的津贴会相应减少。
返回一个布尔值，表示操作是否成功。
触发一个*转账*事件。

#### Transfer(address indexed from, address indexed to, uint256 value)
事件#
当价值代币从一个帐户（从）转移到另一个帐户（到）时发出。
请注意，价值可能为零。

#### Approval(address indexed owner, address indexed spender, uint256 value)
事件#
当调用*approve*设置所有者的支出者的津贴时，发出津贴设置事件。 value是新的津贴。

### IERC20Metadata
```
import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
```
ERC20标准的可选元数据功能的接口。
*自v4.1版本起可用。*

**FUNCTIONS**
name()
symbol()
decimals()

IERC20
totalSupply()
balanceOf(account)
transfer(to, amount)
allowance(owner, spender)
approve(spender, amount)
transferFrom(from, to, amount)

**EVENTS**

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### name() → string
外部#
返回令牌的名称。

#### symbol() → string
外部#
返回令牌的符号。

#### decimals() → uint8 
外部#
返回令牌的小数位数。

### ERC20
```
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```
实现*IERC20*接口。

该实现与代币创建方式无关。这意味着必须在派生合约中使用*_mint*添加供应机制。对于通用机制，请参阅*ERC20PresetMinterPauser*。

>TIP
有关详细信息，请参阅我们的指南 *如何实现供应机制*。

*小数*的默认值为18。要更改此值，您应该重写此函数以返回不同的值。

我们遵循通用的OpenZeppelin Contracts指南：函数在失败时返回而不是返回false。尽管这种行为是传统的，但并不与ERC20应用程序的期望冲突。

此外，在调用*transferFrom*时会发出*Approval*事件。这允许应用程序通过监听这些事件来重构所有帐户的津贴。其他实现EIP的可能不会发出这些事件，因为规范并不要求。

最后，非标准的*decreaseAllowance*和*increaseAllowance*函数已添加以减轻设置津贴时存在的众所周知的问题。请参阅*IERC20.approve*。

**FUNCTIONS**
constructor(name_, symbol_)
name()
symbol()
decimals()
totalSupply()
balanceOf(account)
transfer(to, amount)
allowance(owner, spender)
approve(spender, amount)
transferFrom(from, to, amount)
increaseAllowance(spender, addedValue)
decreaseAllowance(spender, subtractedValue)
_transfer(from, to, amount)
_mint(account, amount)
_burn(account, amount)
_approve(owner, spender, amount)
_spendAllowance(owner, spender, amount)
_beforeTokenTransfer(from, to, amount)
_afterTokenTransfer(from, to, amount)

**EVENTS**

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### constructor(string name_, string symbol_)
公开#
设置*名称*和*符号*的值。
这两个值都是不可变的：它们只能在构建期间设置一次。

#### name() → string
公开#
返回令牌的名称。

#### symbol() → string
公开#
返回代币的符号，通常是名称的缩写。

#### decimals() → uint8
公开#
返回用于用户表示的小数位数。例如，如果小数位数为2，则505个代币的余额应该显示为5.05（505 / 10 ** 2）。
代币通常选择18的值，模仿以太和Wei之间的关系。除非被覆盖，否则该函数返回默认值18。

>NOTE
这些信息仅用于显示目的，它不会影响合同的任何算术，包括*IERC20.balanceOf*和*IERC20.transfer*。

#### totalSupply() → uint256
公开#
请查看*IERC20.totalSupply*。

#### balanceOf(address account) → uint256
公开#
请查看*IERC20.balanceOf*.

#### transfer(address to, uint256 amount) → bool
公开#
请参阅*IERC20.transfer*。
要求：
* to地址不能为零地址。
* 调用者必须至少拥有amount数量的余额。

#### allowance(address owner, address spender) → uint256
公开#
请查看 *IERC20.allowance*.

#### approve(address spender, uint256 amount) → bool
公开#
请查看*IERC20.approve*。

>NOTE
如果金额为最大的uint256，则在transferFrom上不会更新授权。这在语义上等同于无限授权。

要求：
* spender不能是零地址。

#### transferFrom(address from, address to, uint256 amount) → bool
公开#
查看*IERC20.transferFrom*。
触发一个*Approval*事件，表示更新的授权额度。这不是EIP所必需的。请参见*ERC20*开头的注释。

>NOTE
如果当前授权额度是最大的uint256，则不会更新授权额度。

要求：
* from和to不能是零地址。
* from必须至少拥有amount的余额。
* 调用者必须对from的代币拥有至少amount的授权额度。

#### increaseAllowance(address spender, uint256 addedValue) → bool
公开#
原子地增加调用者授予给spender的授权金额。
这是*approve*的一种替代方案，可用作解决*IERC20.approve*中描述的问题的方法。
发出一个*Approval*事件，指示更新的授权金额。
要求：
* spender不能为零地址。

#### decreaseAllowance(address spender, uint256 subtractedValue) → bool
以原子方式减少调用者授予spender的津贴。

这是可以用作对*IERC20.approve*中描述的问题的缓解措施的*approve*的替代方法。

发出一个*Approval*事件，指示更新的津贴。

要求：
* spender不能是零地址。
* spender必须至少具有减去值的调用者的津贴。

#### _transfer(address from, address to, uint256 amount)
内部#
将一定数量的代币从发送者账户转移到接收者账户。
这个内部函数等同于*transfer*函数，可以用于实现自动代币费用、惩罚机制等。
触发一次*转账*事件。
要求：
* 发送者账户不能为0地址。
* 接收者账户不能为0地址。
* 发送者账户余额必须大于等于转移数量。

#### _mint(address account, uint256 amount)
内部#
创建一定数量的代币并将其分配给账户，增加总供应量。
发出一个*转账*事件，其中from地址设置为零地址。

要求：
* 账户不能是零地址。

#### _burn(address account, uint256 amount)
内部#
销毁账户中的代币，减少总供应量。
发出一个*转账*事件，把“to”设置为零地址。

要求：
* 账户不能是零地址。
* 账户必须至少拥有amount个代币。

#### _approve(address owner, address spender, uint256 amount)
内部#
将spender的数量设置为所有者代币的津贴。
该内部函数相当于approve，并可用于为某些子系统设置自动津贴等。
发出*Approval*事件。

要求：
* 所有者不能为零地址。
* spender不能为零地址。

#### _spendAllowance(address owner, address spender, uint256 amount)
内部#
根据已花费的金额更新支出者的拥有津贴。
如果拥有无限制的津贴，则不更新津贴金额。如果津贴金额不足，则撤销操作。
可能会发出一个*Approval*事件。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
内部#
在任何代币转移之前调用的挂钩。这包括铸造和销毁。
调用条件：
* 当 from 和 to 都不为零时，from 的代币数量将转移到 to。
* 当 from 为零时，将为 to 铸造 amount 个代币。
* 当 to 为零时，将销毁 from 的代币数量。
* from 和 to 永远不会同时为零。
要了解更多关于挂钩的信息，请前往*使用挂钩*。

#### _afterTokenTransfer(address from, address to, uint256 amount)
