# ERC 20
这组接口、合约和实用工具都与[ERC20代币标准](https://eips.ethereum.org/EIPS/eip-20)相关。

> TIP
要了解ERC20代币的概述和如何创建代币合约的步骤，请阅读我们的[ERC20指南](../Tokens/ERC20/ERC20.md。

有一些核心合约实现了EIP中指定的行为：

* [IERC20](#ierc20)：所有ERC20实现都应符合的接口。
* [IERC20Metadata](#ierc20metadata)：扩展的ERC20接口，包括[名称](#name-→-string)、[符号](#symbol-→-string)和[小数](#decimals-→-uint8)函数。
* [ERC20](#erc20)：实现ERC20接口，包括[名称](#name-e28692-string-1)、[符号](#symbol-e28692-string-1)和[小数](#decimals-e28692-uint8-1)可选标准扩展到基本接口。

此外，还有多个自定义扩展，包括：

* [ERC20Burnable](#erc20burnable)：销毁自己的代币。
* [ERC20Capped](#erc20capped)：在铸造代币时强制执行总供应量上限。
* [ERC20Pausable](#erc20pausable)：暂停代币转移的能力。
* [ERC20Snapshot](#erc20snapshot)：高效存储过去的代币余额，以便随时查询。
* [ERC20Permit](#erc20permit)：代币的无需gas的批准（标准化为ERC2612）。
* [ERC20FlashMint](#erc20flashmint)：代币级别支持闪电贷款，通过铸造和销毁短暂的代币（标准化为ERC3156）。
* [ERC20Votes](#erc20votes)：支持投票和投票委托。
* [ERC20VotesComp](#erc20votescomp)：支持投票和投票委托（与Compound的代币兼容，具有uint96限制）。
* [ERC20Wrapper](#erc20wrapper)：创建一个由另一个ERC20支持的ERC20包装器，具有存款和取款方法。与[ERC20Votes](#erc20votes)一起使用非常有用。
* [ERC4626](#erc4626)：代币化的保险库，管理由资产（另一个ERC20）支持的股份（表示为ERC20）。

最后，有一些与ERC20合约以各种方式交互的实用程序。

* [SafeERC20](#safeerc20)：一个围绕接口的包装器，消除了处理布尔返回值的需要。
* [TokenTimelock](#tokentimelock)：将代币保留给受益人，直到指定时间。

> NOTE这组核心合约旨在不具有偏见，允许开发人员访问ERC20中的内部函数（如[_mint](#_mintaddress-account-uint256-amount)）并以他们喜欢的方式将它们公开为外部函数。另一方面，[ERC20预设](../Tokens/ERC20/ERC20.md#预设的erc20合约)（如[ERC20PresetMinterPauser](#erc20presetminterpauser)）使用偏见模式设计，为开发人员提供可用于部署的合约。

## 核心

### IERC20
```
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

ERC20标准的接口定义，如EIP中所述。

**FUNCTIONS**
[totalSupply()](#totalsupply-→-uint256)
[balanceOf(account)](#balanceofaddress-account-→-uint256)
[transfer(to, amount)](#transferaddress-to-uint256-amount-→-bool)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-→-uint256)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-→-bool)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-→-bool)

**EVENTS**
[Transfer(from, to, value)](#transferaddress-indexed-from-address-indexed-to-uint256-value)
[Approval(owner, spender, value)](#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)

#### totalSupply() → uint256
外部#
返回存在的代币数量。

#### balanceOf(address account) → uint256
外部#
返回账户拥有的代币数量。

#### transfer(address to, uint256 amount) → bool
外部#
将数量代币从调用者的账户转移到to账户。
返回一个布尔值，指示操作是否成功。
发出一个[transfer](#transferaddress-to-uint256-amount-→-bool)事件。

#### allowance(address owner, address spender) → uint256
外部#
返回spender将被允许通过[transferFrom](#transferfromaddress-from-address-to-uint256-amount-→-bool)代表owner花费的剩余代币数量。默认情况下为零。
当调用[approve](#approveaddress-spender-uint256-amount-→-bool)或[transferFrom](#transferfromaddress-from-address-to-uint256-amount-→-bool)时，此值会发生变化。

#### approve(address spender, uint256 amount) → bool
外部#
将金额设置为调用者代币的支出。
返回一个布尔值，指示操作是否成功。
> IMPORTANT
请注意，使用此方法更改津贴存在的风险，因为某些人可能会通过不幸的交易顺序同时使用旧的和新的津贴。解决这种竞争条件的一种可能的方法是先将支出者的津贴减少到0，然后再设置所需的值：https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729

发出一个[Approval](#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)事件。

#### transferFrom(address from, address to, uint256 amount) → bool
外部#
使用津贴机制，将一定数量的代币从一个账户移动到另一个账户。移动后，调用者的津贴会相应减少。
返回一个布尔值，表示操作是否成功。
触发一个[transfer](#transferaddress-to-uint256-amount-→-bool)事件。

#### Transfer(address indexed from, address indexed to, uint256 value)
事件#
当价值代币从一个帐户（从）转移到另一个帐户（到）时发出。
请注意，价值可能为零。

#### Approval(address indexed owner, address indexed spender, uint256 value)
事件#
当调用[approve](#approveaddress-spender-uint256-amount-→-bool)设置所有者的支出者的津贴时，发出津贴设置事件。 value是新的津贴。

### IERC20Metadata
```
import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
```

ERC20标准的可选元数据功能的接口。
*自v4.1版本起可用。*

**FUNCTIONS**
[name()](#name-→-string)
[symbol()](#symbol-→-string)
[decimals()](#decimals-→-uint8)

IERC20
[totalSupply()](#totalsupply-→-uint256)
[balanceOf(account)](#balanceofaddress-account-→-uint256)
[transfer(to, amount)](#transferaddress-to-uint256-amount-→-bool)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-→-uint256)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-→-bool)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-→-bool)

**EVENTS**

IERC20
[Transfer(from, to, value)](#transferaddress-indexed-from-address-indexed-to-uint256-value)
[Approval(owner, spender, value)](#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)

#### name() → string
外部#
返回代币的名称。

#### symbol() → string
外部#
返回代币的符号。

#### decimals() → uint8 
外部#
返回代币的小数位数。

### ERC20
```
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

实现[IERC20](#ierc20)接口。

该实现与代币创建方式无关。这意味着必须在派生合约中使用[_mint](#_mintaddress-account-uint256-amount)添加供应机制。对于通用机制，请参阅*ERC20PresetMinterPauser*。

> TIP
有关详细信息，请参阅我们的指南 [如何实现供应机制](https://forum.openzeppelin.com/t/how-to-implement-erc20-supply-mechanisms/226)。

[小数](#decimals-e28692-uint8-1)的默认值为18。要更改此值，你应该重写此函数以返回不同的值。

我们遵循通用的OpenZeppelin Contracts指南：函数在失败时返回而不是返回false。尽管这种行为是传统的，但并不与ERC20应用程序的期望冲突。

此外，在调用[transferFrom](#approveaddress-spender-uint256-amount-→-bool)时会发出[Approval](#approveaddress-spender-uint256-amount-→-bool)事件。这允许应用程序通过监听这些事件来重构所有帐户的津贴。其他实现EIP的可能不会发出这些事件，因为规范并不要求。

最后，非标准的[decreaseAllowance](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)和[increaseAllowance](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)函数已添加以减轻设置津贴时存在的众所周知的问题。请参阅[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)。

**FUNCTIONS**
[constructor(name_, symbol_)](#constructorstring-name_-string-symbol_)
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

**EVENTS**

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### constructor(string name_, string symbol_)
public#
设置[名称](#name-e28692-string-1)和[符号](#symbol-e28692-string-1)的值。
这两个值都是不可变的：它们只能在构建期间设置一次。

#### name() → string
public#
返回代币的名称。

#### symbol() → string
public#
返回代币的符号，通常是名称的缩写。

#### decimals() → uint8
public#
返回用于用户表示的小数位数。例如，如果小数位数为2，则505个代币的余额应该显示为5.05（505 / 10 ** 2）。
代币通常选择18的值，模仿以太和Wei之间的关系。除非被重写，否则该函数返回默认值18。

> NOTE
这些信息仅用于显示目的，它不会影响合约的任何算术，包括[IERC20.balanceOf](#balanceofaddress-account-e28692-uint256)和[IERC20.transfer](#transferaddress-to-uint256-amount-→-bool)。

#### totalSupply() → uint256
public#
请查看[IERC20.totalSupply](#totalsupply-→-uint256)。

#### balanceOf(address account) → uint256
public#
请查看[IERC20.balanceOf](#balanceofaddress-account-→-uint256).

#### transfer(address to, uint256 amount) → bool
public#
请参阅[IERC20.transfer](#transferaddress-to-uint256-amount-→-bool)。
要求：
* to地址不能为零地址。
* 调用者必须至少拥有amount数量的余额。

#### allowance(address owner, address spender) → uint256
public#
请查看 [IERC20.allowance](#allowanceaddress-owner-address-spender-→-uint256).

#### approve(address spender, uint256 amount) → bool
public#
请查看[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)。

> NOTE
如果金额为最大的uint256，则在transferFrom上不会更新授权。这在语义上等同于无限授权。

要求：
* spender不能是零地址。

#### transferFrom(address from, address to, uint256 amount) → bool
public#
查看[IERC20.transferFrom](#transferfromaddress-from-address-to-uint256-amount-→-bool)。
触发一个[Approval](#approveaddress-spender-uint256-amount-→-bool)事件，表示更新的授权额度。这不是EIP所必需的。请参见[ERC20](#erc20)开头的注释。

> NOTE
如果当前授权额度是最大的uint256，则不会更新授权额度。

要求：
* from和to不能是零地址。
* from必须至少拥有amount的余额。
* 调用者必须对from的代币拥有至少amount的授权额度。

#### increaseAllowance(address spender, uint256 addedValue) → bool
public#
原子地增加调用者授予给spender的授权金额。
这是[approve](#approveaddress-spender-uint256-amount-e28692-bool-1)的一种替代方案，可用作解决[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)中描述的问题的方法。
发出一个[Approval](#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)事件，指示更新的授权金额。
要求：
* spender不能为零地址。

#### decreaseAllowance(address spender, uint256 subtractedValue) → bool
以原子方式减少调用者授予spender的津贴。

这是可以用作对[IERC20.approve](#approveaddress-spender-uint256-amount-e28692-bool)中描述的问题的缓解措施的[approve](#approveaddress-spender-uint256-amount-→-bool的替代方法。

发出一个[Approval](#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)事件，指示更新的津贴。

要求：
* spender不能是零地址。
* spender必须至少具有减去值的调用者的津贴。

#### _transfer(address from, address to, uint256 amount)
internal#
将一定数量的代币从发送者账户转移到接收者账户。
这个内部函数等同于[transfer](#transferaddress-to-uint256-amount-e28692-bool-1)函数，可以用于实现自动代币费用、惩罚机制等。
触发一次[transfer](#transferaddress-to-uint256-amount-e28692-bool-1)事件。
要求：
* 发送者账户不能为0地址。
* 接收者账户不能为0地址。
* 发送者账户余额必须大于等于转移数量。

#### _mint(address account, uint256 amount)
internal#
创建一定数量的代币并将其分配给账户，增加总供应量。
发出一个[transfer](#transferaddress-to-uint256-amount-e28692-bool-1)事件，其中from地址设置为零地址。

要求：
* 账户不能是零地址。

#### _burn(address account, uint256 amount)
internal#
销毁账户中的代币，减少总供应量。
发出一个[transfer](#transferaddress-to-uint256-amount-e28692-bool-1)事件，把“to”设置为零地址。

要求：
* 账户不能是零地址。
* 账户必须至少拥有amount个代币。

#### _approve(address owner, address spender, uint256 amount)
internal#
将spender的数量设置为所有者代币的津贴。
该内部函数相当于approve，并可用于为某些子系统设置自动津贴等。
发出[Approval](#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)事件。

要求：
* 所有者不能为零地址。
* spender不能为零地址。

#### _spendAllowance(address owner, address spender, uint256 amount)
internal#
根据已花费的金额更新支出者的拥有津贴。
如果拥有无限制的津贴，则不更新津贴金额。如果津贴金额不足，则撤销操作。
可能会发出一个[Approval](#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)事件。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#
在任何代币转移之前调用的挂钩。这包括铸造和销毁。
调用条件：
* 当 from 和 to 都不为零时，from 的代币数量将转移到 to。
* 当 from 为零时，将为 to 铸造 amount 个代币。
* 当 to 为零时，将销毁 from 的代币数量。
* from 和 to 永远不会同时为零。
要了解更多关于挂钩的信息，请前往 [Using Hooks](../Extending-Contracts.md#使用-hooks).

#### _afterTokenTransfer(address from, address to, uint256 amount)
internal#
在任何代币转移之后调用的 hooks 。这包括铸造和销毁。
调用条件：
* 当 from 和 to 都不为零时，将 from 的代币数量转移到 to。
* 当 from 为零时，为 to 铸造了 amount 个代币。
* 当 to 为零时，从 from 销毁了 amount 个代币。
* from 和 to 永远不会同时为零。
要了解更多关于 hooks 的信息，请前往[Using Hooks](../Extending-Contracts.md#使用-hooks)。

## 扩展程序

### ERC20Burnable
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
```

[ERC20](#erc20)的扩展，允许代币持有者销毁他们自己的代币和他们允许的代币，以一种可以通过事件分析在链外识别的方式。

**FUNCTIONS**
[burn(amount)](#burnuint256-amount)
[burnFrom(account, amount)](#burnfromaddress-account-uint256-amount)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1))
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

**EVENTS**

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### burn(uint256 amount)
public#
销毁调用者的代币数量。
参见 [ERC20._burn](#_burnaddress-account-uint256-amount-1)。

#### burnFrom(address account, uint256 amount)
public#
销毁账户中指定数量的代币，从调用者的授权中扣除。
请参阅[ERC20._burn](#_burnaddress-account-uint256-amount-1)和[ERC20.allowance](#allowanceaddress-owner-address-spender-e28692-uint256-1)。

要求：
* 调用者必须具有至少等于销毁数量的账户代币授权。

### ERC20Capped
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Capped.sol";
```

将[ERC20](#erc20)扩展，以添加代币供应量上限。

**FUNCTIONS**
[constructor(cap_)](#constructoruint256-cap_)
[cap()](#cap-→-uint256)
[_mint(account, amount)](#_mintaddress-account-uint256-amount-1)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

**EVENTS**

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### constructor(uint256 cap_)
internal#
设置帽子的价值。这个值是不可变的，只能在构造过程中设置一次。

#### cap() → uint256
public#
返回代币总供应量的上限。

#### _mint(address account, uint256 amount)
internal#
请参阅[ERC20._mint](#_mintaddress-account-uint256-amount)。

### ERC20Pausable
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Pausable.sol";
```

具有可暂停代币转移、铸造和销毁的ERC20代币。
适用于防止交易直到评估期结束或在出现大规模漏洞时有一个紧急开关以冻结所有代币转移的情况等场景。

> IMPORTANT
此合约不包括公共暂停和取消暂停函数。除了继承此合约外，你还必须定义这两个函数，并使用适当的访问控制（例如使用[AccessControl](./Access.md#accesscontrol)或[Ownable](./Access.md#ownable)）调用[Pausable._pause](./Security.md#_pause)和[Pausable._unpause](./Security.md#_unpause)内部函数。否则，合约将无法暂停。

**FUNCTIONS**
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount-1)

PAUSABLE
[paused()](./Security.md#paused-→-bool)
[_requireNotPaused()](./Security.md#_requirenotpaused)
[_requirePaused()](./Security.md#_requirepaused)
[_pause()](./Security.md#_pause)
[_unpause()](./Security.md#_unpause)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1))
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

**EVENTS**

PAUSABLE
[Paused(account)](./Security.md#pausedaddress-account)
[Unpaused(account)](./Security.md#unpausedaddress-account)

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#
请参阅 [ERC20._beforeTokenTransfer](#_beforetokentransferaddress-from-address-to-uint256-amount).
要求：
* 合约不能暂停。

### ERC20Permit
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";
```

实现了ERC20 Permit扩展，允许通过签名进行批准，该扩展在[EIP-2612](https://eips.ethereum.org/EIPS/eip-2612)中定义。
添加了[permit](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)方法，可以通过提供由账户签名的消息来改变账户的ERC20允许额度（参见[IERC20.allowance](#allowanceaddress-owner-address-spender-→-uint256)）。通过不依赖[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)，代币持有人账户不需要发送交易，因此根本不需要持有以太。
*自v3.4版本以来可用。*

**FUNCTIONS**
[constructor(name)](#constructorstring-name)
[permit(owner, spender, value, deadline, v, r, s)](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)
[nonces(owner)](#noncesaddress-owner-→-uint256)
[DOMAIN_SEPARATOR()](#domain_separator-→-bytes32)
[_useNonce(owner)](#_usenonceaddress-owner-→-uint256-current)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1))
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

**EVENTS**

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### constructor(string name)
internal#
使用name参数初始化[EIP712](./Utils.md#eip712)域分隔符，并将version设置为“1”。最好使用与ERC20代币名称相同的名称。

#### permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
public#
请参阅 [IERC20Permit.permit](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s).

#### nonces(address owner) → uint256
public#
请参阅 [IERC20Permit.nonces](#noncesaddress-owner-→-uint256).

#### DOMAIN_SEPARATOR() → bytes32
外部#
请参阅 [IERC20Permit.DOMAIN_SEPARATOR](#domain_separator-→-bytes32).

#### _useNonce(address owner) → uint256 current
internal#
"消耗一个随机数": 返回当前值并递增。 
*自v4.1版本起可用。*

### ERC20Snapshot
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Snapshot.sol";
```

这份合约扩展了一个带有快照机制的ERC20代币。当创建快照时，记录当前时间的余额和总供应量以便以后访问。

这可以用于安全地创建基于代币余额的机制，例如无信任股息或加权投票。在天真的实现中，可以通过从不同的账户重复使用同一余额来执行“双重花费”攻击。通过使用快照来计算股息或投票权，这些攻击不再适用。它也可以用于创建高效的ERC20分叉机制。

快照是通过内部的 [_snapshot](#_snapshot-→-uint256) 函数创建的，该函数将触发[Snapshot](#snapshotuint256-id)事件并返回快照ID。要获取快照时的总供应量，请使用快照ID调用 [totalSupplyAt](#totalsupplyatuint256-snapshotid-→-uint256) 函数。要获取快照时某个账户的余额，请使用快照ID和账户地址调用 [balanceOfAt](#balanceofataddress-account-uint256-snapshotid-→-uint256) 函数。

> NOTE
可以通过重写 [_getCurrentSnapshotId](#_getcurrentsnapshotid-→-uint256) 方法来自定义快照策略。例如，让它返回 block.number 将在每个新块的开头触发快照的创建。在重写此函数时，要小心其结果的单调性。非单调的快照ID会破坏合约。

使用此方法为每个块实现快照将产生显著的gas成本。对于gas效率更高的替代方案，请考虑使用 [ERC20Votes](#erc20votes)。

#### gas成本
快照是高效的。快照创建是 O(1)。从快照中检索余额或总供应量的成本是 O(log n)，其中 n 是已创建的快照数量，尽管对于特定账户的 n 通常会更小，因为连续快照中相同的余额将存储为一个条目。

由于额外的快照记账，普通 ERC20 转账存在常数开销。对于特定账户的第一次转账，这种开销是显著的。随后的转账将具有正常的成本，直到下一次快照，以此类推。

**FUNCTIONS**
[_snapshot()](#_snapshot-→-uint256)
[_getCurrentSnapshotId()](#_getcurrentsnapshotid-→-uint256)
[balanceOfAt(account, snapshotId)](#balanceofataddress-account-uint256-snapshotid-→-uint256)
[totalSupplyAt(snapshotId)](#totalsupplyatuint256-snapshotid-→-uint256)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount-2)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1))
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

#### _snapshot() → uint256
internal#
创建一个新的快照并返回其快照ID。
触发一个包含相同ID的[Snapshot](#snapshotuint256-id) 事件。
[_snapshot](#_snapshot-→-uint256)是内部的，你必须决定如何将其外部公开。它的使用可能会受到一组帐户的限制，例如使用[AccessControl](./Access.md#accesscontrol)，或者它可能向公众开放。

> WARNING
虽然需要一种开放的调用[_snapshot](#_snapshot-→-uint256)的方式来实现某些信任最小化机制，例如分叉，但你必须考虑它可能被攻击者以两种方式使用。

首先，它可以用于增加从快照检索值的成本，尽管它将呈对数增长，因此从长远来看，这种攻击将变得无效。其次，它可以用于针对特定帐户，并增加ERC20转移对它们的成本，以Gas Costs部分中指定的方式。

我们没有测量实际数字；如果你对此有兴趣，请与我们联系。

#### _getCurrentSnapshotId() → uint256
internal#
获取当前snapshotId

#### balanceOfAt(address account, uint256 snapshotId) → uint256
public#
检索创建snapshotId时账户的余额。

#### totalSupplyAt(uint256 snapshotId) → uint256
public#
检索创建snapshotId时的总供应量。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#
在任何代币转移之前调用的 hooks 。这包括铸造和销毁。

调用条件：
* 当 from 和 to 都不为零时，from 的代币数量将转移到 to。
* 当 from 为零时，将为 to 铸造 amount 个代币。
* 当 to 为零时，将销毁 from 的代币数量。
* from 和 to 永远不会同时为零。

要了解更多有关 hooks 的信息，请前往 [Using Hooks](../Extending-Contracts.md#使用-hooks) 。

#### Snapshot(uint256 id)
事件#
当由id标识的快照创建时，由[_snapshot](#_snapshot-→-uint256)发出。

#### ERC20Votes
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
```

ERC20的扩展，支持类似Compound的投票和委托。这个版本比Compound的更通用，支持代币供应量高达2^224-1, 而COMP仅限于2^96-1。

> NOTE
如果需要完全兼容COMP，则使用此模块的[ERC20VotesComp](#erc20votescomp)变体。

此扩展会保留每个帐户的投票权历史记录（检查点）。投票权可以通过直接调用*委托*函数或提供签名以与[delegateBySig](#delegatebysigaddress-delegatee-uint256-nonce-uint256-expiry-uint8-v-bytes32-r-bytes32-s)一起使用来委托。可以通过公共访问器[getVotes](#getvotesaddress-account-→-uint256)和[getPastVotes](#getpastvotesaddress-account-uint256-timepoint-→-uint256)查询投票权。

默认情况下，代币余额不考虑投票权。这使转账更便宜。缺点是需要用户委托给自己以激活检查点并跟踪其投票权。

*自v4.2以来可用。*

**FUNCTIONS**
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[checkpoints(account, pos)](#checkpointsaddress-account-uint32-pos-→-struct-erc20votescheckpoint)
[numCheckpoints(account)](#numcheckpointsaddress-account-→-uint32)
[delegates(account)](#delegatesaddress-account-→-address)
[getVotes(account)](#getvotesaddress-account-→-uint256)
[getPastVotes(account, timepoint)](#getvotesaddress-account-→-uint256)
[getPastTotalSupply(timepoint)](#getpasttotalsupplyuint256-timepoint-→-uint256)
[delegate(delegatee)](#delegateaddress-delegatee)
[delegateBySig(delegatee, nonce, expiry, v, r, s)](#delegatebysigaddress-delegatee-uint256-nonce-uint256-expiry-uint8-v-bytes32-r-bytes32-s)
[_maxSupply()](#_maxsupply-→-uint224)
[_mint(account, amount)](#_mintaddress-account-uint256-amount-2)
[_burn(account, amount)](#_burnaddress-account-uint256-amount-1)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount-1)
[_delegate(delegator, delegatee)](#_delegateaddress-delegator-address-delegatee)

ERC20PERMIT
[permit(owner, spender, value, deadline, v, r, s)](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)
[nonces(owner)](#noncesaddress-owner-→-uint256)
[DOMAIN_SEPARATOR()](#domain_separator-→-bytes32)
[_useNonce(owner)](#_usenonceaddress-owner-→-uint256-current)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1))
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)

**EVENTS**
IVOTES
[DelegateChanged(delegator, fromDelegate, toDelegate)](./Governance.md)
[DelegateVotesChanged(delegate, previousBalance, newBalance)](./Governance.md)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### clock() → uint48
public#
用于标记检查点的时钟。可以被重写以实现基于时间戳的检查点（和投票）。

#### CLOCK_MODE() → string
public#
时钟的描述

#### checkpoints(address account, uint32 pos) → struct ERC20Votes.Checkpoint
public#
获取账户的第pos个检查点。

#### numCheckpoints(address account) → uint32
public#
获取账户的检查点数量。

#### delegates(address account) → address
public#
获取当前账户正在委托给的地址。

#### getVotes(address account) → uint256
public#
获取账户当前的投票余额

#### getPastVotes(address account, uint256 timepoint) → uint256
public#
获取账户在时间点结束时的投票数。

要求：
* 时间点必须在过去。

#### getPastTotalSupply(uint256 timepoint) → uint256
public#
在时间点结束时检索totalSupply。注意，这个值是所有余额的总和。它不是所有委托投票的总和！

要求：
* 时间点必须在过去。

#### delegate(address delegatee)
public#
将选票从发件人委派给受委托人。

#### delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
public#
将签署者的投票委托给代表

#### _maxSupply() → uint224
internal#
最大代币供应量。默认为type(uint224).max（2^224-1）。

#### _mint(address account, uint256 amount)
internal#
在总供应量增加之后totalSupply。

#### _burn(address account, uint256 amount)
internal#
在减少totalSupply之后Snapshots。

#### _afterTokenTransfer(address from, address to, uint256 amount)
internal#
当代币转移时移动投票权。
发出一个[IVotes.DelegateVotesChanged](./Governance.md)事件。

#### _delegate(address delegator, address delegatee)
internal#
将委托从委托者更改为委托受托人。

发出事件 [IVotes.DelegateChanged](./Governance.md) 和 [IVotes.DelegateVotesChanged](./Governance.md)。

### ERC20VotesComp
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20VotesComp.sol";
```

ERC20的扩展，支持Compound的投票和委托。该版本完全符合Compound的接口，但缺点是只支持供应量高达（2^96-1）。

> NOTE
如果你需要与COMP完全兼容（例如为了在Governor Alpha或Bravo中使用你的代币），并且你确定2^96的供应上限足够你使用，请使用此合约。否则，请使用此模块的[ERC20Votes](#erc20votes)变体。

此扩展保留每个帐户投票权的历史记录（检查点）。投票权可以通过直接调用 [delegate](#delegateaddress-delegatee)函数或提供用于[delegateBySig](#delegatebysigaddress-delegatee-uint256-nonce-uint256-expiry-uint8-v-bytes32-r-bytes32-s)的签名来委托。可以通过公共访问器[getCurrentVotes](#getcurrentvotesaddress-account-→-uint96)和[getPriorVotes](#getpriorvotesaddress-account-uint256-blocknumber-→-uint96)查询投票权。

默认情况下，代币余额不考虑投票权。这使得转账更便宜。缺点是需要用户委托给自己，以激活检查点并跟踪其投票权。

*自v4.2以来可用。*

**FUNCTIONS**
[getCurrentVotes(account)](#getcurrentvotesaddress-account-→-uint96)
[getPriorVotes(account, blockNumber)](#getpriorvotesaddress-account-uint256-blocknumber-→-uint96)
[_maxSupply()](#_maxsupply-e28692-uint224-1)

ERC20VOTES
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[checkpoints(account, pos)](#checkpointsaddress-account-uint32-pos-→-struct-erc20votescheckpoint)
[numCheckpoints(account)](#numcheckpointsaddress-account-→-uint32)
[delegates(account)](#delegatesaddress-account-→-address)
[getVotes(account)](#getvotesaddress-account-→-uint256)
[getPastVotes(account, timepoint)](#getvotesaddress-account-→-uint256)
[getPastTotalSupply(timepoint)](#getpasttotalsupplyuint256-timepoint-→-uint256)
[delegate(delegatee)](#delegateaddress-delegatee)
[delegateBySig(delegatee, nonce, expiry, v, r, s)](#delegatebysigaddress-delegatee-uint256-nonce-uint256-expiry-uint8-v-bytes32-r-bytes32-s)
[_mint(account, amount)](#_mintaddress-account-uint256-amount-2)
[_burn(account, amount)](#_burnaddress-account-uint256-amount-1)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount-1)
[_delegate(delegator, delegatee)](#_delegateaddress-delegator-address-delegatee)

ERC20PERMIT
[permit(owner, spender, value, deadline, v, r, s)](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)
[nonces(owner)](#noncesaddress-owner-→-uint256)
[DOMAIN_SEPARATOR()](#domain_separator-→-bytes32)
[_useNonce(owner)](#_usenonceaddress-owner-→-uint256-current)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)

**EVENTS**

IVOTES
[DelegateChanged(delegator, fromDelegate, toDelegate)](./Governance.md)
[DelegateVotesChanged(delegate, previousBalance, newBalance)](./Governance.md)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### getCurrentVotes(address account) → uint96
外部#
[getVotes](#getvotesaddress-account-→-uint256)信息的Comp版本访问器，返回类型为uint96。

#### getPriorVotes(address account, uint256 blockNumber) → uint96
外部#
[getPastVotes](#getpastvotesaddress-account-uint256-timepoint-→-uint256)记录的访问器的Comp版本，返回类型为uint96。

#### _maxSupply() → uint224
internal#
最大代币供应量。缩小至type(uint96).max（2^96-1）以适应COMP接口。

### ERC20Wrapper
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Wrapper.sol";
```

ERC20代币合约的扩展，以支持代币包装。

用户可以存入和取出“基础代币”，并获得相应数量的“包装代币”。这在与其他模块结合使用时非常有用。例如，将此包装机制与[ERC20Votes](#erc20votes)结合使用，可以将现有的“基本”ERC20包装成治理代币。

*自v4.2以来可用。*

**FUNCTIONS**
[constructor(underlyingToken)](#constructorcontract-ierc20-underlyingtoken)
[decimals()](#decimals-e28692-uint8-2)
[underlying()](#underlying-→-contract-ierc20)
[depositFor(account, amount)](#depositforaddress-account-uint256-amount-→-bool)
[withdrawTo(account, amount)](#withdrawtoaddress-account-uint256-amount-→-bool)
[_recover(account)](#_recoveraddress-account-→-uint256)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

**EVENTS**
IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### constructor(contract IERC20 underlyingToken)
internal#

#### decimals() → uint8
public#
请查看 [ERC20.decimals](#decimals-→-uint8).

#### underlying() → contract IERC20
public#
返回被包装的基础ERC返回被封装的基础ERC-20代币的地址。

#### depositFor(address account, uint256 amount) → bool
public#
允许用户存入基础代币，并铸造相应数量的封装代币。

#### withdrawTo(address account, uint256 amount) → bool
public#
允许用户烧掉一定数量的封装代币，并提取相应数量的基础代币。

#### _recover(address account) → uint256
internal#
Mint包装代币以重写可能因错误而转移的任何基础代币。如果需要，此内部函数可以通过访问控制公开。

### ERC20FlashMint
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20FlashMint.sol";
```

实现 ERC3156 闪电贷扩展，如 [ERC-3156](https://eips.ethereum.org/EIPS/eip-3156) 中所定义。

添加 [flashLoan](#flashloancontract-ierc3156flashborrower-receiver-address-token-uint256-amount-bytes-data-→-bool) 方法，为代币级别提供闪电贷支持。默认情况下没有费用，但可以通过重写 [flashFee](#flashfeeaddress-token-uint256-amount-→-uint256) 进行更改。

*自 v4.1 版本起可用。*

**FUNCTIONS**
[maxFlashLoan(token)](#maxflashloanaddress-token-→-uint256)
[flashFee(token, amount)](#flashfeeaddress-token-uint256-amount-→-uint256)
[_flashFee(token, amount)](#_flashfeeaddress-token-uint256-amount-→-uint256)
[_flashFeeReceiver()](#_flashfeereceiver-→-address)
[flashLoan(receiver, token, amount, data)](#flashloancontract-ierc3156flashborrower-receiver-address-token-uint256-amount-bytes-data-→-bool)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)


**EVENTS**

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### maxFlashLoan(address token) → uint256
public#
返回可供借贷的最大代币数量。

#### flashFee(address token, uint256 amount) → uint256
public#
返回执行闪电贷款时应用的费用。该函数调用 [_flashFee](#_flashfeeaddress-token-uint256-amount-→-uint256) 函数，该函数返回执行闪电贷款时应用的费用。

#### _flashFee(address token, uint256 amount) → uint256
internal#
返回进行闪电贷款时应用的费用。默认情况下，此实现没有任何费用。可以重载此函数，使闪电贷款机制通缩。

#### _flashFeeReceiver() → address
internal#
返回闪电交易手续费的接收地址。默认情况下，此实现返回地址(0)，这意味着手续费金额将被烧掉。可以重载此函数以更改手续费接收者。

#### flashLoan(contract IERC3156FlashBorrower receiver, address token, uint256 amount, bytes data) → bool
public#
执行闪电贷款。新的代币被铸造并发送给接收方，接收方需要实现[IERC3156FlashBorrower](./Interfaces.md#ierc3156flashborrower)接口。在闪电贷款结束时，接收方应当拥有amount + fee代币并将其批准回到代币合约本身，以便它们可以被销毁。

### ERC4626
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC4626.sol";
```

实施 ERC4626“代币化保险库标准”，如 [EIP-4626](https://eips.ethereum.org/EIPS/eip-4626) 中定义的。

该扩展允许通过标准化的 [deposit](#deposituint256-assets-address-receiver-→-uint256)、 [mint](#mintuint256-shares-address-receiver-→-uint256)、[redeem](#redeemuint256-shares-address-receiver-address-owner-→-uint256)和 [burn](#burnuint256-amount)流程，以“资产”为代价铸造和销毁“股份”（使用 ERC20 继承表示）。此合约扩展了 ERC20 标准。与它一起包含的任何其他扩展都会影响由此合约表示的“股份”代币，而不是独立合约的“资产”代币。

> CAUTION
在空的（或几乎空的）ERC-4626保险库中，存款有高风险通过“捐款”到保险库中被抢走，从而使股份的价格膨胀。这通常称为“捐赠”或通货膨胀攻击，本质上是滑点问题。保险库部署者可以通过进行一笔非微不足道的资产存款来防止这种攻击，从而使价格操纵变得不可行。提款也可能受到滑点的影响。用户也可以通过验证所接收的金额与预期金额相符来保护自己免受这种攻击以及意外滑点的影响，使用执行这些检查的包装器，例如 [ERC4626Router](https://github.com/fei-protocol/ERC4626#erc4626router-and-base)。

自 v4.9 以来，该实现使用虚拟资产和股份来缓解这种风险。_decimalsOffset() 对应于基础资产的小数位数和保险库小数位数之间的小数表示之间的偏移量。此偏移量还确定了保险库中虚拟股份与虚拟资产之间的比率，这本身确定了初始汇率。虽然不能完全防止攻击，但分析表明，默认偏移量（0）使其不赚钱，因为虚拟股份捕获的价值（来自攻击者的捐赠）与攻击者的预期收益相匹配。使用较大的偏移量，攻击变得比盈利昂贵几个数量级。有关底层数学的更多详细信息，请参见[此处](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#erc4626.adoc#inflation-attack)。

这种方法的缺点是虚拟股份确实捕获了（非常小的）部分累积到保险库的价值。此外，如果保险库遭受损失，用户试图退出保险库，虚拟股份和资产将导致第一个用户退出时经历减少的损失，而最后的用户将经历更大的损失。愿意恢复到 v4.9 之前行为的开发人员只需重写 _convertToShares 和 _convertToAssets 函数即可。

要了解更多信息，请查看我们的 [ERC-4626 指南](../Tokens/ERC4626/ERC4626.md)。

*自 v4.7 以来可用。*

**FUNCTIONS**
[constructor(asset_)](#constructorcontract-ierc20-asset_)
[decimals()](#decimals-e28692-uint8-3)
[asset()](#asset-→-address)
[totalAssets()](#totalassets-→-uint256)
[convertToShares(assets)](#converttosharesuint256-assets-→-uint256)
[convertToAssets(shares)](#converttosharesuint256-assets-→-uint256)
[maxDeposit()](#maxdepositaddress-→-uint256)
[maxMint()](#maxmintaddress-→-uint256)
[maxWithdraw(owner)](#maxwithdrawaddress-owner-→-uint256)
[maxRedeem(owner)](#maxredeemaddress-owner-→-uint256)
[previewDeposit(assets)](#previewdeposituint256-assets-→-uint256)
[previewMint(shares)](#previewmintuint256-shares-→-uint256)
[previewWithdraw(assets)](#previewredeemuint256-shares-→-uint256)
[previewRedeem(shares)](#previewredeemuint256-shares-→-uint256)
[deposit(assets, receiver)](#deposituint256-assets-address-receiver-→-uint256)
[mint(shares, receiver)](#mintuint256-shares-address-receiver-→-uint256)
[withdraw(assets, receiver, owner)](#withdrawuint256-assets-address-receiver-address-owner-→-uint256)
[redeem(shares, receiver, owner)](#redeemuint256-shares-address-receiver-address-owner-→-uint256)
[_convertToShares(assets, rounding)](#_converttosharesuint256-assets-enum-mathrounding-rounding-→-uint256)
[_convertToAssets(shares, rounding)](#_converttoassetsuint256-shares-enum-mathrounding-rounding-→-uint256)
[_deposit(caller, receiver, assets, shares)](#_depositaddress-caller-address-receiver-uint256-assets-uint256-shares)
[_withdraw(caller, receiver, owner, assets, shares)](#_withdrawaddress-caller-address-receiver-address-owner-uint256-assets-uint256-shares)
[_decimalsOffset()](#_decimalsoffset-→-uint8)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)


**EVENTS**
IERC4626
[Deposit(sender, owner, assets, shares)](./Interfaces.md#depositaddress-indexed-sender-address-indexed-owner-uint256-assets-uint256-shares)
[Withdraw(sender, receiver, owner, assets, shares)](./Interfaces.md#withdrawaddress-indexed-sender-address-indexed-receiver-address-indexed-owner-uint256-assets-uint256-shares)

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### constructor(contract IERC20 asset_)
internal#
设置基础资产合约。这必须是一个ERC20兼容合约（ERC20或ERC777）。

#### decimals() → uint8
public#
十进制数是通过将十进制偏移量加到基础资产的小数位上计算出来的。在构建保险库合约期间，这个“原始”值被缓存。如果读取操作失败（例如，资产尚未创建），则默认值为18，表示基础资产的小数位数。

请参阅[IERC20Metadata.decimals](#decimals-→-uint8)。

#### asset() → address
public#
请参阅[IERC4626.asset](./Interfaces.md#totalassets-→-uint256-totalmanagedassets).

#### totalAssets() → uint256
public#
请参阅[IERC4626.totalAssets](./Interfaces.md#totalassets-→-uint256-totalmanagedassets).

#### convertToShares(uint256 assets) → uint256
public#
请参阅[IERC4626.convertToShares](./Interfaces.md#converttosharesuint256-assets-→-uint256-shares).

#### convertToAssets(uint256 shares) → uint256
public#
请参阅[IERC4626.convertToAssets](./Interfaces.md#converttoassetsuint256-shares-→-uint256-assets).

#### maxDeposit(address) → uint256
public#
请参阅 [IERC4626.maxDeposit](./Interfaces.md#maxdepositaddress-receiver-→-uint256-maxassets).

##### maxMint(address) → uint256
public#
请参阅 [IERC4626.maxMint](./Interfaces.md#maxmintaddress-receiver-→-uint256-maxshares).

#### maxWithdraw(address owner) → uint256
public#
请参阅 [IERC4626.maxWithdraw](./Interfaces.md#maxwithdrawaddress-owner-→-uint256-maxassets).

#### maxRedeem(address owner) → uint256
public#
请参阅 [IERC4626.maxRedeem](./Interfaces.md#maxredeemaddress-owner-→-uint256-maxshares).

#### previewDeposit(uint256 assets) → uint256
public#
请参阅 [IERC4626.previewDeposit](./Interfaces.md#previewdeposituint256-assets-→-uint256-shares).

#### previewMint(uint256 shares) → uint256
public#
请参阅 [IERC4626.previewMint](./Interfaces.md#previewmintuint256-shares-→-uint256-assets).

####  previewWithdraw(uint256 assets) → uint256
public#
请参阅 [IERC4626.previewWithdraw](./Interfaces.md#previewwithdrawuint256-assets-→-uint256-shares).

#### previewRedeem(uint256 shares) → uint256
public#
请参阅 [IERC4626.previewRedeem](./Interfaces.md#previewredeemuint256-shares-→-uint256-assets).

#### deposit(uint256 assets, address receiver) → uint256
public#
请参阅 [IERC4626.deposit](./Interfaces.md#deposituint256-assets-address-receiver-→-uint256-shares).

#### mint(uint256 shares, address receiver) → uint256
public#
请参阅 [IERC4626.mint](./Interfaces.md#mintuint256-shares-address-receiver-→-uint256-assets).
与[deposit](#deposituint256-assets-address-receiver-→-uint256)相反，即使保险库处于份额价格为零的状态，也允许进行铸造。在这种情况下，股份将被铸造而无需存入任何资产。

#### withdraw(uint256 assets, address receiver, address owner) → uint256
public#
请参阅 [IERC4626.withdraw](./Interfaces.md#withdrawuint256-assets-address-receiver-address-owner-→-uint256-shares).

#### redeem(uint256 shares, address receiver, address owner) → uint256
public#
请参阅 [IERC4626.redeem](./Interfaces.md#redeemuint256-shares-address-receiver-address-owner-→-uint256-assets).

#### _convertToShares(uint256 assets, enum Math.Rounding rounding) → uint256
internal#
内部转换函数（从资产到股份），支持舍入方向。

#### _convertToAssets(uint256 shares, enum Math.Rounding rounding) → uint256
internal#
支持舍入方向的内部转换函数（从股份转换为资产）。

#### _deposit(address caller, address receiver, uint256 assets, uint256 shares)
internal#
存款/铸造常见工作流程。

#### _withdraw(address caller, address receiver, address owner, uint256 assets, uint256 shares)
internal#
撤回/兑现常见的工作流程。

#### _decimalsOffset() → uint8
internal#

## 预设
这些合约是上述功能的预配置组合。它们可以通过继承或复制粘贴其源代码作为模型使用。

### ERC20PresetMinterPauser
```
import "@openzeppelin/contracts/token/ERC20/presets/ERC20PresetMinterPauser.sol";
```

[ERC20](#erc20)代币，包括：
* 持有者可以烧毁（销毁）他们的代币的能力
* 允许代币铸造（创建）的铸造者角色
* 允许停止所有代币转移的暂停者角色

该合约使用*AccessControl*锁定使用不同角色的权限函数-有关详细信息，请查阅其文档。

部署合约的账户将被授予铸造者和暂停者角色，以及默认管理员角色，这将使其授予其他账户铸造者和暂停者角色。

弃用，有利于[合约向导](https://wizard.openzeppelin.com/)。

**FUNCTIONS**
[constructor(name, symbol)](#constructorstring-name-string-symbol)
[mint(to, amount)](#mintaddress-to-uint256-amount)
[pause()](#pause)
[unpause()](#unpause)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount-3)

PAUSABLE
[paused()](./Security.md#paused-→-bool)
[_requireNotPaused()](./Security.md#_requirenotpaused)
[_requirePaused()](./Security.md#_requirepaused)
[_pause()](./Security.md#_pause)
[_unpause()](./Security.md#_unpause)

ERC20BURNABLE
[burn(amount)](#burnuint256-amount)
[burnFrom(account, amount)](#burnfromaddress-account-uint256-amount)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

ACCESSCONTROLENUMERABLE
[supportsInterface(interfaceId)](./Access.md#supportsinterfacebytes4-interfaceid-e28692-bool-1)
[getRoleMember(role, index)](./Access.md#getrolememberbytes32-role-uint256-index-e28692-address-1)
[getRoleMemberCount(role)](./Access.md#getrolemembercountbytes32-role-e28692-uint256-1)
[_grantRole(role, account)](./Access.md#_grantrolebytes32-role-address-account-1)
[_revokeRole(role, account)](./Access.md#_revokerolebytes32-role-address-account-1)

ACCESSCONTROL
[hasRole(role, account)](./Access.md#hasrolebytes32-role-address-account-e28692-bool-1)
[_checkRole(role)](./Access.md#_checkrolebytes32-role)
[_checkRole(role, account)](./Access.md#_checkrolebytes32-role-address-account)
[getRoleAdmin(role)](./Access.md#getroleadminbytes32-role-e28692-bytes32-1)
[grantRole(role, account)](./Access.md#grantrolebytes32-role-address-account-1)
[revokeRole(role, account)](./Access.md#revokerolebytes32-role-address-account-1)
[renounceRole(role, account)](./Access.md#renouncerolebytes32-role-address-account-1)
[_setupRole(role, account)](./Access.md#_setuprolebytes32-role-address-account)
[_setRoleAdmin(role, adminRole)](./Access.md#_setroleadminbytes32-role-bytes32-adminrole)

**EVENTS**
PAUSABLE
[Paused(account)](./Security.md#pausedaddress-account)
[Unpaused(account)](./Security.md#unpausedaddress-account)

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](./Access.md#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](./Access.md#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](./Access.md#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### constructor(string name, string symbol)
public#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授权给部署合约的账户。
请参阅 [ERC20.constructor](#constructorstring-name_-string-symbol_)。

#### mint(address to, uint256 amount)
public#
为to创建指定数量的新代币。
请参考 [ERC20._mint](#_mintaddress-account-uint256-amount)。
要求：
* 调用者必须具有MINTER_ROLE角色。

#### pause()
public#
暂停所有代币转移。
请参阅 [ERC20Pausable](#erc20pausable) 和 [Pausable._pause](./Security.md#_pause)。
要求：
* 调用者必须具有 PAUSER_ROLE。

#### unpause()
public#
取消所有代币转移的暂停状态。
请参阅 [ERC20Pausable](#erc20pausable)和[Pausable._unpause](./Security.md#_unpause)。
要求：
* 调用者必须具有PAUSER_ROLE。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#

### ERC20PresetFixedSupply
```
import "@openzeppelin/contracts/token/ERC20/presets/ERC20PresetFixedSupply.sol";
```

[ERC20](#erc20)代币，包括：
* 预先发行的初始供应量
* 持有人可以烧毁（销毁）他们的代币的能力
* 没有访问控制机制（用于铸造/暂停），因此没有治理

该合约使用[ERC20Burnable](#erc20burnable0包括烧毁功能-请查看其文档以获取详细信息。

*自v3.4以来可用。*

弃用，推荐使用[Contracts Wizard](https://wizard.openzeppelin.com/)。

**FUNCTIONS**
[constructor(name, symbol, initialSupply, owner)](#constructorstring-name-string-symbol-uint256-initialsupply-address-owner)

ERC20BURNABLE
[burn(amount)](#burnuint256-amount)
[burnFrom(account, amount)](#burnfromaddress-account-uint256-amount)

ERC20
[name()](#name-e28692-string-1)
[symbol()](#symbol-e28692-string-1)
[decimals()](#decimals-e28692-uint8-1)
[totalSupply()](#totalsupply-e28692-uint256-1)
[balanceOf(account)](#balanceofaddress-account-e28692-uint256-1)
[transfer(to, amount)](#transferaddress-to-uint256-amount-e28692-bool-1)
[allowance(owner, spender)](#allowanceaddress-owner-address-spender-e28692-uint256-1)
[approve(spender, amount)](#approveaddress-spender-uint256-amount-e28692-bool-1)
[transferFrom(from, to, amount)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[increaseAllowance(spender, addedValue)](#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)
[decreaseAllowance(spender, subtractedValue)](#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)
[_transfer(from, to, amount)](#_transferaddress-from-address-to-uint256-amount)
[_mint(account, amount)](#_mintaddress-account-uint256-amount)
[_burn(account, amount)](#_burnaddress-account-uint256-amount)
[_approve(owner, spender, amount)](#_approveaddress-owner-address-spender-uint256-amount)
[_spendAllowance(owner, spender, amount)](#_spendallowanceaddress-owner-address-spender-uint256-amount)
[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)
[_afterTokenTransfer(from, to, amount)](#_aftertokentransferaddress-from-address-to-uint256-amount)

**EVENTS**

IERC20
[Transfer(from, to, value)](#transferfromaddress-from-address-to-uint256-amount-e28692-bool-1)
[Approval(owner, spender, value)](#approveaddress-spender-uint256-amount-e28692-bool-1)

#### constructor(string name, string symbol, uint256 initialSupply, address owner)
public#
Mint函数会初始化一定数量的代币并将它们转移到所有者账户中。
请参阅 [ERC20.constructor](#constructorstring-name_-string-symbol_)。

## 应用程序

### SafeERC20
```
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```

SafeERC20是一种在ERC20操作失败时（当代币合约返回false时）抛出异常的包装器。它还支持返回无值（而是在失败时回滚或抛出异常）的代币，假定非回滚调用是成功的。要使用此库，你可以在合约中添加一个using SafeERC20 for IERC20;语句，这允许你调用安全操作，例如token.safeTransfer(…​)等。

**FUNCTIONS**
[safeTransfer(token, to, value)](#safetransfercontract-ierc20-token-address-to-uint256-value)
[safeTransferFrom(token, from, to, value)](#safetransferfromcontract-ierc20-token-address-from-address-to-uint256-value)
[safeApprove(token, spender, value)](#safeapprovecontract-ierc20-token-address-spender-uint256-value)
[safeIncreaseAllowance(token, spender, value)](#safeincreaseallowancecontract-ierc20-token-address-spender-uint256-value)
[safeDecreaseAllowance(token, spender, value)](#safedecreaseallowancecontract-ierc20-token-address-spender-uint256-value)
[forceApprove(token, spender, value)](#forceapprovecontract-ierc20-token-address-spender-uint256-value)
[safePermit(token, owner, spender, value, deadline, v, r, s)](#safepermitcontract-ierc20permit-token-address-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)

#### safeTransfer(contract IERC20 token, address to, uint256 value)
internal#
将代币的价值数量从调用合约转移到to地址。如果代币没有返回值，则假定不会发生错误的调用是成功的。

#### safeTransferFrom(contract IERC20 token, address from, address to, uint256 value)
internal#
从发送者向接收者转移指定数量的代币价值，并使用发送者授权给调用合约的批准。如果代币没有返回任何值，则假定非撤销调用成功。

#### safeApprove(contract IERC20 token, address spender, uint256 value)
internal#
已弃用。此功能存在与[IERC20.approve](#approveaddress-spender-uint256-amount-→-bool)类似的问题，不建议使用。
尽可能使用[safeIncreaseAllowance](#safeincreaseallowancecontract-ierc20-token-address-spender-uint256-value)和[safeDecreaseAllowance](#safedecreaseallowancecontract-ierc20-token-address-spender-uint256-value)。

#### safeIncreaseAllowance(contract IERC20 token, address spender, uint256 value)
internal#
增加调用合约对spender的授权值。如果代币没有返回值，则假定非reverting调用成功。

#### safeDecreaseAllowance(contract IERC20 token, address spender, uint256 value)
internal#
将调用合约对spender的授权减少value。如果代币没有返回值，则假定非回滚调用成功。

#### forceApprove(contract IERC20 token, address spender, uint256 value)
internal#
将调用合约的授权向spender设置为value。如果代币没有返回任何值，则假定非reverting调用成功。与需要将批准设置为0后再将其设置为非零值的代币兼容。

#### safePermit(contract IERC20Permit token, address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
internal#
使用 ERC-2612 签名来设置代币所有者对支出者的批准。在无效签名的情况下回滚。

### TokenTimelock
```
import "@openzeppelin/contracts/token/ERC20/utils/TokenTimelock.sol";
```

一个代币持有者合约，允许受益人在给定的释放时间后提取代币。
适用于简单的归属计划，如“顾问在1年后获得全部代币”。

**FUNCTIONS**
[constructor(token_, beneficiary_, releaseTime_)](#constructorcontract-ierc20-token_-address-beneficiary_-uint256-releasetime_)
[token()](#token-→-contract-ierc20)
[beneficiary()](#beneficiary-→-address)
[releaseTime()](#releasetime-→-uint256)
[release()](#release)

#### constructor(contract IERC20 token_, address beneficiary_, uint256 releaseTime_)
public#
部署一个时间锁实例，能够持有指定的代币，并在 *releaseTime_* 后调用 release 时才释放给 beneficiary_。释放时间以 Unix 时间戳（以秒为单位）指定。

#### token() → contract IERC20
public#
返回当前持有的代币。

#### beneficiary() → address
public#
返回将收到代币的受益人。

#### releaseTime() → uint256
public#
返回代币释放时间，以自 Unix 纪元以来的秒数表示（即 Unix 时间戳）。

#### release()
public#
将由时间锁持有的代币转移给受益人。只有在释放时间之后调用才会成功。