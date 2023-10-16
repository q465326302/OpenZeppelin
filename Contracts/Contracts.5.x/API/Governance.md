# Governance
这个目录包含了链上治理的基本元素。

## Governor
这个Governor合约的模块化系统允许部署类似于[Compound的Governor Alpha和Bravo的链上投票协议](https://compound.finance/docs/governance)，甚至更多，通过能够轻松定制协议的多个方面。

> TIP
对于引导式体验，可以使用合约向导来设置你的[Governor合约](https://wizard.openzeppelin.com/#governor)。
对于书面教程，可以查看我们的指南*《如何设置链上治理》*。

* *Governor*：这是一个包含所有逻辑和基本元素的核心合约。它是抽象的，需要选择下面的每一个模块，或者自定义的模块。

投票模块决定了投票权力的来源，有时也决定了法定人数。

* *GovernorVotes*：从*ERC20Votes*或者自v4.5的*ERC721Votes*代币中提取投票权重。

* *GovernorVotesQuorumFraction*：与GovernorVotes结合，将法定人数设定为总代币供应量的一部分。

计数模块决定了有效的投票选项。

* *GovernorCountingSimple*：简单的投票机制，有3个投票选项：反对、赞成和弃权。

Timelock扩展为治理决策的执行增加了延迟。工作流程需要在执行前有一个队列步骤。使用这些模块，提案由外部的timelock合约执行，因此，需要由timelock持有被治理的资产。

* *GovernorTimelockControl*：与*TimelockController*的一个实例连接。除了Governor本身，还允许多个提议者和执行者。

*GovernorTimelockCompound*：与Compound的[Timelock](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Timelock.sol)合约的一个实例连接。

其他扩展可以以多种方式定制行为或接口。

*GovernorStorage*：在链上存储提案的详细信息，并提供提案的枚举。这对于一些存储成本低于calldata的L2链可能很有用。

*GovernorSettings*：以一种可以通过治理提案更新的方式管理一些设置（投票延迟、投票期限和提案阈值），无需进行升级。

*GovernorPreventLateQuorum*：确保在达到法定人数后有一个最小的投票期限，作为对大选民的安全保护。

除了模块和扩展，核心合约还需要实现一些特定规格的虚拟函数：

* *votingDelay()*：从提案提交到投票权力固定和投票开始的延迟（在EIP-6372时钟中）。这可以用来在提案发布后强制用户购买代币，或者委托他们的投票。

* *votingPeriod()*：从提案开始到投票结束的延迟（在EIP-6372时钟中）。

* *quorum(uint256 timepoint)*：提案成功所需的法定人数。这个函数包括一个时间点参数（参见EIP-6372），所以法定人数可以随时间调整，例如，跟随一个代币的totalSupply。

> NOTE
Governor合约的函数不包括访问控制。如果你想限制访问，你应该通过重载特定的函数来添加这些检查。其中，*Governor._cancel*默认是内部的，如果需要这个函数，你将不得不自己暴露它（用正确的访问控制机制）。

## Core

### [IGovernor](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/IGovernor.sol)
```
import "@openzeppelin/contracts/governance/IGovernor.sol";
```

核心*调控器*的接口

**FUNCTIONS**
name()

version()

COUNTING_MODE()

hashProposal(targets, values, calldatas, descriptionHash)

state(proposalId)

proposalThreshold()

proposalSnapshot(proposalId)

proposalDeadline(proposalId)

proposalProposer(proposalId)

proposalEta(proposalId)

proposalNeedsQueuing(proposalId)

votingDelay()

votingPeriod()

quorum(timepoint)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

hasVoted(proposalId, account)

propose(targets, values, calldatas, description)

queue(targets, values, calldatas, descriptionHash)

execute(targets, values, calldatas, descriptionHash)

cancel(targets, values, calldatas, descriptionHash)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

IERC6372
clock()

CLOCK_MODE()

IERC165
supportsInterface(interfaceId)

**EVENTS**
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

**ERRORS**
GovernorInvalidProposalLength(targets, calldatas, values)

GovernorAlreadyCastVote(voter)

GovernorDisabledDeposit()

GovernorOnlyProposer(account)

GovernorOnlyExecutor(account)

GovernorNonexistentProposal(proposalId)

GovernorUnexpectedProposalState(proposalId, current, expectedStates)

GovernorInvalidVotingPeriod(votingPeriod)

GovernorInsufficientProposerVotes(proposer, votes, threshold)

GovernorRestrictedProposer(proposer)

GovernorInvalidVoteType()

GovernorQueueNotImplemented()

GovernorNotQueuedProposal(proposalId)

GovernorAlreadyQueuedProposal(proposalId)

GovernorInvalidSignature(voter)

#### name() → string
external#
调控器实例的名称（用于构建ERC712域分隔符）。

#### version() → string
external#
构建ERC712域分隔符所使用的调控器实例的版本。默认值："1"

#### COUNTING_MODE() → string
external#
这是对*castVote*可能的支持值的描述，以及这些投票如何被计算，这是为了让UI显示正确的投票选项并解释结果。这个字符串是一个URL编码的键值对序列，每个键值对描述一个方面，例如support=bravo&quorum=for,abstain。

有两个标准键：support和quorum。

* support=bravo指的是投票选项0 = 反对，1 = 赞成，2 = 弃权，如同GovernorBravo。

* quorum=bravo意味着只有赞成票被计算在法定人数内。

* quorum=for,abstain意味着赞成和弃权票都被计算在法定人数内。

如果一个计数模块使用了编码参数，它应该在一个带有描述行为的唯一名称的params键下包含这个。例如：

* params=fractional可能指的是一个方案，其中投票被分数地分配给赞成/反对/弃权。

* params=erc721可能指的是一个方案，其中特定的NFT被委托投票。

> NOTE
这个字符串可以通过标准的[URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) JavaScript类进行解码。

#### hashProposal(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
external#
用于从提案详情中（重新）构建提案ID的哈希函数。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
external#
提案的当前状态，遵循Compound的惯例

#### proposalThreshold() → uint256
external#
需要多少票才能成为提案人。

#### proposalSnapshot(uint256 proposalId) → uint256
external#
用于检索用户投票和法定人数的时间点。如果使用区块号（如Compound的Comp），则在此区块结束时进行快照。因此，对此提案的投票将在下一个区块开始时开始。

#### proposalDeadline(uint256 proposalId) → uint256
external#
投票结束的时间点。如果使用区块编号，投票将在此区块结束时关闭，因此在此区块期间可以进行投票。

#### proposalProposer(uint256 proposalId) → address
external#
创建提案的账户。

#### proposalEta(uint256 proposalId) → uint256
external#
排队提案变为可执行的时间（"ETA"）。与proposalSnapshot和proposalDeadline不同，这并不使用州长的时钟，而是依赖执行者的时钟，这可能会有所不同。在大多数情况下，这将是一个时间戳。

#### proposalNeedsQueuing(uint256 proposalId) → bool
external#
一个提案在执行前是否需要排队。

#### votingDelay() → uint256
external#
延迟，即从提案创建到投票开始之间的时间。这个持续时间的单位取决于此合约使用的时钟（参见EIP-6372）。

这个可以增加，以便在提案的投票开始之前，用户有时间购买投票权或委托投票权。

> NOTE
虽然这个接口返回一个uint256，但时间点是以ERC-6372时钟类型的uint48存储的。因此，这个值必须适合在一个uint48中（当加上当前的时钟时）。参见*IERC6372.clock*。

#### votingPeriod() → uint256
external#
投票开始和投票结束之间的延迟。这个持续时间的单位取决于该合约使用的时钟（参见EIP-6372）。

> NOTE
*votingDelay*可以延迟投票的开始。在设置投票持续时间与投票延迟时，必须考虑到这一点。

> NOTE
这个值在提交提案时被存储，以便可能的值变化不影响已经提交的提案。用来保存它的类型是uint32。因此，虽然这个接口返回一个uint256，但它返回的值应该适合在一个uint32中。

#### quorum(uint256 timepoint) → uint256
external#
所需的最少投票数以使提案成功。

> NOTE
时间点参数对应于用于计数投票的快照。这允许根据此时间点的值（如此时的代币总供应量，参见*ERC20Votes*）来调整法定人数。

#### getVotes(address account, uint256 timepoint) → uint256
external#
在特定时间点的账户投票权力。

注意：这可以通过多种方式实现，例如通过从一个（或多个）*ERC20Votes*代币中读取委托余额。

#### getVotesWithParams(address account, uint256 timepoint, bytes params) → uint256
external#
在特定时间点，考虑额外编码参数后的账户投票权力。

#### hasVoted(uint256 proposalId, address account) → bool
external#

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256 proposalId
external#
创建一个新的提案。投票开始在*IGovernor.votingDelay*指定的延迟后，并持续*IGovernor.votingPeriod*指定的时间。

发出一个*ProposalCreated*事件。

#### queue(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256 proposalId
external#
提交一个提案。一些州长要求在执行之前必须进行此步骤。如果不需要排队，此功能可能会恢复。排队提案需要达到法定人数，投票成功，并达到截止日期。

发出一个提案已排队的事件。

#### execute(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256 proposalId
external#
执行成功的提案。这需要达到法定人数，投票成功，并且达到截止日期。根据管理者的不同，可能还需要提案已经排队并且过了一段延迟时间。

发出一个*ProposalExecuted*事件。

> NOTE
一些模块可以修改执行的要求，例如通过添加额外的时间锁。

#### cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256 proposalId
external#
取消提案。提案可以由提案者取消，但只有在处于待定状态，即投票开始前。

将触发一个*ProposalCanceled*事件。

#### castVote(uint256 proposalId, uint8 support) → uint256 balance
external#
投票

发出一个*VoteCast*事件。

#### castVoteWithReason(uint256 proposalId, uint8 support, string reason) → uint256 balance
external#
投出一票并附带理由

发出一个*VoteCast*事件。

#### castVoteWithReasonAndParams(uint256 proposalId, uint8 support, string reason, bytes params) → uint256 balance
external#
投票并附带理由和额外编码参数

根据params的长度，发出*VoteCast*或*VoteCastWithParams*事件。

#### castVoteBySig(uint256 proposalId, uint8 support, address voter, bytes signature) → uint256 balance
external#
使用选民的签名投票，包括支持ERC-1271签名。

发出一个*VoteCast*事件。

#### castVoteWithReasonAndParamsBySig(uint256 proposalId, uint8 support, address voter, string reason, bytes params, bytes signature) → uint256 balance
external#
使用选民的签名（包括ERC-1271签名支持）投票，并附带理由和额外的编码参数。

根据params的长度，发出*VoteCast*或*VoteCastWithParams*事件。

#### ProposalCreated(uint256 proposalId, address proposer, address[] targets, uint256[] values, string[] signatures, bytes[] calldatas, uint256 voteStart, uint256 voteEnd, string description)
event#
当提案被创建时发出。

#### ProposalQueued(uint256 proposalId, uint256 etaSeconds)
event#
当提案排队时发出。

#### ProposalExecuted(uint256 proposalId)
event#
当提案被执行时发出。

#### ProposalCanceled(uint256 proposalId)
event#
当提案被取消时发出。

#### VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason)
event#
在未使用参数投票时发出。

注意：支持值应被视为存储桶。它们的解释取决于使用的投票模块。

#### VoteCastWithParams(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason, bytes params)
event#
当投票时发出，带有参数。

注意：支持值应被视为存储桶。它们的解释取决于使用的投票模块。参数是额外的编码参数。它们的解释也取决于使用的投票模块。

#### GovernorInvalidProposalLength(uint256 targets, uint256 calldatas, uint256 values)
error#
空的提案或提案调用的参数长度不匹配。

#### GovernorAlreadyCastVote(address voter)
error#
投票已经完成。

#### GovernorDisabledDeposit()
error#
此合约已禁用代币存款。

#### GovernorOnlyProposer(address account)
error# 
该账户不是提议者。

#### GovernorOnlyExecutor(address account)
error#
该账户不是治理执行者。

#### GovernorNonexistentProposal(uint256 proposalId)
error#
提案ID不存在。

#### GovernorUnexpectedProposalState(uint256 proposalId, enum IGovernor.ProposalState current, bytes32 expectedStates)
error#
一个提案的当前状态不适合执行操作。expectedStates是一个位图，从右到左为每个ProposalState枚举位置启用位。

> NOTE
如果expectedState是bytes32(0)，则预期提案不处于任何状态（即不存在）。这是预期未设置的提案已经启动（提案被复制）的情况。

请参阅*Governor._encodeStateBitmap*。

#### GovernorInvalidVotingPeriod(uint256 votingPeriod)
error#
设置的投票期限不是有效的期限。

#### GovernorInsufficientProposerVotes(address proposer, uint256 votes, uint256 threshold)
error#
提案人没有获得创建提案所需的投票。

#### GovernorRestrictedProposer(address proposer)
error#
提案人不允许创建提案。

#### GovernorInvalidVoteType()
error#
所使用的投票类型对应的计数模块无效。

#### GovernorQueueNotImplemented()
error#
此管理器未实现队列操作。应直接调用执行。

#### GovernorNotQueuedProposal(uint256 proposalId)
error#
该提案尚未排队。

#### GovernorAlreadyQueuedProposal(uint256 proposalId)
error#
提案已经进入队列。

#### GovernorInvalidSignature(address voter)
error#
提供的签名对预期的投票者无效。如果投票者是合同，那么使用IERC1271.isValidSignature的签名无效。