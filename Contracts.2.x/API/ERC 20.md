# ERC 20
这组接口、合约和工具都与[ERC20令牌标准](https://eips.ethereum.org/EIPS/eip-20)相关。

> TIP
要了解ERC20令牌的概述并了解如何创建令牌合约，请阅读我们的*ERC20指南*。


有几个核心合约实现了EIP中指定的行为：

* IERC20：所有ERC20实现都应符合的接口

* ERC20：ERC20接口的基本实现

* ERC20Detailed：包括*名称*、*符号*和*小数点*的可选标准扩展到基本接口

此外，还有多个自定义扩展，包括：

* 指定可以创建令牌供应的地址（*ERC20Mintable*），可选的最大上限（*ERC20Capped*）

* 销毁自己的令牌（*ERC20Burnable*）

* 指定可以暂停所有用户的令牌操作的地址（*ERC20Pausable*）。

最后，还有一些与ERC20合约进行各种交互的实用工具。

* *SafeERC20*是一个包装器，可以消除处理布尔返回值的需要。

* *TokenTimelock*可以将代币保存给受益人，直到指定的时间。

> NOTE
此页面尚未完成。我们正在努力改进它以供下一个版本发布。敬请关注！

## 核心

### IERC20
ERC20标准的接口，如在EIP中定义的。不包括可选函数；要访问它们，请参阅*ERC20Detailed*。

**FUNCTIONS**
totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

**EVENTS**
Transfer(from, to, value)

Approval(owner, spender, value)

#### totalSupply() → uint256
外部#
返回现有的代币数量。

#### balanceOf(address account) → uint256
外部#
返回账户拥有的代币数量。

#### transfer(address recipient, uint256 amount) → bool
外部#
将一定数量的代币从调用者的账户转移到接收者的账户。

返回一个布尔值，表示操作是否成功。

触发一个*Transfer*事件。

#### allowance(address owner, address spender) → uint256
外部#
返回spender将被允许代表owner通过*transferFrom*花费的剩余代币数量。默认情况下，该值为零。

当调用*approve*或*transferFrom*时，此值会发生变化。

#### approve(address spender, uint256 amount) → bool
外部#
将金额设置为调用者令牌上支出者的津贴。

返回一个布尔值，指示操作是否成功。

> IMPORTANT
请注意，使用此方法更改津贴存在风险，因为不幸的交易顺序可能导致某人同时使用旧的和新的津贴。缓解此竞争条件的一种可能解决方案是首先将支出者的津贴减少到0，然后再设置所需的值：https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729

发出一个*Approval*事件。

#### transferFrom(address sender, address recipient, uint256 amount) → bool
外部#
使用授权机制将数量令牌从发送者移动到接收者。然后从调用者的授权中扣除数量。

返回一个布尔值，指示操作是否成功。

触发一个*Transfer*事件。

#### Transfer(address from, address to, uint256 value)
事件#
当价值代币从一个账户（from）转移到另一个账户（to）时发出。

请注意，价值可能为零。

#### Approval(address owner, address spender, uint256 value)
事件#
当通过调用*approve*设置所有者的支出者的津贴时发出。 value是新的津贴金额。

### ERC20
*IERC20接口*的实现。

这个实现对于代币的创建方式是不可知的。这意味着在派生合约中必须使用*_mint*添加一个供应机制。对于通用机制，请参阅*ERC20Mintable*。

> TIP
有关详细的撰写，请参阅我们的 [供应机制实施指南](https://forum.zeppelin.solutions/t/how-to-implement-erc20-supply-mechanisms/226)。

我们遵循了OpenZeppelin的一般准则：函数在失败时回滚，而不是返回false。尽管如此，这种行为是常规的，不会与ERC20应用程序的期望发生冲突。

此外，在调用*transferFrom*时会发出一个*Approval*事件。这使得应用程序可以通过监听这些事件来重建所有账户的津贴。其他实现EIP的方式可能不会发出这些事件，因为规范中没有要求。

最后，非标准的*decreaseAllowance*和*increaseAllowance*函数已被添加以减轻围绕设置津贴的众所周知的问题。请参阅*IERC20.approve*。

**FUNCTIONS**
totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

_transfer(sender, recipient, amount)

_mint(account, amount)

_burn(account, amount)

_approve(owner, spender, amount)

_burnFrom(account, amount)

CONTEXT
constructor()

_msgSender()

_msgData()

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### totalSupply() → uint256
公开#
请查阅  *IERC20.totalSupply*.

#### balanceOf(address account) → uint256
公开#
请查阅 *IERC20.balanceOf*.

#### transfer(address recipient, uint256 amount) → bool
公开#
请参阅IERC20.Transfer。

要求：

* 接收者不能是零地址。

* 调用者必须拥有至少amount的余额。

#### allowance(address owner, address spender) → uint256
公开#
请查阅  *IERC20.allowance*.

#### approve(address spender, uint256 amount) → bool
公开#
请查阅 *IERC20.approve*。

要求：

* spender 不能为零地址。

#### transferFrom(address sender, address recipient, uint256 amount) → bool
公开#
参见*IERC20.transferFrom*。

发出一个*Approval*事件，表示更新的授权额度。这不是EIP所要求的。请参阅*ERC20*开头的注释；

要求：- 发送者和接收者不能是零地址。- 发送者必须至少拥有amount数量的余额。- 调用者必须至少拥有发送者的令牌的amount数量的授权额度。

#### increaseAllowance(address spender, uint256 addedValue) → bool
公开#
原子性地增加调用者授予给spender的津贴。

这是一个替代*approve*的方法，可以用作解决*IERC20.approve*中描述的问题的方法。

发出一个*Approval*事件，指示更新的津贴。

要求：
* spender不能为零地址。

#### decreaseAllowance(address spender, uint256 subtractedValue) → bool
公开#
以原子方式将调用者授予给spender的津贴减少。

这是一种替代*approve*的方法，可用于解决*IERC20.approve*中描述的问题。

发出一个Approval事件，指示更新的津贴。

要求：
* spender不能是零地址。
* spender必须至少有subtractValue的津贴给调用者。

#### _transfer(address sender, address recipient, uint256 amount)
内部#
将令牌数量从发送者转移到接收者。

这是一个内部函数，相当于*转账*，可以用于实现自动令牌费用、减持机制等。

触发一个*转账*事件。

要求：
* 发送者不能是零地址。

* 接收者不能是零地址。

* 发送者必须拥有至少amount数量的余额。

#### _mint(address account, uint256 amount)
内部#
创建一定数量的代币并将其分配给账户，增加总供应量。

发出一个*转账*事件，将from设置为零地址。

要求

* to不能是零地址。

#### _burn(address account, uint256 amount)
内部#
销毁账户中的一定数量代币，从而减少总供应量。

发出一个*转账*事件，将to设置为零地址。

要求

* 账户不能是零地址。

* 账户必须至少拥有amount个代币。

#### _approve(address owner, address spender, uint256 amount)
内部#
将金额设置为spender对所有者代币的津贴。

这个内部函数等同于approve，并可以用于设置某些子系统的自动津贴等。

发出一个*Approval*事件。

要求：

* 所有者不能是零地址。

* spender不能是零地址。

#### _burnFrom(address account, uint256 amount)
内部#
销毁账户中的代币数量。然后从调用者的可用额度中扣除相应的数量。

请参考*_burn*和*_approve*。

### ERC20Detailed
来自ERC20标准的可选功能。

**FUNCTIONS**
constructor(name, symbol, decimals)

name()

symbol()

decimals()

IERC20
totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### constructor(string name, string symbol, uint8 decimals)
公开#
设置name、symbol和decimals的值。这三个值都是不可变的：它们只能在构造过程中设置一次。

#### name() → string
公开#
返回令牌的名称。

#### symbol() → string
公开#
返回令牌的符号，通常是名称的缩写版本。

#### decimals() → uint8
公开#
返回用于获取其用户表示的小数位数。例如，如果小数位数为2，则用户的余额为505个代币应显示为5.05（505 / 10 ** 2）。

代币通常选择18个小数位，模仿以太币和Wei之间的关系。

> NOTE
这些信息仅用于显示目的，不会影响合约的任何算术运算，包括IERC20.balanceOf和IERC20.Transfer。

## 扩展

### ERC20Mintable
*ERC20*的扩展版本，添加了一组具有*MinterRole*的账户，这些账户有权根据自己的意愿铸造（创建）新的代币。

在构建时，合约的部署者是唯一的铸造者。

**MODIFIERS**
MINTERROLE
onlyMinter()

**FUNCTIONS**
mint(account, amount)

MINTERROLE
constructor()

isMinter(account)

addMinter(account)

renounceMinter()

_addMinter(account)

_removeMinter(account)

ERC20
totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

_transfer(sender, recipient, amount)

_mint(account, amount)

_burn(account, amount)

_approve(owner, spender, amount)

_burnFrom(account, amount)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
MINTERROLE
MinterAdded(account)

MinterRemoved(account)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### mint(address account, uint256 amount) → bool
公开#
请参考*ERC20._mint*。

要求：

* 调用者必须具有*MinterRole*身份。

#### ERC20Burnable

*ERC20*的扩展，允许代币持有者销毁自己拥有的代币以及其授权的代币，以一种可以通过链外事件分析识别的方式。

**FUNCTIONS**
burn(amount)

burnFrom(account, amount)

ERC20
totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

_transfer(sender, recipient, amount)

_mint(account, amount)

_burn(account, amount)

_approve(owner, spender, amount)

_burnFrom(account, amount)

CONTEXT
constructor()

_msgSender()

_msgData()

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### burn(uint256 amount)
公开#
销毁来自调用者的代币数量。

参见*ERC20._burn*。

#### burnFrom(address account, uint256 amount)
公开#
请查阅 *ERC20._burnFrom*.

### ERC20Pausable
带有可暂停转账和授权功能的ERC20。

如果您希望在众筹结束之前停止交易，或者在出现严重错误时紧急冻结所有代币转账，这将非常有用。

**MODIFIERS**
PAUSABLE
whenNotPaused()

whenPaused()

PAUSERROLE
onlyPauser()

**FUNCTIONS**
transfer(to, value)

transferFrom(from, to, value)

approve(spender, value)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

PAUSABLE
constructor()

paused()

pause()

unpause()

PAUSERROLE
isPauser(account)

addPauser(account)

renouncePauser()

_addPauser(account)

_removePauser(account)

ERC20
totalSupply()

balanceOf(account)

allowance(owner, spender)

_transfer(sender, recipient, amount)

_mint(account, amount)

_burn(account, amount)

_approve(owner, spender, amount)

_burnFrom(account, amount)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
PAUSABLE
Paused(account)

Unpaused(account)

PAUSERROLE
PauserAdded(account)

PauserRemoved(account)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### transfer(address to, uint256 value) → bool
公开#

#### transferFrom(address from, address to, uint256 value) → bool
公开#

#### approve(address spender, uint256 value) → bool
公开#

#### increaseAllowance(address spender, uint256 addedValue) → bool
公开#

#### decreaseAllowance(address spender, uint256 subtractedValue) → bool
公开#


### ERC20Capped
*ERC20Mintable*的扩展，为代币的供应添加了上限。

**MODIFIERS**
MINTERROLE
onlyMinter()

**FUNCTIONS**
constructor(cap)

cap()

_mint(account, value)

ERC20MINTABLE
mint(account, amount)

MINTERROLE
isMinter(account)

addMinter(account)

renounceMinter()

_addMinter(account)

_removeMinter(account)

ERC20
totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

_transfer(sender, recipient, amount)

_burn(account, amount)

_approve(owner, spender, amount)

_burnFrom(account, amount)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
MINTERROLE
MinterAdded(account)

MinterRemoved(account)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)