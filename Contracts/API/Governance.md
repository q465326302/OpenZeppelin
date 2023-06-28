# Governance
该目录包括用于链上治理的原语。

## Governor
这个模块化的Governor合约系统允许部署类似于[Compound的Governor Alpha和Bravo](https://compound.finance/docs/governance)等链上投票协议，通过能够轻松定制协议的多个方面。

> TIP
对于一个引导式体验，可以使用[Contracts Wizard](https://wizard.openzeppelin.com/#governor)来设置您的Governor合约。
对于一个书面步骤，可以查看我们的*《如何设置链上治理》指南*。

* *Governor*: 包含所有逻辑和原语的核心合约。它是抽象的，并需要选择下面每个模块中的一个，或者自定义模块。

Votes模块确定投票权的来源，有时也确定法定人数。

* *GovernorVotes*: 从*ERC20Votes*或v4.5以后的*ERC721Votes*代币中提取投票权重。
* *GovernorVotesComp*: 从类似于COMP或*ERC20VotesComp*代币中提取投票权重。
* *GovernorVotesQuorumFraction*: 与GovernorVotes结合使用，将法定人数设置为总代币供应量的一部分。

Counting模块确定有效的投票选项。

* *GovernorCountingSimple*: 简单的投票机制，有3个投票选项：反对、赞成和弃权。

Timelock扩展为治理决策添加了延迟。工作流程扩展为在执行之前需要一个队列步骤。有了这些模块，提案将由外部的Timelock合约执行，因此Timelock必须持有被治理的资产。

* *GovernorTimelockControl*: 连接到*TimelockController*的实例。允许多个提案者和执行者，以及Governor本身。
* *GovernorTimelockCompound*: 连接到Compound的*Timelock*合约的实例。

其他扩展可以以多种方式定制行为或接口。

* *GovernorCompatibilityBravo*: 扩展接口以完全兼容GovernorBravo。请注意，无论是否包含此扩展，事件都是兼容的。

* *GovernorSettings*: 以可以通过治理提案更新的方式管理一些设置（投票延迟、投票期间持续时间和提案阈值），而无需进行升级。

* *GovernorPreventLateQuorum*: 确保在达到法定人数后有一个最小的投票期限，以作为对大型投票人的安全防护措施。

除了模块和扩展之外，核心合约还需要实现一些虚拟函数来满足您的特定规格要求：

* *votingDelay()*: 提案提交后到固定投票权和开始投票之间的延迟（以EIP-6372时钟为单位）。这可以用来强制在提案发布后用户购买代币或委托他们的投票之前进行延迟。

* *votingPeriod()*: 提案开始后到投票结束之间的延迟（以EIP-6372时钟为单位）。

* *quorum(uint256 timepoint)*: 提案成功所需的法定人数。此函数包括一个时间点参数（参见EIP-6372），以便法定人数可以随时间变化，例如跟随代币的totalSupply。

> NOTE
Governor合约的函数不包括访问控制。如果要限制访问，应通过重载特定函数来添加这些检查。其中，Governor._cancel默认是internal的，如果需要使用此函数，您将需要自己公开它（并使用正确的访问控制机制）。

## 核心

### IGovernor
```
import "@openzeppelin/contracts/governance/IGovernor.sol";
```
*Governor* 核心的界面。

*从v4.3版本开始可用。*

**FUNCTIONS**
name()
version()
clock()
CLOCK_MODE()
COUNTING_MODE()
hashProposal(targets, values, calldatas, descriptionHash)
state(proposalId)
proposalSnapshot(proposalId)
proposalDeadline(proposalId)
proposalProposer(proposalId)
votingDelay()
votingPeriod()
quorum(timepoint)
getVotes(account, timepoint)
getVotesWithParams(account, timepoint, params)
hasVoted(proposalId, account)
propose(targets, values, calldatas, description)
execute(targets, values, calldatas, descriptionHash)
cancel(targets, values, calldatas, descriptionHash)
castVote(proposalId, support)
castVoteWithReason(proposalId, support, reason)
castVoteWithReasonAndParams(proposalId, support, reason, params)
castVoteBySig(proposalId, support, v, r, s)
castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)

IERC165
supportsInterface(interfaceId)

**EVENTS**
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)
ProposalCanceled(proposalId)
ProposalExecuted(proposalId)
VoteCast(voter, proposalId, support, weight, reason)
VoteCastWithParams(voter, proposalId, support, weight, reason, params)

#### name() → string
公开#
构建ERC712域分隔符时使用的governor实例的名称。

#### version() → string
公开#
governor实例的版本（用于构建ERC712域分隔符）。默认值为“1”。

#### clock() → uint48
公开#
请参阅 *IERC6372*

#### CLOCK_MODE() → string
公开#
请参阅 *EIP-6372.*

#### COUNTING_MODE() → string
公开#
这是一个描述*castVote*可能的支持值以及这些投票如何计算的字符串，用于UI显示正确的投票选项和解释结果。该字符串是一个URL编码的键值对序列，每个键值对描述一个方面，例如support=bravo&quorum=for,abstain。

有两个标准键：support和quorum。

* support=bravo表示投票选项为0=反对，1=支持，2=弃权，例如GovernorBravo。
* quorum=bravo表示只有支持票计入法定人数。
* quorum=for,abstain表示支持票和弃权票都计入法定人数。

如果计数模块使用编码的参数，应将其包含在具有描述行为的唯一名称的params键下。例如：

* params=fractional可能表示一种将投票在支持/反对/弃权之间按比例分配的方案。
* params=erc721可能表示一种将特定的NFT委托进行投票的方案。

> NOTE
该字符串可以通过标准的[URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) JavaScript类解码。

#### hashProposal(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
公开#
用于从提案详细信息中重新构建提案ID的哈希函数。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
公开#
根据Compound的约定，提案的当前状态。

#### proposalSnapshot(uint256 proposalId) → uint256
公开#
用于检索用户投票和法定人数的时间点。如果使用块编号（按照Compound的Comp），快照将在该块结束时执行。因此，对该提案的投票将从以下块的开始开始。

#### proposalDeadline(uint256 proposalId) → uint256
公开#
投票截止时间点。如果使用块编号，则投票在该块结束时关闭，因此可以在该块期间投票。

#### proposalProposer(uint256 proposalId) → address
公开#
创建提案的账户。

#### votingDelay() → uint256
公开#
在提案创建和投票开始之间存在延迟。此持续时间的单位取决于此合约使用的时钟（请参阅EIP-6372）。

可以增加此延迟时间，以便用户有时间购买投票权或委托投票权，然后再开始对提案进行投票。

#### votingPeriod() → uint256
公开#
投票开始和投票结束之间的延迟。此持续时间的单位取决于此合约使用的时钟（参见EIP-6372）。

votingDelay可以延迟投票的开始。在设置投票持续时间与投票延迟相比时，必须考虑到这一点。

#### quorum(uint256 timepoint) → uint256
公开#
成功提案所需的最低投票数。

> NOTE
时间点参数对应于用于计算投票的快照。这允许根据此时间点的总供应量等值来调整法定人数（请参阅*ERC20Votes*）。

#### getVotes(address account, uint256 timepoint) → uint256
公开#
特定时间点上帐户的投票权力。

注意：这可以通过多种方式实现，例如通过从一个（或多个）ERC20Votes代币中读取委托余额来实现。

#### getVotesWithParams(address account, uint256 timepoint, bytes params) → uint256
公开#
给定额外的编码参数，一个账户在特定时间点的投票权力。

#### hasVoted(uint256 proposalId, address account) → bool
公开#
返回帐户是否对提案ID投票。

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256 proposalId
公开#

创建一个新提案。投票在*IGovernor.votingDelay*指定的延迟后开始，并持续*IGovernor.votingPeriod*指定的持续时间。

发出一个*ProposalCreated*事件。

#### execute(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256 proposalId
公开#

成功执行提案。这需要达到法定人数，投票成功，并达到截止日期。

发布*ProposalExecuted*事件。

注意：某些模块可以修改执行要求，例如添加额外的时间锁。

#### cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256 proposalId
公开#
取消提案。提案可以被提出者取消，但只能在待定状态下取消，即在投票开始之前。

发出一个*ProposalCanceled*事件。

#### castVote(uint256 proposalId, uint8 support) → uint256 balance
公开#
投票

发出一个*VoteCast*事件。

#### castVoteWithReason(uint256 proposalId, uint8 support, string reason) → uint256 balance
公开#
投票并附上理由。

触发VoteCast事件。

#### castVoteWithReasonAndParams(uint256 proposalId, uint8 support, string reason, bytes params) → uint256 balance
公开#
根据参数的长度，发出*VoteCast*或*VoteCastWithParams*事件。

#### castVoteBySig(uint256 proposalId, uint8 support, uint8 v, bytes32 r, bytes32 s) → uint256 balance
公开#
使用用户的密码签名进行投票。

发出一个*VoteCast*事件。

#### castVoteWithReasonAndParamsBySig(uint256 proposalId, uint8 support, string reason, bytes params, uint8 v, bytes32 r, bytes32 s) → uint256 balance
公开#
使用用户的加密签名进行投票，并附加编码参数进行投票。

根据params的长度发出*VoteCast*或*VoteCastWithParams*事件。

#### ProposalCreated(uint256 proposalId, address proposer, address[] targets, uint256[] values, string[] signatures, bytes[] calldatas, uint256 voteStart, uint256 voteEnd, string description)
事件#
当提案被创建时发出。

#### ProposalCanceled(uint256 proposalId)
事件#
当一个提案被取消时发出的信号。

#### ProposalExecuted(uint256 proposalId)
事件#
当提案执行时发出。

#### VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason)
事件#
当没有参数投票时发出的信号。

注意：支持值应视为桶。它们的解释取决于所使用的投票模块。

#### VoteCastWithParams(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason, bytes params)
事件#
在使用参数进行投票时发出。

注意：支持值应被视为桶。它们的解释取决于所使用的投票模块。参数是额外编码的参数。它们的解释也取决于所使用的投票模块。

### Governor
```
import "@openzeppelin/contracts/governance/Governor.sol";
```
这个治理系统的核心是被设计成可以通过各种模块进行扩展的。

该合约是抽象的，并需要在各个模块中实现几个函数：

* 计数模块必须实现*quorum*、*_quorumReached*、*_voteSucceeded*和*_countVote*函数。

* 投票模块必须实现*_getVotes*函数。

* 此外，还必须实现*votingPeriod*函数。

*该功能从v4.3版本开始提供。*

**MODIFIERS**
onlyGovernance()

**FUNCTIONS**
constructor(name_)
receive()
supportsInterface(interfaceId)
name()
version()
hashProposal(targets, values, calldatas, descriptionHash)
state(proposalId)
proposalThreshold()
proposalSnapshot(proposalId)
proposalDeadline(proposalId)
proposalProposer(proposalId)
_quorumReached(proposalId)
_voteSucceeded(proposalId)
_getVotes(account, timepoint, params)
_countVote(proposalId, account, support, weight, params)
_defaultParams()
propose(targets, values, calldatas, description)
execute(targets, values, calldatas, descriptionHash)
cancel(targets, values, calldatas, descriptionHash)
_execute(, targets, values, calldatas, )
_beforeExecute(, targets, , calldatas, )
_afterExecute(, , , , )
_cancel(targets, values, calldatas, descriptionHash)
getVotes(account, timepoint)
getVotesWithParams(account, timepoint, params)
castVote(proposalId, support)
castVoteWithReason(proposalId, support, reason)
castVoteWithReasonAndParams(proposalId, support, reason, params)
castVoteBySig(proposalId, support, v, r, s)
castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)
_castVote(proposalId, account, support, reason)
_castVote(proposalId, account, support, reason, params)
relay(target, value, data)
_executor()
onERC721Received(, , , )
onERC1155Received(, , , , )
onERC1155BatchReceived(, , , , )
_isValidDescriptionForProposer(proposer, description)

IGOVERNOR
clock()
CLOCK_MODE()
COUNTING_MODE()
votingDelay()
votingPeriod()
quorum(timepoint)
hasVoted(proposalId, account)

EIP712
_domainSeparatorV4()
_hashTypedDataV4(structHash)
eip712Domain()

EVENTS
IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)
ProposalCanceled(proposalId)
ProposalExecuted(proposalId)
VoteCast(voter, proposalId, support, weight, reason)
VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

#### onlyGovernance()
修饰符#
限制一个函数，只能通过治理提案执行。例如，*GovernorSettings*中的治理参数设置器使用此修饰符进行保护。

治理执行地址可能与Governor自身的地址不同，例如可能是一个时间锁。这可以通过模块自定义，通过覆盖*_executor*函数来实现。执行者只能在治理程序的*执行*函数执行期间调用这些函数，而不能在其他任何情况下调用。因此，例如，附加的时间锁提议者不能在不经过治理协议的情况下更改治理参数（自v4.6起）。

#### constructor(string name_)
内部#
设置*name*和*version*的值

#### receive()
外部#
接收ETH的函数，将由治理者处理（如果执行者是第三方合约，则禁用）

#### supportsInterface(bytes4 interfaceId) → bool
公开#
请参见 *IERC165.supportsInterface*。

#### name() → string
公开#
请参见 *IGovernor.name*.

#### version() → string
公开#
请参见 *IGovernor.version*.

#### hashProposal(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
公开#
请参考 *IGovernor.hashProposal*。

提案ID是通过对ABI编码的targets数组、values数组、calldatas数组和descriptionHash（bytes32，它本身是描述字符串的keccak256哈希）进行哈希计算得到的。这个提案ID可以从*ProposalCreated*事件的提案数据中生成。甚至可以在提案提交之前提前计算出来。

请注意，chainId和governor地址不是提案ID计算的一部分。因此，如果在多个网络上的多个治理者提交相同的提案（具有相同的操作和相同的描述），则它们将具有相同的ID。这也意味着为了在同一个治理者上执行相同的操作两次，提案人必须更改描述以避免提案ID冲突。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
公开#
请参考 *IGovernor.state*.

#### proposalThreshold() → uint256
公开#
Governor Bravo界面的一部分：*“选民成为提案人所需的票数”*。

#### proposalSnapshot(uint256 proposalId) → uint256
公开#
请参阅*IGovernor.proposalSnapshot*。

#### proposalDeadline(uint256 proposalId) → uint256
公开#
请参阅 *IGovernor.proposalDeadline*.

#### proposalProposer(uint256 proposalId) → address
公开#
返回创建给定提案的账户。

#### _quorumReached(uint256 proposalId) → bool
内部#
已经投票的数量超过了阈值限制。

#### _voteSucceeded(uint256 proposalId) → bool
内部#
提案成功与否

#### _getVotes(address account, uint256 timepoint, bytes params) → uint256
内部#
获取特定时间点的帐户的投票权重，用params描述的投票方式。

#### _countVote(uint256 proposalId, address account, uint8 support, uint256 weight, bytes params)
内部#

根据给定的支持、投票权重和投票参数，为提案ID的账户注册一次投票。

注意：支持是通用的，根据所使用的投票系统可以代表各种不同的含义。

#### _defaultParams() → bytes
内部#
默认情况下，castVote方法使用的附加编码参数（不包括它们）。

注意：应该由特定的实现来覆盖以使用适当的值，附加参数的含义在该实现的上下文中。

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256
公开#
查看 *IGovernor.propose* 函数。该函数包含 opt-in 前置交易保护，其描述在 *_isValidDescriptionForProposer* 中说明。

#### execute(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
公开#
请参阅 *IGovernor.execute*.

#### cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
公开#
请参阅 *IGovernor.cancel*.

#### _execute(uint256, address[] targets, uint256[] values, bytes[] calldatas, bytes32)
内部#
内部执行机制。可以被覆盖以实现不同的执行机制。

#### _beforeExecute(uint256, address[] targets, uint256[], bytes[] calldatas, bytes32)
内部#
在执行触发之前的钩子。

#### _afterExecute(uint256, address[], uint256[], bytes[], bytes32)
内部#
在执行被触发后的钩子。

#### _cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
内部#
内部取消机制：锁定提案计时器，防止其重新提交。将其标记为已取消，以便与已执行的提案区分。

发出*IGovernor.ProposalCanceled*事件。

#### getVotes(address account, uint256 timepoint) → uint256
公开#
请参阅 *IGovernor.getVotes*.

#### getVotesWithParams(address account, uint256 timepoint, bytes params) → uint256
公开#
请参阅 *IGovernor.getVotesWithParams*.

#### castVote(uint256 proposalId, uint8 support) → uint256
公开#
请参阅 *IGovernor.castVote*.

#### castVoteWithReason(uint256 proposalId, uint8 support, string reason) → uint256
公开#
请参阅 *IGovernor.castVoteWithReason*.

#### castVoteWithReasonAndParams(uint256 proposalId, uint8 support, string reason, bytes params) → uint256
公开#
请参阅 *IGovernor.castVoteWithReasonAndParams*.

#### castVoteBySig(uint256 proposalId, uint8 support, uint8 v, bytes32 r, bytes32 s) → uint256
公开#
请参阅 *IGovernor.castVoteBySig*.

#### castVoteWithReasonAndParamsBySig(uint256 proposalId, uint8 support, string reason, bytes params, uint8 v, bytes32 r, bytes32 s) → uint256
公开#
请参阅 *IGovernor.castVoteWithReasonAndParamsBySig*.

#### _castVote(uint256 proposalId, address account, uint8 support, string reason) → uint256
内部#

内部投票机制：检查投票是否待定，尚未投票，使用*IGovernor.getVotes*检索投票权重，并调用*_countVote*内部函数。使用_defaultParams()。

触发*IGovernor.VoteCast*事件。

#### _castVote(uint256 proposalId, address account, uint8 support, string reason, bytes params) → uint256
内部#
内部投票投票机制：检查投票是否挂起，是否尚未投票，使用*IGovernor.getVotes*检索投票权重，并调用*_countVote*内部函数。

发出*IGovernor.VoteCast*事件。

#### relay(address target, uint256 value, bytes data)
外部#
将交易或函数调用转发到任意目标。在使用时间锁定等合约作为治理执行者的情况下，可以在治理提案中调用此函数来恢复错误发送到治理合约的代币或以太币。请注意，如果执行者只是治理本身，则使用中继是多余的。

#### _executor() → address
内部#
governor执行行动的地址。将通过执行通过另一个合同（如时间锁）的模块来进行重载。

#### onERC721Received(address, address, uint256, bytes) → bytes4
公开#
请参阅  *IERC721Receiver.onERC721Received*.

####  onERC1155Received(address, address, uint256, uint256, bytes) → bytes4
公开#
请参阅 IERC1155Receiver.onERC1155Received.

#### onERC1155BatchReceived(address, address, uint256[], uint256[], bytes) → bytes4
公开#
请参阅 IERC1155Receiver.onERC1155BatchReceived.

#### _isValidDescriptionForProposer(address proposer, string description) → bool
内部#
检查提案人是否被授权提交具有给定描述的提案。

如果提案描述以#proposer=0x???结尾，其中0x???是以十六进制字符串（不区分大小写）表示的地址，则只有该地址被授权提交此提案。

这用于防止前置交易。通过在其提案末尾添加此模式，提案人可以确保没有其他地址可以提交相同的提案。攻击者必须删除或更改该部分，这将导致不同的提案ID。

如果描述不符合此模式，则不受限制，任何人都可以提交。这包括： - 如果0x???部分不是有效的十六进制字符串。 - 如果0x???部分是有效的十六进制字符串，但不包含确切的40个十六进制数字。 - 如果以预期的后缀后跟换行符或其他空格符号结尾。 - 如果以其他类似的后缀结尾，例如#other=abc。 - 如果没有以任何此类后缀结尾。

#### 模块