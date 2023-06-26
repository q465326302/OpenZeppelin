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