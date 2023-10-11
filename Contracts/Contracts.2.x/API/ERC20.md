# ERC 20
这组接口、合约和工具都与[ERC20代币标准](https://eips.ethereum.org/EIPS/eip-20)相关。

> TIP
要了解ERC20代币的概述并了解如何创建代币合约，请阅读我们的[ERC20指南](../Tokens/ERC20/ERC20.md)。


有几个核心合约实现了EIP中指定的行为：

* [IERC20](#ierc20)：所有ERC20实现都应符合的接口

* [ERC20](#erc20)：ERC20接口的基本实现

* [ERC20Detailed](#erc20detailed)：包括 [name](#name-→-string)、 [symbol](#symbol-→-string)和[decimals](#decimals-→-uint8)的可选标准扩展到基本接口

此外，还有多个自定义扩展，包括：

* 指定可以创建代币供应的地址（[ERC20Mintable](#erc20mintable)），可选的最大上限（[ERC20Capped](#erc20capped)）

* 销毁自己的代币（[ERC20Burnable](#erc20burnable)）

* 指定可以暂停所有用户的代币操作的地址（[ERC20Pausable](#erc20pausable)）。

最后，还有一些与ERC20合约进行各种交互的实用工具。

* [SafeERC20](#safeerc20)是一个包装器，可以消除处理布尔返回值的需要。

* [TokenTimelock](#tokentimelock)可以将代币保存给受益人，直到指定的时间。

> NOTE
此页面尚未完成。我们正在努力改进它以供下一个版本发布。敬请关注！

## 核心

### IERC20
ERC20标准的接口，如在EIP中定义的。不包括可选函数；要访问它们，请参阅[ERC20Detailed](#erc20detailed)。

**FUNCTIONS**
[totalSupply()](#totalsupply-→-uint256)

[balanceOf(account)](#balanceofaddress-account-→-uint256)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-→-bool)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-→-uint256)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-→-bool)

[transferFrom(sender, recipient, amount)](#transferaddress-recipient-uint256-amount-→-bool)

**EVENTS**
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

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

触发一个[Transfer](#transferaddress-from-address-to-uint256-value)事件。

#### allowance(address owner, address spender) → uint256
外部#
返回spender将被允许代表owner通过[transferFrom](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)花费的剩余代币数量。默认情况下，该值为零。

当调用[approve](#approveaddress-spender-uint256-amount-→-bool)或[transferFrom](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)时，此值会发生变化。

#### approve(address spender, uint256 amount) → bool
外部#
将金额设置为调用者代币上支出者的津贴。

返回一个布尔值，指示操作是否成功。

> IMPORTANT
请注意，使用此方法更改津贴存在风险，因为不幸的交易顺序可能导致某人同时使用旧的和新的津贴。缓解此竞争条件的一种可能解决方案是首先将支出者的津贴减少到0，然后再设置所需的值：https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件。

#### transferFrom(address sender, address recipient, uint256 amount) → bool
外部#
使用授权机制将数量代币从发送者移动到接收者。然后从调用者的授权中扣除数量。

返回一个布尔值，指示操作是否成功。

触发一个[Transfer](#transferaddress-from-address-to-uint256-value)事件。

#### Transfer(address from, address to, uint256 value)
事件#
当价值代币从一个账户（from）转移到另一个账户（to）时发出。

请注意，价值可能为零。

#### Approval(address owner, address spender, uint256 value)
事件#
当通过调用[approve](#approveaddress-spender-uint256-amount-→-bool)设置所有者的支出者的津贴时发出。 value是新的津贴金额。

### ERC20
[IERC20](#ierc20)接口的实现。

这个实现对于代币的创建方式是不可知的。这意味着在派生合约中必须使用[_mint](#_mintaddress-account-uint256-amount)添加一个供应机制。对于通用机制，请参阅[ERC20Mintable](#erc20mintable)。

> TIP
有关详细的撰写，请参阅我们的 [供应机制实施指南](https://forum.zeppelin.solutions/t/how-to-implement-erc20-supply-mechanisms/226)。

我们遵循了OpenZeppelin的一般准则：函数在失败时回滚，而不是返回false。尽管如此，这种行为是常规的，不会与ERC20应用程序的期望发生冲突。

此外，在调用[transferFrom](#transferaddress-recipient-uint256-amount-e28692-bool-1)时会发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件。这使得应用程序可以通过监听这些事件来重建所有账户的津贴。其他实现EIP的方式可能不会发出这些事件，因为规范中没有要求。

最后，非标准的[decreaseAllowance](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)和[increaseAllowance](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)函数已被添加以减轻围绕设置津贴的众所周知的问题。请参阅*IERC20.approve*。

**FUNCTIONS**
[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-e28692-bool-1)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-e28692-bool-1)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_burnFrom(account, amount)](#_burnfromaddress-account-uint256-amount)

CONTEXT
[constructor()](./GSN.md#constructoraddress-trustedsigner)

[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### totalSupply() → uint256
public#
请查阅  [IERC20.totalSupply](#totalsupply-→-uint256).

#### balanceOf(address account) → uint256
public#
请查阅 [IERC20.balanceOf](#balanceofaddress-account-→-uint256).

#### transfer(address recipient, uint256 amount) → bool
public#
请参阅[IERC20.Transfer](#transferaddress-recipient-uint256-amount-→-bool)。

要求：

* 接收者不能是零地址。

* 调用者必须拥有至少amount的余额。

#### allowance(address owner, address spender) → uint256
public#
请查阅 [IERC20.allowance](#allowanceaddress-owner-address-spender-→-uint256).

#### approve(address spender, uint256 amount) → bool
public#
请查阅 [IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)。

要求：

* spender 不能为零地址。

#### transferFrom(address sender, address recipient, uint256 amount) → bool
public#
参见[IERC20.transferFrom](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)。

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件，表示更新的授权额度。这不是EIP所要求的。请参阅[ERC20](#erc20)开头的注释；

要求：- 发送者和接收者不能是零地址。- 发送者必须至少拥有amount数量的余额。- 调用者必须至少拥有发送者的代币的amount数量的授权额度。

#### increaseAllowance(address spender, uint256 addedValue) → bool
public#
原子性地增加调用者授予给spender的津贴。

这是一个替代[approve](#approveaddress-spender-uint256-amount-e28692-bool-1)的方法，可以用作解决[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)中描述的问题的方法。

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件，指示更新的津贴。

要求：
* spender不能为零地址。

#### decreaseAllowance(address spender, uint256 subtractedValue) → bool
public#
以原子方式将调用者授予给spender的津贴减少。

这是一种替代[approve](#approveaddress-spender-uint256-amount-e28692-bool-1)的方法，可用于解决[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)中描述的问题。

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件，指示更新的津贴。

要求：
* spender不能是零地址。
* spender必须至少有subtractValue的津贴给调用者。

#### _transfer(address sender, address recipient, uint256 amount)
internal#
将代币数量从发送者转移到接收者。

这是一个内部函数，相当于[transfer](#transferaddress-recipient-uint256-amount-e28692-bool-1)，可以用于实现自动代币费用、减持机制等。

触发一个[transfer](#transferaddress-recipient-uint256-amount-e28692-bool-1)事件。

要求：
* 发送者不能是零地址。

* 接收者不能是零地址。

* 发送者必须拥有至少amount数量的余额。

#### _mint(address account, uint256 amount)
internal#
创建一定数量的代币并将其分配给账户，增加总供应量。

发出一个[transfer](#transferaddress-recipient-uint256-amount-e28692-bool-1)事件，将from设置为零地址。

要求

* to不能是零地址。

#### _burn(address account, uint256 amount)
internal#
销毁账户中的一定数量代币，从而减少总供应量。

发出一个[transfer](#transferaddress-recipient-uint256-amount-e28692-bool-1)事件，将to设置为零地址。

要求

* 账户不能是零地址。

* 账户必须至少拥有amount个代币。

#### _approve(address owner, address spender, uint256 amount)
internal#
将金额设置为spender对所有者代币的津贴。

这个内部函数等同于approve，并可以用于设置某些子系统的自动津贴等。

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件。

要求：

* 所有者不能是零地址。

* spender不能是零地址。

#### _burnFrom(address account, uint256 amount)
internal#
销毁账户中的代币数量。然后从调用者的可用额度中扣除相应的数量。

请参考[_burn](#_burnaddress-account-uint256-amount)和[_approve](#_approveaddress-owner-address-spender-uint256-amount)。

### ERC20Detailed
来自ERC20标准的可选功能。

**FUNCTIONS**
[constructor(name, symbol, decimals)](#constructorstring-name-string-symbol-uint8-decimals)

[name()](#name-→-string)

[symbol()](#symbol-→-string)

[decimals()](#decimals-→-uint8)

IERC20
[totalSupply()](#totalsupply-→-uint256)

[balanceOf(account)](#balanceofaddress-account-→-uint256)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-→-bool)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-→-uint256)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-→-bool)

[transferFrom(sender, recipient, amount)](#transferaddress-recipient-uint256-amount-→-bool)

**EVENTS**
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### constructor(string name, string symbol, uint8 decimals)
public#
设置name、symbol和decimals的值。这三个值都是不可变的：它们只能在构造过程中设置一次。

#### name() → string
public#
返回代币的名称。

#### symbol() → string
public#
返回代币的符号，通常是名称的缩写版本。

#### decimals() → uint8
public#
返回用于获取其用户表示的小数位数。例如，如果小数位数为2，则用户的余额为505个代币应显示为5.05（505 / 10 ** 2）。

代币通常选择18个小数位，模仿以太和Wei之间的关系。

> NOTE
这些信息仅用于显示目的，不会影响合约的任何算术运算，包括[IERC20.balanceOf](#balanceofaddress-account-→-uint256)和[IERC20.Transfer](#transferaddress-recipient-uint256-amount-→-bool)。

## 扩展

### ERC20Mintable
[ERC20](#erc20)的扩展版本，添加了一组具有[MinterRole](#mintaddress-account-uint256-amount-→-bool)的账户，这些账户有权根据自己的意愿铸造（创建）新的代币。

在构建时，合约的部署者是唯一的铸造者。

**MODIFIERS**
MINTERROLE
[onlyMinter()](./Access.md#onlyminter)

**FUNCTIONS**
[mint(account, amount)](#mintaddress-account-uint256-amount-→-bool)

MINTERROLE
[constructor()](./Access.md#constructor)

[isMinter(account)](./Access.md#isminteraddress-account-→-bool)

[addMinter(account)](./Access.md#addminteraddress-account)

[renounceMinter()](./Access.md#renounceminter)

[_addMinter(account)](./Access.md#_addminteraddress-account)

[_removeMinter(account)](./Access.md#_removeminteraddress-account)

ERC20
[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-e28692-bool-1)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-e28692-bool-1)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_burnFrom(account, amount)](#_burnfromaddress-account-uint256-amount)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
MINTERROLE
[MinterAdded(account)](./Access.md#minteraddedaddress-account)

[MinterRemoved(account)](./Access.md#minterremovedaddress-account)

IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### mint(address account, uint256 amount) → bool
public#
请参考[ERC20._mint](#_mintaddress-account-uint256-amount)。

要求：

* 调用者必须具有[MinterRole](./Access.md#minterrole)身份。

#### ERC20Burnable

[ERC20](./ERC20.md)的扩展，允许代币持有者销毁自己拥有的代币以及其授权的代币，以一种可以通过链外事件分析识别的方式。

**FUNCTIONS**
[burn(amount)](#burnuint256-amount)

[burnFrom(account, amount)](#burnfromaddress-account-uint256-amount)

ERC20
[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-e28692-bool-1)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-e28692-bool-1)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_burnFrom(account, amount)](#_burnfromaddress-account-uint256-amount)

CONTEXT
[constructor()](./GSN.md#constructoraddress-trustedsigner)

[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### burn(uint256 amount)
public#
销毁来自调用者的代币数量。

参见[ERC20._burn](#_burnaddress-account-uint256-amount)。

#### burnFrom(address account, uint256 amount)
public#
请查阅 [ERC20._burnFrom](#_burnfromaddress-account-uint256-amount).

### ERC20Pausable
带有可暂停转账和授权功能的ERC20。

如果你希望在众筹结束之前停止交易，或者在出现严重错误时紧急冻结所有代币转账，这将非常有用。

**MODIFIERS**
PAUSABLE
[whenNotPaused()](./Lifecycle.md#whennotpaused)

[whenPaused()](./Lifecycle.md#whenpaused)

PAUSERROLE
[onlyPauser()](./Access.md#onlypauser)

**FUNCTIONS**
[transfer(to, value)](#transferaddress-to-uint256-value-→-bool)

[transferFrom(from, to, value)](#transferfromaddress-from-address-to-uint256-value-→-bool)

[approve(spender, value)](#approveaddress-spender-uint256-value-→-bool)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-e28692-bool-1)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-e28692-bool-1)

PAUSABLE
[constructor()](./Lifecycle.md#constructor)

[paused()](./Lifecycle.md#paused-→-bool)

[pause()](./Lifecycle.md#pause)

[unpause()](./Lifecycle.md#unpause)

PAUSERROLE
[isPauser(account)](./Access.md#ispauseraddress-account-→-bool)

[addPauser(account)](./Access.md#addpauseraddress-account)

[renouncePauser()](./Access.md#renouncepauser)

[_addPauser(account)](./Access.md#_addpauseraddress-account)

[_removePauser(account)](./Access.md#_removepauseraddress-account)

ERC20
[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_burnFrom(account, amount)](#_burnfromaddress-account-uint256-amount)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
PAUSABLE
[Paused(account)](./Lifecycle.md#paused-→-bool)

[Unpaused(account)](./Lifecycle.md#unpausedaddress-account)

PAUSERROLE
[PauserAdded(account)](./Access.md#pauseraddedaddress-account)

[PauserRemoved(account)](./Access.md#pauserremovedaddress-account)

IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### transfer(address to, uint256 value) → bool
public#

#### transferFrom(address from, address to, uint256 value) → bool
public#

#### approve(address spender, uint256 value) → bool
public#

#### increaseAllowance(address spender, uint256 addedValue) → bool
public#

#### decreaseAllowance(address spender, uint256 subtractedValue) → bool
public#


### ERC20Capped
[ERC20Mintable](#erc20mintable)的扩展，为代币的供应添加了上限。

**MODIFIERS**
MINTERROLE
[onlyMinter()](./Access.md#onlyminter)

**FUNCTIONS**
[constructor(cap)](#constructoruint256-cap)

[cap()](#cap-→-uint256)

[_mint(account, value)](#_mintaddress-account-uint256-value)

ERC20MINTABLE
[mint(account, amount)](#mintaddress-account-uint256-amount-→-bool)

MINTERROLE
[isPauser(account)](./Access.md#ispauseraddress-account-→-bool)

[addPauser(account)](./Access.md#addpauseraddress-account)

[renouncePauser()](./Access.md#renouncepauser)

[_addPauser(account)](./Access.md#_addpauseraddress-account)

[_removePauser(account)](./Access.md#_removepauseraddress-account)

ERC20
[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-e28692-bool-1)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-e28692-bool-1)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_burnFrom(account, amount)](#_burnfromaddress-account-uint256-amount)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)


**EVENTS**
MINTERROLE
[MinterAdded(account)](./Access.md#minteraddedaddress-account)

[MinterRemoved(account)](./Access.md#minterremovedaddress-account)

IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### constructor(uint256 cap)
public#
设置上限的值。这个值是不可变的，只能在构造过程中设置一次。

#### cap() → uint256
public#
返回代币总供应量的上限。

#### _mint(address account, uint256 value)
internal#
请参阅[ERC20Mintable.mint](#mintaddress-account-uint256-amount-→-bool)。

要求：

* value的值不能导致总供应量超过上限。

## Utilities

### SafeERC20
在ERC20操作周围包装器，当代币合约返回false时会抛出异常。也支持返回无值（而是在失败时回滚或抛出异常）的代币，假设非回滚调用是成功的。要使用这个库，你可以在你的合约中添加一个using SafeERC20 for ERC20;语句，这样你就可以调用安全操作，如token.safeTransfer(…​)等。

**FUNCTIONS**
[safeTransfer(token, to, value)](#safetransfercontract-ierc20-token-address-to-uint256-value)

[safeTransferFrom(token, from, to, value)](#safetransferfromcontract-ierc20-token-address-from-address-to-uint256-value)

[safeApprove(token, spender, value)](#safeapprovecontract-ierc20-token-address-spender-uint256-value)

[safeIncreaseAllowance(token, spender, value)](#safeincreaseallowancecontract-ierc20-token-address-spender-uint256-value)

[safeDecreaseAllowance(token, spender, value)](#safedecreaseallowancecontract-ierc20-token-address-spender-uint256-value)

#### safeTransfer(contract IERC20 token, address to, uint256 value)
internal#

#### safeTransferFrom(contract IERC20 token, address from, address to, uint256 value)
internal#

#### safeApprove(contract IERC20 token, address spender, uint256 value)
internal#

#### safeIncreaseAllowance(contract IERC20 token, address spender, uint256 value)
internal#

#### safeDecreaseAllowance(contract IERC20 token, address spender, uint256 value)
internal#

### TokenTimelock
一个代币持有者合约，允许受益人在给定的释放时间后提取代币。

适用于简单的解锁计划，例如“顾问在1年后获得所有代币”。

对于更完整的解锁计划，请参阅[TokenVesting](./Drafts.md#tokenvesting)。

**FUNCTIONS**
[constructor(token, beneficiary, releaseTime)](#constructorcontract-ierc20-token-address-beneficiary-uint256-releasetime)

[token()](#token-→-contract-ierc20)

[beneficiary()](#beneficiary-→-address)

[releaseTime()](#releasetime-→-uint256)

[release()](#release)

#### constructor(contract IERC20 token, address beneficiary, uint256 releaseTime)
public#

#### token() → contract IERC20
public#

#### beneficiary() → address
public#

#### releaseTime() → uint256
公开

#### release()
public#