# ERC 20
这组接口、合约和实用程序都与[ERC20代币标准](https://eips.ethereum.org/EIPS/eip-20)有关。

> TIP
关于ERC20代币的概述和创建代币合约的步骤，请阅读我们的[ERC20指南](../Tokens/ERC20/ERC20.md)。

有几个核心合约实现了EIP中指定的行为：

* [IERC20](#ierc20)：所有ERC20实现都应符合的接口。

* [ERC20](#erc20)：实现了ERC20接口，包括[name](#name-→-string), [symbol](#symbol-→-string)和 [decimals](#decimals-→-uint8)的可选标准扩展。

此外还有多个自定义扩展，包括：

* [ERC20Permit](#erc20pausable)：无需消耗燃料即可批准代币。

* [ERC20Snapshot](#erc20snapshot)：高效存储过去的代币余额，以便随时查询。

* [ERC20Burnable](#erc20burnable)：销毁自己的代币。

* [ERC20Capped](#erc20capped)：在铸造代币时对总供应量进行限制。

* [ERC20Pausable](#erc20pausable)：暂停代币转账的能力。

最后，还有一些与ERC20合约交互的实用程序。

* [SafeERC20](#safeerc20)：对接口的封装，消除了处理布尔返回值的需要。

* [TokenTimelock](#tokentimelock)：将代币保留给受益人直到指定时间。

以下相关的EIP处于草案状态，可以在草案目录中找到。

* [IERC20Permit](./Drafts.md#ierc20permit)

* [ERC20Permit](./Drafts.md#erc20permit)

> NOTE
这个核心合约集被设计为无偏见，允许开发人员访问ERC20的内部函数（如[_mint](#_mintaddress-account-uint256-amount)）并以他们喜欢的方式将其作为外部函数暴露出来。另一方面，[ERC20预设合约](../Tokens/ERC20/ERC20.md#预设erc20合约)（如[ERC20PresetMinterPauser](./Presets.md#erc20presetminterpauser)）使用有偏见的模式设计，以提供给开发人员可用的、可部署的合约。

## 核心

### IERC20
ERC20标准接口是根据EIP定义的。

**FUNCTIONS**
[totalSupply()](#totalsupply-→-uint256)

[balanceOf(account)](#balanceofaddress-account-→-uint256)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-→-bool)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-→-uint256)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-→-bool)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)

**EVENTS**
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### totalSupply() → uint256
外部#
返回存在的代币数量。

#### balanceOf(address account) → uint256
外部#
返回账户拥有的代币数量。

#### transfer(address recipient, uint256 amount) → bool
外部#
将指定数量的代币从调用者的账户转移到接收者的账户。

返回一个布尔值，表示操作是否成功。

触发一个[Transfer](#transferaddress-from-address-to-uint256-value)事件。

#### allowance(address owner, address spender) → uint256
外部#
返回spender将被允许代表owner通过[transferFrom](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)进行花费的剩余代币数量。默认情况下，该值为零。

当调用[approve](#approveaddress-spender-uint256-amount-→-bool)或[transferFrom](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool)时，此值会发生变化。

#### approve(address spender, uint256 amount) → bool
外部#
将spender的津贴设置为caller的代币数量。

返回一个布尔值，指示操作是否成功。

> IMPORTANT
请注意，使用此方法更改津贴存在一个风险，即某人可能通过不幸的交易顺序同时使用旧的和新的津贴。缓解这种竞态条件的一种可能解决方案是首先将spender的津贴减少到0，然后再设置所需的值：https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件。

#### transferFrom(address sender, address recipient, uint256 amount) → bool
外部#
使用津贴机制，将数量代币从发送者移动到接收者。然后从调用者的津贴中扣除该数量。

返回一个布尔值，指示操作是否成功。

发出一个[Transfer](#transferaddress-from-address-to-uint256-value)事件。

#### Transfer(address from, address to, uint256 value)
事件#
当价值代币从一个账户（from）转移到另一个账户（to）时发出。

请注意，价值可能为零。

#### Approval(address owner, address spender, uint256 value)
事件#
当通过调用[approve](#approveaddress-spender-uint256-amount-→-bool)设置所有者的支出者的津贴时发出。 value是新的津贴。

#### ERC20
[IERC20](#ierc20)接口的实现。

该实现对于代币的创建方式是不可知的。这意味着在派生合约中需要使用[_mint](#_mintaddress-account-uint256-amount)添加供应机制。对于通用机制，请参见[ERC20PresetMinterPauser](./Presets.md#erc20presetminterpauser)。

> TIP
有关详细信息，请参阅我们的指南：如何实现供应机制。

我们遵循了OpenZeppelin的一般准则：函数在失败时回滚而不是返回false。尽管如此，这种行为是传统的，并且不与ERC20应用程序的期望相冲突。

此外，在调用[transferFrom](#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)时会发出[Approval](#approvaladdress-owner-address-spender-uint256-value)事件。这允许应用程序通过监听这些事件来重建所有帐户的津贴。其他实现EIP的方式可能不会发出这些事件，因为规范不要求发出。

最后，为了缓解围绕设置津贴的众所周知的问题，我们添加了非标准的[decreaseAllowance](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)和[increaseAllowance](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)函数。请参见[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)。

**FUNCTIONS**
[constructor(name_, symbol_)](#constructorstring-name_-string-symbol_)

[name()](#name-→-string)

[symbol()](#symbol-→-string)

[decimals()](#decimals-→-uint8)

[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](#_setupdecimalsuint8-decimals)

[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)

**EVENTS**
IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### constructor(string name_, string symbol_)
public#
设置[名称](#name-→-string)和[符号](#symbol-→-string)的值，并使用默认值18初始化[小数位](#decimals-→-uint8)。

要选择不同的[小数位](#decimals-→-uint8)值，请使用[_setupDecimals](#_setupdecimalsuint8-decimals)。

这三个值都是不可变的：它们只能在构造期间设置一次。

#### name() → string
public#
返回代币的名称。

#### symbol() → string
public#
返回代币的符号，通常是名称的缩写版本。

#### decimals() → uint8
public#
返回用于获取其用户表示的小数位数。例如，如果小数位数等于2，则用户的余额为505个代币应显示为5.05（505 / 10 ** 2）。

通常，代币选择18个小数位，模仿以太和Wei之间的关系。这是[ERC20](#erc20)使用的值，除非调用了[_setupDecimals](#_setupdecimalsuint8-decimals)。

> NOTE
此信息仅用于显示目的：它绝不会影响合约的任何算术操作，包括[IERC20.balanceOf](#balanceofaddress-account-→-uint256)和[IERC20.Transfer](#transferaddress-recipient-uint256-amount-→-bool)。

#### totalSupply() → uint256
public#
请参阅 [IERC20.totalSupply](#totalsupply-→-uint256).

#### balanceOf(address account) → uint256
public#
请参阅 [IERC20.balanceOf](#balanceofaddress-account-→-uint256).

#### transfer(address recipient, uint256 amount) → bool
public#
请参阅 [IERC20.Transfer](#transferaddress-from-address-to-uint256-value).
要求：
* 接收者不能是零地址。

* 调用者必须拥有至少等额的余额。

#### allowance(address owner, address spender) → uint256
public#
请参阅[IERC20.allowance](#allowanceaddress-owner-address-spender-→-uint256).

#### approve(address spender, uint256 amount) → bool
public#
请参阅 [IERC20.approve](#approveaddress-spender-uint256-amount-→-bool).
要求:
* 接收者（spender）不能是零地址。

#### transferFrom(address sender, address recipient, uint256 amount) → bool
public#
请参阅 [IERC20.transferFrom](#transferfromaddress-sender-address-recipient-uint256-amount-→-bool).
发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件，表示更新的津贴。根据EIP的要求，这不是必需的。请参阅[ERC20](#erc20)开头的注释。

要求：
* 发送者和接收者不能是零地址。

* 发送者的余额必须至少为amount。

* 调用者必须至少拥有发送者的代币的amount的津贴。

#### increaseAllowance(address spender, uint256 addedValue) → bool
public#
以调用者为基础，原子性地增加授权给spender的津贴。

这是一个可用作对[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)中描述的问题的缓解措施的替代方法。

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件，指示更新后的津贴。

要求：
* spender不能是零地址。

#### decreaseAllowance(address spender, uint256 subtractedValue) → bool
public#
以原子方式减少调用者授予给spender的津贴。

这是一种替代[approve](#approveaddress-spender-uint256-amount-e28692-bool-1)的方法，可以用作解决[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)中描述的问题的方法。

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件，指示更新后的津贴。

要求：
* spender不能是零地址。

* spender必须对调用者拥有至少减去的值的津贴。

#### _transfer(address sender, address recipient, uint256 amount)
internal#
将代币数量从发送方移动到接收方。

这个内部函数等同于[transfer](#transferaddress-recipient-uint256-amount-e28692-bool-1)，并且可以用于实现自动代币费用、削减机制等。

发出一个[transfer](#transferaddress-recipient-uint256-amount-e28692-bool-1)事件。

要求：
* 发送方不能是零地址。

* 接收方不能是零地址。

* 发送方必须至少有amount的余额。

#### _mint(address account, uint256 amount)
internal#
创建一定数量的代币，并将其分配给账户，增加总供应量。

触发一个[transfer](#transferaddress-from-address-to-uint256-value)事件，其中将from设置为零地址。

要求：
* to不能为零地址。

#### _burn(address account, uint256 amount)
internal#
销毁账户中的代币数量，从而减少总供应量。

发出一个[transfer](#transferaddress-from-address-to-uint256-value)事件，将目标地址设置为零地址。

要求：
* 账户不能是零地址。

* 账户必须至少拥有指定数量的代币。

#### _approve(address owner, address spender, uint256 amount)
internal#
将金额设置为拥有者的代币分配给支出者的金额。

这个内部函数等同于approve，并可以用于为某些子系统设置自动津贴等。

发出一个[Approval](#approvaladdress-owner-address-spender-uint256-value)事件。

要求:
* 拥有者不能是零地址。

* 支出者不能是零地址。

#### _setupdecimals(uint8 decimals)

internal#
将 [decimals](#decimals-→-uint8)设置为除默认值18以外的其他值。

> WARNING
此函数只应在构造函数中调用。与代币合约交互的大多数应用程序不会期望[decimals](#decimals-→-uint8)发生变化，并且如果发生变化可能会工作不正确。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#
在任何代币转移之前调用的 hooks 。这包括铸造和销毁。

调用条件：
* 当 from 和 to 都不为零时，将从 from 的代币转移给 to。

* 当 from 为零时，将为 to 铸造 amount 个代币。

* 当 to 为零时，将销毁 from 的 amount 个代币。

* from 和 to 永远不会同时为零。

要了解更多关于 hooks 的信息，请查看 [Using Hooks](../Extending-Contracts.md#使用hooks)页面。

## Extensions

### ERC20Snapshot
该合约通过快照机制扩展了一个ERC20代币。当创建快照时，会记录下当时的余额和总供应量以供以后访问。

这可以用于安全地创建基于代币余额的机制，例如无信任分红或加权投票。在简单的实现中，可能会通过重复使用不同账户的相同余额来进行“双重花费”攻击。通过使用快照来计算分红或投票权，这些攻击将不再适用。它还可以用于创建高效的ERC20分叉机制。

快照是通过内部的[_snapshot](#_snapshot-→-uint256)函数创建的，该函数将触发 [Snapshot](#snapshotuint256-id)事件并返回一个快照id。要获取快照时的总供应量，请调用[totalSupplyAt](#totalsupplyatuint256-snapshotid-→-uint256)函数并提供快照id。要获取快照时账户的余额，请调用[balanceOfAt](#balanceofataddress-account-uint256-snapshotid-→-uint256)函数并提供快照id和账户地址。

### gas成本
快照是高效的。创建快照的操作复杂度为O(1)。从快照中检索余额或总供应量的操作复杂度为O(log n)，其中n为已创建的快照数量，尽管对于特定账户来说，n通常会小得多，因为后续快照中的相同余额将被存储为单个条目。

由于额外的快照记录，普通的ERC20转账存在固定的开销。这种开销只对紧随特定账户快照之后的第一次转账显著。随后的转账将具有正常的成本，直到下一个快照，以此类推。

**FUNCTIONS**
[_snapshot()]

[balanceOfAt(account, snapshotId)]

[totalSupplyAt(snapshotId)]

[_beforeTokenTransfer(from, to, amount)]

ERC20
[constructor(name_, symbol_)](#constructorstring-name_-string-symbol_)

[name()](#name-→-string)

[symbol()](#symbol-→-string)

[decimals()](#decimals-→-uint8)

[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](#_setupdecimalsuint8-decimals)

**EVENTS**
[Snapshot(id)](#snapshotuint256-id)

IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### _snapshot() → uint256
internal#
创建一个新的快照并返回 Snapshot的 Snapshotid。

发出一个包含相同id的 [Snapshot](#snapshotuint256-id)事件。

[_snapshot](#_snapshot-→-uint256)是内部的，你必须决定如何将其外部公开。它的使用可能仅限于一组账户，例如使用 [AccessControl](./Access.md#accesscontrol)，或者可能对公众开放。

> WARNING
尽管某些信任最小化机制（如分叉）需要以开放的方式调用[_snapshot](#_snapshot-→-uint256)，但你必须考虑到它可能被攻击者以两种方式使用。
首先，它可以用于增加从快照中检索值的成本，尽管它将以对数方式增长，因此在长期内这种攻击将失效。其次，它可以用于针对特定账户，并增加ERC20转账的成本，如上文的gas成本部分所述。
我们还没有测量实际数字；如果你对此感兴趣，请与我们联系。

#### balanceOfAt(address account, uint256 snapshotId) → uint256
public#
检索在创建快照ID时的账户余额。

#### totalSupplyAt(uint256 snapshotId) → uint256
public#
检索在创建快照ID时的总供应量。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#

#### Snapshot(uint256 id)
事件#
当通过id标识的快照被创建时，[_snapshot](#_snapshot-→-uint256)发出的。

### ERC20Pausable
具有可暂停的代币转账、铸造和销毁功能的ERC20代币。

适用于防止交易直到评估期结束的情况，或在发生重大错误时紧急切换以冻结所有代币转账的情况。

**FUNCTIONS**
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount-2)

PAUSABLE
[constructor()](./Utils.md#constructor)

[paused()](./Utils.md#paused-→-bool)

[_pause()](./Utils.md#_pause)

[_unpause()](./Utils.md#_unpause)

ERC20
[name()](#name-→-string)

[symbol()](#symbol-→-string)

[decimals()](#decimals-→-uint8)

[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](#_setupdecimalsuint8-decimals)

**EVENTS**
PAUSABLE
[Paused(account)](./Utils.md#pausedaddress-account)

[Unpaused(account)](./Utils.md#unpausedaddress-account)

IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#
请参考[ERC20._beforeTokenTransfer](#_beforetokentransferaddress-from-address-to-uint256-amount).

要求：
* 合约不能处于暂停状态。

### ERC20Burnable
[ERC20](#erc20)的扩展允许代币持有者销毁自己拥有的代币和其授权的代币，以一种可以通过链下事件分析来识别的方式。

**FUNCTIONS**
[burn(amount)](#burnuint256-amount)

[burnFrom(account, amount)](#burnfromaddress-account-uint256-amount)

ERC20
[constructor(name_, symbol_)](#constructorstring-name_-string-symbol_)

[name()](#name-→-string)

[symbol()](#symbol-→-string)

[decimals()](#decimals-→-uint8)

[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](#_setupdecimalsuint8-decimals)

[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)

**EVENTS**
IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### burn(uint256 amount)
public#
从调用者销毁指定数量的代币。

参见 [ERC20._burn](#_burnaddress-account-uint256-amount)。

#### burnFrom(address account, uint256 amount)
public#
从账户中销毁指定数量的代币，从调用者的津贴中扣除。

参见[ERC20._burn](#_burnaddress-account-uint256-amount)和[ERC20.allowance](#allowanceaddress-owner-address-spender-e28692-uint256-1)。

要求：
* 调用者必须至少拥有账户的代币津贴中的指定数量的津贴。

### ERC20Capped
将[ERC20](#erc20)扩展为在代币供应中添加上限的功能。

**FUNCTIONS**
[constructor(cap_)](#constructoruint256-cap_)

[cap()](#cap-→-uint256)

[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount-3)

ERC20
[name()](#name-→-string)

[symbol()](#symbol-→-string)

[decimals()](#decimals-→-uint8)

[totalSupply()](#totalsupply-e28692-uint256-1)

[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[_burn(account, amount)](#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](#_setupdecimalsuint8-decimals)

**EVENTS**
IERC20
[Transfer(from, to, value)](#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](#approvaladdress-owner-address-spender-uint256-value)

#### constructor(uint256 cap_)
internal#
设置上限值。该值是不可变的，只能在构造过程中设置一次。

#### cap() → uint256
public#
返回代币总供应量的上限。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#
请参阅[ERC20._beforeTokenTransfer](#_beforetokentransferaddress-from-address-to-uint256-amount)。

需求：
* 铸造的代币不能使总供应量超过上限。

## Utilities

### SafeERC20
对ERC20操作的包装，当代币合约返回false时会抛出异常。同时支持返回无值（而是在失败时回滚或抛出异常）的代币，假设非回滚调用都是成功的。要使用这个库，你可以在你的合约中添加一个使用SafeERC20 for IERC20;语句，这样你就可以调用安全操作，如token.safeTransfer(…​)等。

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
已弃用。此函数存在与[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)中发现的类似问题，不建议使用。

尽可能使用[safeIncreaseAllowance](#safeincreaseallowancecontract-ierc20-token-address-spender-uint256-value)和[safeDecreaseAllowance](#safedecreaseallowancecontract-ierc20-token-address-spender-uint256-value)。

#### safeIncreaseAllowance(contract IERC20 token, address spender, uint256 value)
internal#

#### safeDecreaseAllowance(contract IERC20 token, address spender, uint256 value)
internal#


### TokenTimelock
一个代币持有者合约，允许受益人在给定的释放时间之后提取代币。

适用于简单的锁定期安排，比如“顾问在1年后获得全部代币”。

**FUNCTIONS**
[constructor(token_, beneficiary_, releaseTime_)](#constructorcontract-ierc20-token_-address-beneficiary_-uint256-releasetime_)

[token()](#token-→-contract-ierc20)

[beneficiary()](#beneficiary-→-address)

[releaseTime()](#releasetime-→-uint256)

[release()](#release)

#### constructor(contract IERC20 token_, address beneficiary_, uint256 releaseTime_)
public#

#### token() → contract IERC20
public#

#### beneficiary() → address
public#

#### releaseTime() → uint256
public#

#### release()
public#
