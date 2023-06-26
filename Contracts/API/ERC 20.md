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
要了解更多关于挂钩的信息，请前往使用*挂钩*。

#### _afterTokenTransfer(address from, address to, uint256 amount)
内部#
在任何代币转移之后调用的钩子。这包括铸造和销毁。
调用条件：
* 当 from 和 to 都不为零时，将 from 的代币数量转移到 to。
* 当 from 为零时，为 to 铸造了 amount 个代币。
* 当 to 为零时，从 from 销毁了 amount 个代币。
* from 和 to 永远不会同时为零。
要了解更多关于钩子的信息，请前往使用*挂钩*。

## 扩展程序

### ERC20Burnable
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
```
*ERC20*的扩展，允许代币持有者销毁他们自己的代币和他们允许的代币，以一种可以通过事件分析在链外识别的方式。

**FUNCTIONS**
burn(amount)
burnFrom(account, amount)

ERC20
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

#### burn(uint256 amount)
公开#
销毁调用者的代币数量。
参见 *ERC20._burn*。

#### burnFrom(address account, uint256 amount)
公开#
销毁账户中指定数量的代币，从调用者的授权中扣除。
请参阅*ERC20._burn*和*ERC20.allowance*。

要求：
* 调用者必须具有至少等于销毁数量的账户代币授权。

#### ERC20Capped
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Capped.sol";
```
将ERC20扩展，以添加令牌供应量上限。

**FUNCTIONS**
constructor(cap_)
cap()
_mint(account, amount)

ERC20
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
_burn(account, amount)
_approve(owner, spender, amount)
_spendAllowance(owner, spender, amount)
_beforeTokenTransfer(from, to, amount)
_afterTokenTransfer(from, to, amount)

**EVENTS**

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### constructor(uint256 cap_)
内部#
设置帽子的价值。这个值是不可变的，只能在构造过程中设置一次。

#### cap() → uint256
公开#
返回代币总供应量的上限。

#### _mint(address account, uint256 amount)
内部#
请参阅ERC20._mint。

### ERC20Pausable
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Pausable.sol";
```
具有可暂停令牌转移、铸造和销毁的ERC20令牌。
适用于防止交易直到评估期结束或在出现大规模漏洞时有一个紧急开关以冻结所有令牌转移的情况等场景。
>IMPORTANT
此合约不包括公共暂停和取消暂停函数。除了继承此合约外，您还必须定义这两个函数，并使用适当的访问控制（例如使用*AccessControl*或*Ownable*）调用*Pausable._pause*和*Pausable._unpause*内部函数。否则，合约将无法暂停。

**FUNCTIONS**
_beforeTokenTransfer(from, to, amount)

PAUSABLE
paused()
_requireNotPaused()
_requirePaused()
_pause()
_unpause()

ERC20
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
_afterTokenTransfer(from, to, amount)

**EVENTS**

PAUSABLE
Paused(account)
Unpaused(account)

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### _beforeTokenTransfer(address from, address to, uint256 amount)
内部#
请参阅 *ERC20._beforeTokenTransfer*.
要求：
* 合约不能暂停。

### ERC20Permit
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";
```
实现了ERC20 Permit扩展，允许通过签名进行批准，该扩展在*EIP-2612*中定义。
添加了*permit*方法，可以通过提供由账户签名的消息来改变账户的ERC20允许额度（参见*IERC20.allowance*）。通过不依赖*IERC20.approve*，代币持有人账户不需要发送交易，因此根本不需要持有以太币。
*自v3.4版本以来可用。*

**FUNCTIONS**
constructor(name)
permit(owner, spender, value, deadline, v, r, s)
nonces(owner)
DOMAIN_SEPARATOR()
_useNonce(owner)

EIP712
_domainSeparatorV4()
_hashTypedDataV4(structHash)
eip712Domain()

ERC20
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

IERC5267
EIP712DomainChanged()

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### constructor(string name)
内部#
使用name参数初始化*EIP712*域分隔符，并将version设置为“1”。最好使用与ERC20代币名称相同的名称。

#### permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
公开#
请参阅 *IERC20Permit.permit*.

#### nonces(address owner) → uint256
公开#
请参阅 *IERC20Permit.nonces*.

#### DOMAIN_SEPARATOR() → bytes32
外部#
请参阅 *IERC20Permit.DOMAIN_SEPARATOR*.

#### _useNonce(address owner) → uint256 current
内部#
"消耗一个随机数": 返回当前值并递增。 
*自v4.1版本起可用。*

### ERC20Snapshot
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Snapshot.sol";
```
这份合约扩展了一个带有快照机制的ERC20代币。当创建快照时，记录当前时间的余额和总供应量以便以后访问。

这可以用于安全地创建基于代币余额的机制，例如无信任股息或加权投票。在天真的实现中，可以通过从不同的账户重复使用同一余额来执行“双重花费”攻击。通过使用快照来计算股息或投票权，这些攻击不再适用。它也可以用于创建高效的ERC20分叉机制。

快照是通过内部的 *_snapshot* 函数创建的，该函数将触发*Snapshot*事件并返回快照ID。要获取快照时的总供应量，请使用快照ID调用 *totalSupplyAt* 函数。要获取快照时某个账户的余额，请使用快照ID和账户地址调用 *balanceOfAt* 函数。

>NOTE
可以通过覆盖 *_getCurrentSnapshotId* 方法来自定义快照策略。例如，让它返回 block.number 将在每个新块的开头触发快照的创建。在覆盖此函数时，要小心其结果的单调性。非单调的快照ID会破坏合约。

使用此方法为每个块实现快照将产生显著的燃气成本。对于燃气效率更高的替代方案，请考虑使用 *ERC20Votes*。

#### 燃气成本
快照是高效的。快照创建是 O(1)。从快照中检索余额或总供应量的成本是 O(log n)，其中 n 是已创建的快照数量，尽管对于特定账户的 n 通常会更小，因为连续快照中相同的余额将存储为一个条目。

由于额外的快照记账，普通 ERC20 转账存在常数开销。对于特定账户的第一次转账，这种开销是显著的。随后的转账将具有正常的成本，直到下一次快照，以此类推。

**FUNCTIONS**
_snapshot()
_getCurrentSnapshotId()
balanceOfAt(account, snapshotId)
totalSupplyAt(snapshotId)
_beforeTokenTransfer(from, to, amount)

ERC20
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
_afterTokenTransfer(from, to, amount)

#### _snapshot() → uint256
内部#
创建一个新的快照并返回其快照ID。
触发一个包含相同ID的*Snapshot* 事件。
*_snapshot*是内部的，你必须决定如何将其外部公开。它的使用可能会受到一组帐户的限制，例如使用*AccessControl*，或者它可能向公众开放。

>WARNING
虽然需要一种开放的调用*_snapshot*的方式来实现某些信任最小化机制，例如分叉，但您必须考虑它可能被攻击者以两种方式使用。

首先，它可以用于增加从快照检索值的成本，尽管它将呈对数增长，因此从长远来看，这种攻击将变得无效。其次，它可以用于针对特定帐户，并增加ERC20转移对它们的成本，以Gas Costs部分中指定的方式。

我们没有测量实际数字；如果您对此有兴趣，请与我们联系。

#### _getCurrentSnapshotId() → uint256
内部#
获取当前snapshotId

#### balanceOfAt(address account, uint256 snapshotId) → uint256
公开#
检索创建snapshotId时账户的余额。

#### totalSupplyAt(uint256 snapshotId) → uint256
公开#
检索创建snapshotId时的总供应量。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
内部#
在任何代币转移之前调用的钩子。这包括铸造和销毁。

调用条件：
* 当 from 和 to 都不为零时，from 的代币数量将转移到 to。
* 当 from 为零时，将为 to 铸造 amount 个代币。
* 当 to 为零时，将销毁 from 的代币数量。
* from 和 to 永远不会同时为零。

要了解更多有关钩子的信息，请前往使用*钩子*。

#### Snapshot(uint256 id)
事件#
当由id标识的快照创建时，由_snapshot发出。

#### ERC20Votes
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
```

ERC20的扩展，支持类似Compound的投票和委托。这个版本比Compound的更通用，支持令牌供应量高达2^224-1, 而COMP仅限于2^96-1。

>NOTE
如果需要完全兼容COMP，则使用此模块的*ERC20VotesComp*变体。

此扩展会保留每个帐户的投票权历史记录（检查点）。投票权可以通过直接调用*委托*函数或提供签名以与*delegateBySig*一起使用来委托。可以通过公共访问器*getVotes*和*getPastVotes*查询投票权。

默认情况下，令牌余额不考虑投票权。这使转账更便宜。缺点是需要用户委托给自己以激活检查点并跟踪其投票权。

*自v4.2以来可用。*

**FUNCTIONS**
clock()
CLOCK_MODE()
checkpoints(account, pos)
numCheckpoints(account)
delegates(account)
getVotes(account)
getPastVotes(account, timepoint)
getPastTotalSupply(timepoint)
delegate(delegatee)
delegateBySig(delegatee, nonce, expiry, v, r, s)
_maxSupply()
_mint(account, amount)
_burn(account, amount)
_afterTokenTransfer(from, to, amount)
_delegate(delegator, delegatee)

ERC20PERMIT
permit(owner, spender, value, deadline, v, r, s)
nonces(owner)
DOMAIN_SEPARATOR()
_useNonce(owner)

EIP712
_domainSeparatorV4()
_hashTypedDataV4(structHash)
eip712Domain()

ERC20
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
_approve(owner, spender, amount)
_spendAllowance(owner, spender, amount)
_beforeTokenTransfer(from, to, amount)

**EVENTS**
IVOTES
DelegateChanged(delegator, fromDelegate, toDelegate)
DelegateVotesChanged(delegate, previousBalance, newBalance)

IERC5267
EIP712DomainChanged()

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### clock() → uint48
公开#
用于标记检查点的时钟。可以被覆盖以实现基于时间戳的检查点（和投票）。

#### CLOCK_MODE() → string
公开#
时钟的描述

#### checkpoints(address account, uint32 pos) → struct ERC20Votes.Checkpoint
公开#
获取账户的第pos个检查点。

#### numCheckpoints(address account) → uint32
公开#
获取账户的检查点数量。

#### delegates(address account) → address
公开#
获取当前账户正在委托给的地址。

#### getVotes(address account) → uint256
公开#
获取账户当前的投票余额

#### getPastVotes(address account, uint256 timepoint) → uint256
公开#
获取账户在时间点结束时的投票数。

要求：
* 时间点必须在过去。

#### getPastTotalSupply(uint256 timepoint) → uint256
公开#
在时间点结束时检索totalSupply。注意，这个值是所有余额的总和。它不是所有委托投票的总和！

要求：
* 时间点必须在过去。

#### delegate(address delegatee)
公开#
将选票从发件人委派给受委托人。

#### delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
公开#
将签署者的投票委托给代表

#### _maxSupply() → uint224
内部#
最大代币供应量。默认为type(uint224).max（2^224-1）。

#### _mint(address account, uint256 amount)
内部#
在总供应量增加之后totalSupply。

#### _burn(address account, uint256 amount)
内部#
在减少totalSupply之后Snapshots。

#### _afterTokenTransfer(address from, address to, uint256 amount)
内部#
当代币转移时移动投票权。
发出一个*IVotes.DelegateVotesChanged*事件。

#### _delegate(address delegator, address delegatee)
内部#
将委托从委托者更改为委托受托人。

发出事件 *IVotes.DelegateChanged* 和 *IVotes.DelegateVotesChanged*。

### ERC20VotesComp
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20VotesComp.sol";
```
ERC20的扩展，支持Compound的投票和委托。该版本完全符合Compound的接口，但缺点是只支持供应量高达（2^96-1）。

>NOTE
如果您需要与COMP完全兼容（例如为了在Governor Alpha或Bravo中使用您的代币），并且您确定2^96的供应上限足够您使用，请使用此合约。否则，请使用此模块的*ERC20Votes*变体。

此扩展保留每个帐户投票权的历史记录（检查点）。投票权可以通过直接调用*委托*函数或提供用于*delegateBySig*的签名来委托。可以通过公共访问器*getCurrentVotes*和*getPriorVotes*查询投票权。

默认情况下，代币余额不考虑投票权。这使得转账更便宜。缺点是需要用户委托给自己，以激活检查点并跟踪其投票权。

*自v4.2以来可用。*

**FUNCTIONS**
getCurrentVotes(account)
getPriorVotes(account, blockNumber)
_maxSupply()

ERC20VOTES
clock()
CLOCK_MODE()
checkpoints(account, pos)
numCheckpoints(account)
delegates(account)
getVotes(account)
getPastVotes(account, timepoint)
getPastTotalSupply(timepoint)
delegate(delegatee)
delegateBySig(delegatee, nonce, expiry, v, r, s)
_mint(account, amount)
_burn(account, amount)
_afterTokenTransfer(from, to, amount)
_delegate(delegator, delegatee)

ERC20PERMIT
permit(owner, spender, value, deadline, v, r, s)
nonces(owner)
DOMAIN_SEPARATOR()
_useNonce(owner)

EIP712
_domainSeparatorV4()
_hashTypedDataV4(structHash)
eip712Domain()

ERC20
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
_approve(owner, spender, amount)
_spendAllowance(owner, spender, amount)
_beforeTokenTransfer(from, to, amount)

**EVENTS**

IVOTES
DelegateChanged(delegator, fromDelegate, toDelegate)
DelegateVotesChanged(delegate, previousBalance, newBalance)

IERC5267
EIP712DomainChanged()

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### getCurrentVotes(address account) → uint96
外部#
*getVotes*信息的Comp版本访问器，返回类型为uint96。

#### getPriorVotes(address account, uint256 blockNumber) → uint96
外部#
*getPastVotes*记录的访问器的Comp版本，返回类型为uint96。

#### _maxSupply() → uint224
内部#
最大代币供应量。缩小至type(uint96).max（2^96-1）以适应COMP接口。

### ERC20Wrapper
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Wrapper.sol";
```

ERC20代币合约的扩展，以支持代币包装。

用户可以存入和取出“基础代币”，并获得相应数量的“包装代币”。这在与其他模块结合使用时非常有用。例如，将此包装机制与*ERC20Votes*结合使用，可以将现有的“基本”ERC20包装成治理代币。

*自v4.2以来可用。*

**FUNCTIONS**
constructor(underlyingToken)
decimals()
underlying()
depositFor(account, amount)
withdrawTo(account, amount)
_recover(account)

ERC20
name()
symbol()
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

#### constructor(contract IERC20 underlyingToken)
内部#

#### decimals() → uint8
公开#
请查看 *ERC20.decimals*.

#### underlying() → contract IERC20
公开#
返回被包装的基础ERC返回被封装的基础ERC-20代币的地址。

#### depositFor(address account, uint256 amount) → bool
公开#
允许用户存入基础代币，并铸造相应数量的封装代币。

#### withdrawTo(address account, uint256 amount) → bool
公开#
允许用户烧掉一定数量的封装代币，并提取相应数量的基础代币。

#### _recover(address account) → uint256
内部#
Mint包装代币以覆盖可能因错误而转移的任何基础代币。如果需要，此内部函数可以通过访问控制公开。

### ERC20FlashMint
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20FlashMint.sol";
```
实现 ERC3156 闪电贷扩展，如 [ERC-3156](https://eips.ethereum.org/EIPS/eip-3156) 中所定义。

添加 *flashLoan* 方法，为代币级别提供闪电贷支持。默认情况下没有费用，但可以通过覆盖 *flashFee* 进行更改。

*自 v4.1 版本起可用。*

**FUNCTIONS**
maxFlashLoan(token)
flashFee(token, amount)
_flashFee(token, amount)
_flashFeeReceiver()
flashLoan(receiver, token, amount, data)

ERC20
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

#### maxFlashLoan(address token) → uint256
公开#
返回可供借贷的最大代币数量。

#### flashFee(address token, uint256 amount) → uint256
公开#
返回执行闪电贷款时应用的费用。该函数调用 *_flashFee* 函数，该函数返回执行闪电贷款时应用的费用。

#### _flashFee(address token, uint256 amount) → uint256
内部#
返回进行闪电贷款时应用的费用。默认情况下，此实现没有任何费用。可以重载此函数，使闪电贷款机制通缩。

#### _flashFeeReceiver() → address
内部#
返回闪电交易手续费的接收地址。默认情况下，此实现返回地址(0)，这意味着手续费金额将被烧掉。可以重载此函数以更改手续费接收者。

#### flashLoan(contract IERC3156FlashBorrower receiver, address token, uint256 amount, bytes data) → bool
公开#
执行闪电贷款。新的代币被铸造并发送给接收方，接收方需要实现*IERC3156FlashBorrower*接口。在闪电贷款结束时，接收方应当拥有amount + fee代币并将其批准回到代币合约本身，以便它们可以被销毁。

### ERC4626
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC4626.sol";
```
实施 ERC4626“代币化保险库标准”，如 [EIP-4626](https://eips.ethereum.org/EIPS/eip-4626) 中定义的。

该扩展允许通过标准化的*存款*、*铸造*、*赎回*和*销毁*流程，以“资产”为代价铸造和销毁“股份”（使用 ERC20 继承表示）。此合约扩展了 ERC20 标准。与它一起包含的任何其他扩展都会影响由此合约表示的“股份”代币，而不是独立合约的“资产”代币。

>CAUTION
在空的（或几乎空的）ERC-4626保险库中，存款有高风险通过“捐款”到保险库中被抢走，从而使股份的价格膨胀。这通常称为“捐赠”或通货膨胀攻击，本质上是滑点问题。保险库部署者可以通过进行一笔非微不足道的资产存款来防止这种攻击，从而使价格操纵变得不可行。提款也可能受到滑点的影响。用户也可以通过验证所接收的金额与预期金额相符来保护自己免受这种攻击以及意外滑点的影响，使用执行这些检查的包装器，例如 [ERC4626Router](https://github.com/fei-protocol/ERC4626#erc4626router-and-base)。

自 v4.9 以来，该实现使用虚拟资产和股份来缓解这种风险。_decimalsOffset() 对应于基础资产的小数位数和保险库小数位数之间的小数表示之间的偏移量。此偏移量还确定了保险库中虚拟股份与虚拟资产之间的比率，这本身确定了初始汇率。虽然不能完全防止攻击，但分析表明，默认偏移量（0）使其不赚钱，因为虚拟股份捕获的价值（来自攻击者的捐赠）与攻击者的预期收益相匹配。使用较大的偏移量，攻击变得比盈利昂贵几个数量级。有关底层数学的更多详细信息，请参见*此处*。

这种方法的缺点是虚拟股份确实捕获了（非常小的）部分累积到保险库的价值。此外，如果保险库遭受损失，用户试图退出保险库，虚拟股份和资产将导致第一个用户退出时经历减少的损失，而最后的用户将经历更大的损失。愿意恢复到 v4.9 之前行为的开发人员只需覆盖 _convertToShares 和 _convertToAssets 函数即可。

要了解更多信息，请查看我们的 *ERC-4626 指南*。

*自 v4.7 以来可用。*

**FUNCTIONS**
constructor(asset_)
decimals()
asset()
totalAssets()
convertToShares(assets)
convertToAssets(shares)
maxDeposit()
maxMint()
maxWithdraw(owner)
maxRedeem(owner)
previewDeposit(assets)
previewMint(shares)
previewWithdraw(assets)
previewRedeem(shares)
deposit(assets, receiver)
mint(shares, receiver)
withdraw(assets, receiver, owner)
redeem(shares, receiver, owner)
_convertToShares(assets, rounding)
_convertToAssets(shares, rounding)
_deposit(caller, receiver, assets, shares)
_withdraw(caller, receiver, owner, assets, shares)
_decimalsOffset()

ERC20
name()
symbol()
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
IERC4626
Deposit(sender, owner, assets, shares)
Withdraw(sender, receiver, owner, assets, shares)

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### constructor(contract IERC20 asset_)
内部#
设置基础资产合约。这必须是一个ERC20兼容合约（ERC20或ERC777）。

#### decimals() → uint8
公开#
十进制数是通过将十进制偏移量加到基础资产的小数位上计算出来的。在构建保险库合约期间，这个“原始”值被缓存。如果读取操作失败（例如，资产尚未创建），则默认值为18，表示基础资产的小数位数。

请参阅*IERC20Metadata.decimals*。

#### asset() → address
公开#
请参阅*IERC4626.asset*.

#### totalAssets() → uint256
公开#
请参阅*IERC4626.totalAssets*.

#### convertToShares(uint256 assets) → uint256
公开#
请参阅*IERC4626.convertToShares*.

#### convertToAssets(uint256 shares) → uint256
公开#
请参阅*IERC4626.convertToAssets*.

#### maxDeposit(address) → uint256
公开#
请参阅 *IERC4626.maxDeposit*.

##### maxMint(address) → uint256
公开#
请参阅 *IERC4626.maxMint*.

#### maxWithdraw(address owner) → uint256
公开#
请参阅 *IERC4626.maxWithdraw*.

#### maxRedeem(address owner) → uint256
公开#
请参阅 *IERC4626.maxRedeem*.

#### previewDeposit(uint256 assets) → uint256
公开#
请参阅 *IERC4626.previewDeposit*.

#### previewMint(uint256 shares) → uint256
公开#
请参阅 *IERC4626.previewMint*.

####  previewWithdraw(uint256 assets) → uint256
公开#
请参阅 *IERC4626.previewWithdraw*.

#### previewRedeem(uint256 shares) → uint256
公开#
请参阅 *IERC4626.previewRedeem*.

#### deposit(uint256 assets, address receiver) → uint256
公开#
请参阅 *IERC4626.deposit*.

#### mint(uint256 shares, address receiver) → uint256
公开#
请参阅 *IERC4626.mint*.
与*deposit*相反，即使保险库处于份额价格为零的状态，也允许进行铸造。在这种情况下，股份将被铸造而无需存入任何资产。

#### withdraw(uint256 assets, address receiver, address owner) → uint256
公开#
请参阅 *IERC4626.withdraw*.

#### redeem(uint256 shares, address receiver, address owner) → uint256
公开#
请参阅*IERC4626.redeem*.

#### _convertToShares(uint256 assets, enum Math.Rounding rounding) → uint256
内部#
内部转换函数（从资产到股份），支持舍入方向。

#### _convertToAssets(uint256 shares, enum Math.Rounding rounding) → uint256
内部#
支持舍入方向的内部转换函数（从股份转换为资产）。

#### _deposit(address caller, address receiver, uint256 assets, uint256 shares)
内部#
存款/铸造常见工作流程。

#### _withdraw(address caller, address receiver, address owner, uint256 assets, uint256 shares)
内部#
撤回/兑现常见的工作流程。

#### _decimalsOffset() → uint8
内部#

## 预设
这些合约是上述功能的预配置组合。它们可以通过继承或复制粘贴其源代码作为模型使用。

### ERC20PresetMinterPauser
```
import "@openzeppelin/contracts/token/ERC20/presets/ERC20PresetMinterPauser.sol";
```
*ERC20*代币，包括：
* 持有者可以烧毁（销毁）他们的代币的能力
* 允许代币铸造（创建）的铸造者角色
* 允许停止所有代币转移的暂停者角色

该合约使用*AccessControl*锁定使用不同角色的权限函数-有关详细信息，请查阅其文档。

部署合约的账户将被授予铸造者和暂停者角色，以及默认管理员角色，这将使其授予其他账户铸造者和暂停者角色。

弃用，有利于[合同向导](https://wizard.openzeppelin.com/)。

**FUNCTIONS**
constructor(name, symbol)
mint(to, amount)
pause()
unpause()
_beforeTokenTransfer(from, to, amount)

PAUSABLE
paused()
_requireNotPaused()
_requirePaused()
_pause()
_unpause()

ERC20BURNABLE
burn(amount)
burnFrom(account, amount)

ERC20
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
_afterTokenTransfer(from, to, amount)

ACCESSCONTROLENUMERABLE
supportsInterface(interfaceId)
getRoleMember(role, index)
getRoleMemberCount(role)
_grantRole(role, account)
_revokeRole(role, account)

ACCESSCONTROL
hasRole(role, account)
_checkRole(role)
_checkRole(role, account)
getRoleAdmin(role)
grantRole(role, account)
revokeRole(role, account)
renounceRole(role, account)
_setupRole(role, account)
_setRoleAdmin(role, adminRole)

**EVENTS**
PAUSABLE
Paused(account)
Unpaused(account)

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

IACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)
RoleGranted(role, account, sender)
RoleRevoked(role, account, sender)

#### constructor(string name, string symbol)
公开#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授权给部署合约的账户。
请参阅*ERC20.constructor*。

#### mint(address to, uint256 amount)
公开#
为to创建指定数量的新代币。
请参考*ERC20._mint*。
要求：
* 调用者必须具有MINTER_ROLE角色。

#### pause()
公开#
暂停所有代币转移。
请参阅 *ERC20Pausable* 和 *Pausable._pause*。
要求：
* 调用者必须具有 PAUSER_ROLE。

#### unpause()
公开#
取消所有代币转移的暂停状态。
请参阅*ERC20Pausable*和*Pausable._unpause*。
要求：
* 调用者必须具有PAUSER_ROLE。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
内部#

### ERC20PresetFixedSupply
```
import "@openzeppelin/contracts/token/ERC20/presets/ERC20PresetFixedSupply.sol";
```
*ERC20*代币，包括：
* 预先发行的初始供应量
* 持有人可以烧毁（销毁）他们的代币的能力
* 没有访问控制机制（用于铸造/暂停），因此没有治理

该合同使用*ERC20Burnable*包括烧毁功能-请查看其文档以获取详细信息。

*自v3.4以来可用。*

弃用，推荐使用[Contracts Wizard](https://wizard.openzeppelin.com/)。

**FUNCTIONS**
constructor(name, symbol, initialSupply, owner)

ERC20BURNABLE
burn(amount)
burnFrom(account, amount)

ERC20
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

#### constructor(string name, string symbol, uint256 initialSupply, address owner)
公开#
Mint函数会初始化一定数量的代币并将它们转移到所有者账户中。
请参阅*ERC20.constructor*。

## 应用程序

### SafeERC20
```
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
SafeERC20是一种在ERC20操作失败时（当代币合约返回false时）抛出异常的包装器。它还支持返回无值（而是在失败时回滚或抛出异常）的代币，假定非回滚调用是成功的。要使用此库，您可以在合约中添加一个using SafeERC20 for IERC20;语句，这允许您调用安全操作，例如token.safeTransfer(…​)等。

**FUNCTIONS**
safeTransfer(token, to, value)
safeTransferFrom(token, from, to, value)
safeApprove(token, spender, value)
safeIncreaseAllowance(token, spender, value)
safeDecreaseAllowance(token, spender, value)
forceApprove(token, spender, value)
safePermit(token, owner, spender, value, deadline, v, r, s)

#### safeTransfer(contract IERC20 token, address to, uint256 value)
内部#
将代币的价值数量从调用合约转移到to地址。如果代币没有返回值，则假定不会发生错误的调用是成功的。

#### safeTransferFrom(contract IERC20 token, address from, address to, uint256 value)
内部#
从发送者向接收者转移指定数量的代币价值，并使用发送者授权给调用合约的批准。如果代币没有返回任何值，则假定非撤销调用成功。

#### safeApprove(contract IERC20 token, address spender, uint256 value)
内部#
已弃用。此功能存在与*IERC20.approve*类似的问题，不建议使用。
尽可能使用*safeIncreaseAllowance*和*safeDecreaseAllowance*。

#### safeIncreaseAllowance(contract IERC20 token, address spender, uint256 value)
内部#
增加调用合约对spender的授权值。如果代币没有返回值，则假定非reverting调用成功。

#### safeDecreaseAllowance(contract IERC20 token, address spender, uint256 value)
内部#
将调用合约对spender的授权减少value。如果代币没有返回值，则假定非回滚调用成功。

#### forceApprove(contract IERC20 token, address spender, uint256 value)
内部#
将调用合约的授权向spender设置为value。如果代币没有返回任何值，则假定非reverting调用成功。与需要将批准设置为0后再将其设置为非零值的代币兼容。

#### safePermit(contract IERC20Permit token, address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
内部#
使用 ERC-2612 签名来设置令牌所有者对支出者的批准。在无效签名的情况下回滚。

### TokenTimelock
```
import "@openzeppelin/contracts/token/ERC20/utils/TokenTimelock.sol";
```
一个代币持有者合约，允许受益人在给定的释放时间后提取代币。
适用于简单的归属计划，如“顾问在1年后获得全部代币”。

**FUNCTIONS**
constructor(token_, beneficiary_, releaseTime_)
token()
beneficiary()
releaseTime()
release()

#### constructor(contract IERC20 token_, address beneficiary_, uint256 releaseTime_)
公开#
部署一个时间锁实例，能够持有指定的代币，并在 *releaseTime_* 后调用 release 时才释放给 beneficiary_。释放时间以 Unix 时间戳（以秒为单位）指定。

#### token() → contract IERC20
公开#
返回当前持有的令牌。

#### beneficiary() → address
公开#
返回将收到代币的受益人。

#### releaseTime() → uint256
公开#
返回令牌释放时间，以自 Unix 纪元以来的秒数表示（即 Unix 时间戳）。

#### release()
公开#
将由时间锁持有的代币转移给受益人。只有在释放时间之后调用才会成功。