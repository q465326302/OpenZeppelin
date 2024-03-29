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
排队提案变为可执行的时间（"ETA"）。与proposalSnapshot和proposalDeadline不同，这并不使用调控器的时钟，而是依赖执行者的时钟，这可能会有所不同。在大多数情况下，这将是一个时间戳。

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
提交一个提案。一些调控器要求在执行之前必须进行此步骤。如果不需要排队，此功能可能会恢复。排队提案需要达到法定人数，投票成功，并达到截止日期。

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
提供的签名对预期的投票者无效。如果投票者是合约，那么使用IERC1271.isValidSignature的签名无效。

### [Governor](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/Governor.sol)
```
import "@openzeppelin/contracts/governance/Governor.sol";
```

这个合约是治理系统的核心，设计为通过各种模块进行扩展。

这个合约是抽象的，需要在各种模块中实现几个函数：

* 一个计数模块必须实现*quorum*，*_quorumReached*，*_voteSucceeded*和*_countVote*

* 一个投票模块必须实现*_getVotes*

* 此外，还必须实现*votingPeriod*。

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

proposalEta(proposalId)

proposalNeedsQueuing()

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_getVotes(account, timepoint, params)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

_queueOperations(, , , , )

execute(targets, values, calldatas, descriptionHash)

_executeOperations(, targets, values, calldatas, )

cancel(targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

_executor()

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

clock()

CLOCK_MODE()

votingDelay()

votingPeriod()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### onlyGovernance()
modifier#
限制一个函数，使其只能通过治理提案来执行。例如，*GovernorSettings*中的治理参数设置器就是使用这个修饰符进行保护的。

执行治理的地址可能与Governor自身的地址不同，例如，它可能是一个时间锁。模块可以通过覆盖*_executor*来自定义这一点。执行器只能在执行governor的*execute*函数期间调用这些函数，而不能在任何其他情况下调用。因此，例如，额外的时间锁提议者不能在不通过治理协议的情况下改变治理参数（自v4.6起）。

#### constructor(string name_)
internal#
设置名称和版本的值

#### receive()
external#
接收ETH的函数将由管理员处理（如果执行者是第三方合约，则禁用）

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查阅 *IERC165.supportsInterface*.

#### name() → string
public#
请查阅 *IGovernor.name*.

#### version() → string
public#
请查阅 IGovernor.version.

#### hashProposal(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
查看*IGovernor.hashProposal*。

提案id是通过哈希ABI编码的目标数组、值数组、calldatas数组和descriptionHash（bytes32，它本身是描述字符串的keccak256哈希）生成的。这个提案id可以从*ProposalCreated*事件的提案数据中产生。它甚至可以在提案提交之前预先计算出来。

注意，链id和调控器地址并不是提案id计算的一部分。因此，如果在多个网络上的多个调控器提交相同的提案（具有相同的操作和相同的描述），那么这个提案将具有相同的id。这也意味着，为了执行相同的操作两次（在同一个调控器上），提议者将不得不改变描述，以避免提案id冲突。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
public#
请查阅 *IGovernor.state*.

#### proposalThreshold() → uint256
public#
请查阅 *IGovernor.proposalThreshold*.

#### proposalSnapshot(uint256 proposalId) → uint256
public#
请查阅 *IGovernor.proposalSnapshot*.

#### proposalDeadline(uint256 proposalId) → uint256
public#
请查阅 *IGovernor.proposalDeadline*.

#### proposalProposer(uint256 proposalId) → address
public#
请查阅 *IGovernor.proposalProposer*.

#### proposalEta(uint256 proposalId) → uint256
public#
请查阅 *IGovernor.proposalEta*.

#### proposalNeedsQueuing(uint256) → bool
public#
请查阅 *IGovernor.proposalNeedsQueuing*.

#### _checkGovernance()
internal#
如果msg.sender不是执行者，则撤销。如果执行者不是这个合约本身，那么如果msg.data由于*执行*操作而没有被列入白名单，函数就会撤销。参见*onlyGovernance*。

#### _quorumReached(uint256 proposalId) → bool
internal#
已经投出的选票数量超过了阈值限制。

#### _voteSucceeded(uint256 proposalId) → bool
internal#
提案是否成功。

#### _getVotes(address account, uint256 timepoint, bytes params) → uint256
internal#
获取特定时间点上，根据参数描述的投票的账户投票权重。

#### _countVote(uint256 proposalId, address account, uint8 support, uint256 weight, bytes params)
internal#
为proposalId注册一个投票，该投票由具有给定支持、投票权重和投票参数的账户进行。

注意：支持是通用的，可以根据使用的投票系统表示各种不同的事物。

#### _defaultParams() → bytes
internal#
默认的由castVote方法使用但不包含的额外编码参数

注意：应由特定实现来覆盖以使用适当的值，这些额外的参数在该实现的上下文中的含义。

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256
public#
请查阅*IGovernor.propose*。这个函数有可选择的前运行保护，详细描述在*_isValidDescriptionForProposer*中。

#### _propose(address[] targets, uint256[] values, bytes[] calldatas, string description, address proposer) → uint256 proposalId
internal#
内部提议机制。可以被覆盖以在提案创建时添加更多逻辑。

触发一个*IGovernor.ProposalCreated*事件。

#### queue(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
请查阅 *IGovernor.queue*.

#### _queueOperations(uint256, address[], uint256[], bytes[], bytes32) → uint48
internal#
内部队列机制。可以被覆盖（不需要超级调用）以修改队列的执行方式（例如添加一个保险库/定时锁）。

默认情况下，这是空的，必须被覆盖以实现队列。

这个函数返回一个描述预期执行ETA的时间戳。如果返回值是0（这是默认值），核心将认为队列没有成功，公共*队列*函数将回滚。

> NOTE
直接调用这个函数将不会检查提案的当前状态，或发出ProposalQueued事件。应使用*队列*来排队提案。

#### execute(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
请查阅 *IGovernor.execute*.

#### _executeOperations(uint256, address[] targets, uint256[] values, bytes[] calldatas, bytes32)
internal#
内部执行机制。可以被覆盖（无需调用super）以修改执行方式（例如添加保险库/时间锁）。

直接调用此函数将不会检查提案的当前状态，将执行标志设置为真或发出ProposalExecuted事件。执行提案应使用*execute*或{_execute}。

#### cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
请查阅 IGovernor.cancel.

#### _cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
internal#
具有最小限制的内部取消机制。提案可以在除已取消、已过期或已执行的状态之外的任何状态下被取消。一旦取消，提案不能重新提交。

会触发一个*IGovernor.ProposalCanceled*事件。

#### getVotes(address account, uint256 timepoint) → uint256
public#
请查阅 *IGovernor.getVotes*.

#### getVotesWithParams(address account, uint256 timepoint, bytes params) → uint256
public#
请查阅 *IGovernor.getVotesWithParams*.

#### castVote(uint256 proposalId, uint8 support) → uint256
public#
请查阅 *IGovernor.castVote*.

#### castVoteWithReason(uint256 proposalId, uint8 support, string reason) → uint256
public#
请查阅 *IGovernor.castVoteWithReason*.

#### castVoteWithReasonAndParams(uint256 proposalId, uint8 support, string reason, bytes params) → uint256
public#
请查阅 *IGovernor.castVoteWithReasonAndParams*

#### castVoteBySig(uint256 proposalId, uint8 support, address voter, bytes signature) → uint256
public#
请查阅 *IGovernor.castVoteBySig*.

#### castVoteWithReasonAndParamsBySig(uint256 proposalId, uint8 support, address voter, string reason, bytes params, bytes signature) → uint256
public#
请查阅 *IGovernor.castVoteWithReasonAndParamsBySig*.

#### _castVote(uint256 proposalId, address account, uint8 support, string reason) → uint256
internal#
内部投票机制：检查投票是否待决，是否还未投出，使用*IGovernor.getVotes*检索投票权重，并调用*_countVote*内部函数。使用_defaultParams()。

触发一个*IGovernor.VoteCast*事件。

#### _castVote(uint256 proposalId, address account, uint8 support, string reason, bytes params) → uint256
internal#
内部投票机制：检查投票是否待决，是否还未投出，使用*IGovernor.getVotes*获取投票权重，并调用*_countVote*内部函数。

触发一个*IGovernor.VoteCast*事件。

#### relay(address target, uint256 value, bytes data)
external#
将交易或函数调用转发到任意目标。在治理执行者是除了调控器本身之外的一些合约的情况下，例如在使用时间锁时，可以在治理提案中调用此功能来恢复错误地发送到调控器合约的代币或以太币。请注意，如果执行者只是调控器本身，那么使用中继是多余的。

#### _executor() → address
internal#
通过该地址，调控器执行操作。将被通过其他合约执行操作的模块重载，例如时间锁。

#### onERC721Received(address, address, uint256, bytes) → bytes4
public#
请参阅*IERC721Receiver.onERC721Received*。如果治理执行者不是治理者本身（例如，使用时间锁时），则接收代币将被禁用。

#### onERC1155Received(address, address, uint256, uint256, bytes) → bytes4
public#
请参阅*IERC1155Receiver.onERC1155Received*。如果治理执行者不是治理者本身（例如，使用时间锁时），则会禁用接收令牌。

#### _encodeStateBitmap(enum IGovernor.ProposalState proposalState) → bytes32
internal#
将ProposalState编码为一个bytes32表示，其中每个启用的位对应于ProposalState枚举中的底层位置。例如：

0x000…​10000 ^^------ …​^----- 成功 ^---- 被击败 ^--- 取消 ^-- 活动 ^- 待处理

#### _isValidDescriptionForProposer(address proposer, string description) → bool
internal#

#### clock() → uint48
public#
用于标记检查点的时钟。可以被覆盖以实现基于时间戳的检查点（和投票）。

#### CLOCK_MODE() → string
public#
时间锁的描述

#### votingDelay() → uint256
public#
延迟，从提案创建到投票开始之间的时间。这个持续时间的单位取决于这个合约使用的时钟（参见EIP-6372）。

这个可以增加，以便在提案的投票开始之前，用户有时间购买投票权，或者委托它。

> NOTE
虽然这个接口返回一个uint256，但是时间点是以ERC-6372时钟类型的uint48存储的。因此，这个值必须适合在一个uint48中（当加到当前时钟时）。参见*IERC6372.clock*。

#### votingPeriod() → uint256
public#
投票开始和投票结束之间的延迟。这个持续时间的单位取决于该合约使用的时钟（参见EIP-6372）。

> NOTE
*votingDelay*可以延迟投票的开始。在设置投票持续时间与投票延迟时，必须考虑到这一点。

> NOTE
当提案被提交时，这个值会被存储，以便可能的值变化不会影响已经提交的提案。用来保存它的类型是uint32。因此，虽然这个接口返回一个uint256，但它返回的值应该适合在uint32中。

#### quorum(uint256 timepoint) → uint256
public#
所需的最小投票数以使提案成功。

时间点参数对应于用于计数投票的快照。这允许根据此时间点的值（如此时点的代币总供应量，参见*ERC20Votes*）来调整法定人数。

#### BALLOT_TYPEHASH() → bytes32
public#

#### EXTENDED_BALLOT_TYPEHASH() → bytes32
public#

### [Modules](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorCountingSimple.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";
```

为简单起见，对调控器的任期延长进行投票，有三个选项，进行投票计数。

**FUNCTIONS**
COUNTING_MODE()

hasVoted(proposalId, account)

proposalVotes(proposalId)

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_countVote(proposalId, account, support, weight, )

GOVERNOR
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

proposalEta(proposalId)

proposalNeedsQueuing()

_checkGovernance()

_getVotes(account, timepoint, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

_queueOperations(, , , , )

execute(targets, values, calldatas, descriptionHash)

_executeOperations(, targets, values, calldatas, )

cancel(targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

_executor()

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

clock()

CLOCK_MODE()

votingDelay()

votingPeriod()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### COUNTING_MODE() → string
public#
请查阅 *IGovernor.COUNTING_MODE*.

#### hasVoted(uint256 proposalId, address account) → bool
public#
请查阅 *IGovernor.hasVoted*.

#### proposalVotes(uint256 proposalId) → uint256 againstVotes, uint256 forVotes, uint256 abstainVotes
public#
访问内部投票计数的访问器。

#### _quorumReached(uint256 proposalId) → bool
internal#
请查阅 Governor._quorumReached.

#### _voteSucceeded(uint256 proposalId) → bool
internal#
在*Governor._voteSucceeded*模块中，赞成票必须严格超过反对票。

#### _countVote(uint256 proposalId, address account, uint8 support, uint256 weight, bytes)
internal#
在这个模块中，支持遵循Governor Bravo的VoteType枚举。请查看*Governor._countVote*。

### [GovernorVotes](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorVotes.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";
```

扩展投票权重提取的调控器，从ERC20Votes代币，或者从v4.5开始的ERC721Votes代币。

**FUNCTIONS**
constructor(tokenAddress)

token()

clock()

CLOCK_MODE()

_getVotes(account, timepoint, )

GOVERNOR
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

proposalEta(proposalId)

proposalNeedsQueuing()

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

_queueOperations(, , , , )

execute(targets, values, calldatas, descriptionHash)

_executeOperations(, targets, values, calldatas, )

cancel(targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

_executor()

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

votingDelay()

votingPeriod()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### constructor(contract IVotes tokenAddress)
internal#

#### token() → contract IERC5805
public#
投票权来源的代币。

#### clock() → uint48
public#
（根据EIP-6372规定的）时钟设定为与代币的时钟匹配。如果代币未实现EIP-6372，则回退到区块编号。

#### CLOCK_MODE() → string
public#
EIP-6372中指定的时钟的机器可读描述。

#### _getVotes(address account, uint256 timepoint, bytes) → uint256
internal#

### [GovernorVotesQuorumFraction](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorVotesQuorumFraction.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorVotesQuorumFraction.sol";
```
扩展投票权重从*ERC20Votes*令牌中提取的*调控器*，并以总供应量的一部分表示的法定人数。

**FUNCTIONS**
constructor(quorumNumeratorValue)

quorumNumerator()

quorumNumerator(timepoint)

quorumDenominator()

quorum(timepoint)

updateQuorumNumerator(newQuorumNumerator)

_updateQuorumNumerator(newQuorumNumerator)

GOVERNORVOTES
token()

clock()

CLOCK_MODE()

_getVotes(account, timepoint, )

GOVERNOR
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

proposalEta(proposalId)

proposalNeedsQueuing()

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

_queueOperations(, , , , )

execute(targets, values, calldatas, descriptionHash)

_executeOperations(, targets, values, calldatas, )

cancel(targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

_executor()

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

votingDelay()

votingPeriod()

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
QuorumNumeratorUpdated(oldQuorumNumerator, newQuorumNumerator)

IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
GovernorInvalidQuorumFraction(quorumNumerator, quorumDenominator)

IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### constructor(uint256 quorumNumeratorValue)
internal#
将法定人数初始化为代币总供应量的一部分。

该比例被指定为分子/分母。默认情况下，分母为100，所以法定人数被指定为百分比：分子为10对应于法定人数为总供应量的10%。可以通过覆盖*quorumDenominator*来自定义分母。

#### quorumNumerator() → uint256
public#
返回当前的法定人数分子。参见*quorumDenominator*。

#### quorumNumerator(uint256 timepoint) → uint256
public#
在特定时间点返回法定人数的分子。参见*quorumDenominator*。

#### quorumDenominator() → uint256
public#
返回法定人数的分母。默认为100，但可以被覆盖。

#### quorum(uint256 timepoint) → uint256
public#
返回一个时间点的法定人数，以投票数表示：供应量*分子/分母。

#### updateQuorumNumerator(uint256 newQuorumNumerator)
external#
更改法定人数的分子。

触发一个*QuorumNumeratorUpdated*事件。

要求：
* 必须通过治理提案来调用。

* 新的分子必须小于或等于分母。

#### _updateQuorumNumerator(uint256 newQuorumNumerator)
internal#
更改法定人数的分子。

发出一个*QuorumNumeratorUpdated*事件。

要求：
* 新的分子必须小于或等于分母。

#### QuorumNumeratorUpdated(uint256 oldQuorumNumerator, uint256 newQuorumNumerator)
event#

#### GovernorInvalidQuorumFraction(uint256 quorumNumerator, uint256 quorumDenominator)
error#
法定人数集合不是一个有效的比例。

### Extensions

### [GovernorTimelockControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorTimelockControl.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";
```

这是一个扩展*Governor*的模块，它将执行过程绑定到*TimelockController*的一个实例。这将为所有成功的提案（除了投票期限）增加一个由*TimelockController*强制执行的延迟。为了使*Governor*正常工作，需要提案者（理想情况下还需要执行者）的角色一起配合。

使用这种模型意味着提案将由*TimelockController*操作，而不是由*Governor*操作。因此，资产和权限必须附加到*TimelockController*。任何发送到*Governor*的资产，除非通过*Governor.relay*执行，否则在提案中将无法访问。

> WARNING
将TimelockController设置为除governor之外的额外提案者或取消者是非常危险的，因为这将授予他们以下能力：1) 作为timelock执行操作，因此可能执行操作或访问只有通过投票才能访问的资金，2) 阻止已经得到选民批准的治理提案，有效地执行拒绝服务攻击。

> NOTE
AccessManager不支持同时调度具有相同目标和calldata的多个操作。请参阅*AccessManager.schedule*以获取解决方法。

**FUNCTIONS**
constructor(timelockAddress)

state(proposalId)

timelock()

proposalNeedsQueuing()

_queueOperations(proposalId, targets, values, calldatas, descriptionHash)

_executeOperations(proposalId, targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

_executor()

updateTimelock(newTimelock)

GOVERNOR
receive()

supportsInterface(interfaceId)

name()

version()

hashProposal(targets, values, calldatas, descriptionHash)

proposalThreshold()

proposalSnapshot(proposalId)

proposalDeadline(proposalId)

proposalProposer(proposalId)

proposalEta(proposalId)

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_getVotes(account, timepoint, params)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

execute(targets, values, calldatas, descriptionHash)

cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

clock()

CLOCK_MODE()

votingDelay()

votingPeriod()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
TimelockChange(oldTimelock, newTimelock)

IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### constructor(contract TimelockController timelockAddress)
internal#
设置时间锁。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
public#
覆盖版本的*Governor.state*函数，该函数考虑由时间锁报告的状态。

#### timelock() → address
public#
公共访问器用于检查时间锁的地址

#### proposalNeedsQueuing(uint256) → bool
public#
请查阅 *IGovernor.proposalNeedsQueuing*.

#### _queueOperations(uint256 proposalId, address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint48
internal#
将提案排队到时间锁的函数。

#### _executeOperations(uint256 proposalId, address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash)
internal#
已重写的*Governor._executeOperations*函数，该函数通过时间锁运行已排队的提案。

#### _cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
internal#
覆盖*Governor._cancel*函数的版本，用于取消已经排队的定时锁定提案。

#### _executor() → address
internal#
执行行动的地址，即在这种情况下的时间锁。

#### updateTimelock(contract TimelockController newTimelock)
external#
更新底层时间锁实例的公共端点。仅限于时间锁本身，因此必须通过治理提案提出、安排和执行更新。

在有其他排队的治理提案时，不建议更改时间锁。

#### TimelockChange(address oldTimelock, address newTimelock)
event#
当用于提案执行的时间锁控制器被修改时发出。

### [GovernorTimelockCompound](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorTimelockCompound.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorTimelockCompound.sol";
```

这是一个扩展*Governor*的程序，它将执行过程绑定到一个Compound Timelock。这将在所有成功的提案上添加一个由外部timelock强制执行的延迟（除了投票期限）。*Governor*需要是timelock的管理员才能执行任何操作。一个公开的，不受限制的，*GovernorTimelockCompound.acceptAdmin*可用于接受timelock的所有权。

使用这种模型意味着提案将由*TimelockController*操作，而不是由*Governor*操作。因此，资产和权限必须附加到*TimelockController*。发送给*Governor*的任何资产都将无法访问。

**FUNCTIONS**
constructor(timelockAddress)

state(proposalId)

timelock()

proposalNeedsQueuing()

_queueOperations(proposalId, targets, values, calldatas, )

_executeOperations(proposalId, targets, values, calldatas, )

_cancel(targets, values, calldatas, descriptionHash)

_executor()

__acceptAdmin()

updateTimelock(newTimelock)

GOVERNOR
receive()

supportsInterface(interfaceId)

name()

version()

hashProposal(targets, values, calldatas, descriptionHash)

proposalThreshold()

proposalSnapshot(proposalId)

proposalDeadline(proposalId)

proposalProposer(proposalId)

proposalEta(proposalId)

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_getVotes(account, timepoint, params)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

execute(targets, values, calldatas, descriptionHash)

cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

clock()

CLOCK_MODE()

votingDelay()

votingPeriod()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
TimelockChange(oldTimelock, newTimelock)

IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### constructor(contract ICompoundTimelock timelockAddress)
internal#
设置时间锁。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
public#
覆盖版本的*Governor.state*函数，增加了对过期状态的支持。

#### timelock() → address
public#
公共访问器用于检查时间锁的地址

#### proposalNeedsQueuing(uint256) → bool
public#
请查阅 IGovernor.proposalNeedsQueuing.

#### _queueOperations(uint256 proposalId, address[] targets, uint256[] values, bytes[] calldatas, bytes32) → uint48
internal#
将提案排队到时间锁的函数。

#### _executeOperations(uint256 proposalId, address[] targets, uint256[] values, bytes[] calldatas, bytes32)
internal#
已重写的Governor._executeOperations函数，该函数通过时间锁运行已排队的提案。

#### _cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
internal#
覆盖*Governor._cancel*函数的版本，用于取消已经排队的时间锁定提案。

#### _executor() → address
internal#
调控器通过该地址执行操作。在这种情况下，是时间锁。

#### __acceptAdmin()
public#
接受对时间锁的管理员权限。

#### updateTimelock(contract ICompoundTimelock newTimelock)
external#
公共端点用于更新底层的时间锁实例。这个操作仅限于时间锁本身，因此更新必须通过治理提案进行提出、安排和执行。

出于安全考虑，在设置新的时间锁之前，必须将时间锁交给另一个管理员。这两个操作（交接时间锁和进行更新）可以在一个提案中进行批处理。

请注意，如果时间锁管理员在之前的操作中已经被交接，我们会拒绝通过时间锁进行的更新，如果时间锁的管理员已经被接受，并且操作是在治理范围之外执行的。

> CAUTION
在有其他排队的治理提案时，不建议更改时间锁。

#### TimelockChange(address oldTimelock, address newTimelock)
event#
当用于提案执行的时间锁控制器被修改时发出。

### [GovernorSettings](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorSettings.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";
```

通过治理进行设置更新的调控器扩展。

**FUNCTIONS**
constructor(initialVotingDelay, initialVotingPeriod, initialProposalThreshold)

votingDelay()

votingPeriod()

proposalThreshold()

setVotingDelay(newVotingDelay)

setVotingPeriod(newVotingPeriod)

setProposalThreshold(newProposalThreshold)

_setVotingDelay(newVotingDelay)

_setVotingPeriod(newVotingPeriod)

_setProposalThreshold(newProposalThreshold)

GOVERNOR
receive()

supportsInterface(interfaceId)

name()

version()

hashProposal(targets, values, calldatas, descriptionHash)

state(proposalId)

proposalSnapshot(proposalId)

proposalDeadline(proposalId)

proposalProposer(proposalId)

proposalEta(proposalId)

proposalNeedsQueuing()

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_getVotes(account, timepoint, params)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

_queueOperations(, , , , )

execute(targets, values, calldatas, descriptionHash)

_executeOperations(, targets, values, calldatas, )

cancel(targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

_executor()

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

clock()

CLOCK_MODE()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
VotingDelaySet(oldVotingDelay, newVotingDelay)

VotingPeriodSet(oldVotingPeriod, newVotingPeriod)

ProposalThresholdSet(oldProposalThreshold, newProposalThreshold)

IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### constructor(uint48 initialVotingDelay, uint32 initialVotingPeriod, uint256 initialProposalThreshold)
internal#
初始化治理参数。

#### votingDelay() → uint256
public#
请查阅 *IGovernor.votingDelay*.

#### votingPeriod() → uint256
public#
请查阅 *IGovernor.votingPeriod*.

#### proposalThreshold() → uint256
public#
请查阅 *Governor.proposalThreshold*.

#### setVotingDelay(uint48 newVotingDelay)
public#
更新投票延迟。此操作只能通过治理提案进行。

触发一个*VotingDelaySet*事件。

#### setVotingPeriod(uint32 newVotingPeriod)
public#
更新投票期限。此操作只能通过治理提案进行。

发出一个*VotingPeriodSet*事件。

#### setProposalThreshold(uint256 newProposalThreshold)
public#
更新提案阈值。此操作只能通过治理提案执行。

发出一个*ProposalThresholdSet*事件。

#### _setVotingDelay(uint48 newVotingDelay)
internal#
内部设定投票延迟。

发出一个*VotingDelaySet*事件。

#### _setVotingPeriod(uint32 newVotingPeriod)
internal#
为投票期设置内部设定器。

发出一个*VotingPeriodSet*事件。

#### _setProposalThreshold(uint256 newProposalThreshold)
internal#
为提案阈值设置内部设定器。

发出一个*ProposalThresholdSet*事件。

#### VotingDelaySet(uint256 oldVotingDelay, uint256 newVotingDelay)
event#

#### VotingPeriodSet(uint256 oldVotingPeriod, uint256 newVotingPeriod)
event#

#### ProposalThresholdSet(uint256 oldProposalThreshold, uint256 newProposalThreshold)
event#

### [GovernorPreventLateQuorum](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorPreventLateQuorum.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorPreventLateQuorum.sol";
```

一个模块，确保在达到法定人数后有最短的投票期。这防止了大选民在最后一刻影响投票并触发法定人数，因为它始终确保其他选民有时间反应并试图反对决定。

如果一次投票使得法定人数得以达到，那么提案的投票期可能会被延长，以确保在至少指定的时间（"投票延长"参数）过去之后才结束。这个参数可以通过治理提案来设定。

**FUNCTIONS**
constructor(initialVoteExtension)

proposalDeadline(proposalId)

_castVote(proposalId, account, support, reason, params)

lateQuorumVoteExtension()

setLateQuorumVoteExtension(newVoteExtension)

_setLateQuorumVoteExtension(newVoteExtension)

GOVERNOR
receive()

supportsInterface(interfaceId)

name()

version()

hashProposal(targets, values, calldatas, descriptionHash)

state(proposalId)

proposalThreshold()

proposalSnapshot(proposalId)

proposalProposer(proposalId)

proposalEta(proposalId)

proposalNeedsQueuing()

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_getVotes(account, timepoint, params)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

_propose(targets, values, calldatas, description, proposer)

queue(targets, values, calldatas, descriptionHash)

_queueOperations(, , , , )

execute(targets, values, calldatas, descriptionHash)

_executeOperations(, targets, values, calldatas, )

cancel(targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

relay(target, value, data)

_executor()

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

clock()

CLOCK_MODE()

votingDelay()

votingPeriod()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
ProposalExtended(proposalId, extendedDeadline)

LateQuorumVoteExtensionSet(oldVoteExtension, newVoteExtension)

IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### constructor(uint48 initialVoteExtension)
internal#
初始化投票扩展参数：自提案达到法定人数的那一刻起，需要经过的块数或秒数（取决于管理者时钟模式），直到其投票期结束。如果需要，投票期将超过在提案创建期间设置的时间。

#### proposalDeadline(uint256 proposalId) → uint256
public#
返回提案截止日期，如果提案在投票期间晚些时候达到法定人数，该日期可能已经超过了提案创建时设定的日期。请参阅*Governor.proposalDeadline*。

#### _castVote(uint256 proposalId, address account, uint8 support, string reason, bytes params) → uint256
internal#
投票并检测是否达到了法定人数，可能会延长投票期。请参阅*Governor._castVote*。

可能会发出*ProposalExtended*事件。

#### lateQuorumVoteExtension() → uint48
public#
返回投票扩展参数的当前值：从提案达到法定人数到其投票期结束所需的区块数量。

#### setLateQuorumVoteExtension(uint48 newVoteExtension)
public#
更改*lateQuorumVoteExtension*。此操作只能由治理执行者执行，通常通过治理提案进行。

触发一个*LateQuorumVoteExtensionSet*事件。

#### _setLateQuorumVoteExtension(uint48 newVoteExtension)
internal#
更改*lateQuorumVoteExtension*。这是一个内部函数，如果需要其他访问控制机制，可以在公共函数（如*setLateQuorumVoteExtension*）中公开。

发出一个*LateQuorumVoteExtensionSet*事件。

#### ProposalExtended(uint256 indexed proposalId, uint64 extendedDeadline)
event#
当在投票期限内晚达到法定人数而推迟提案截止日期时发出。

#### LateQuorumVoteExtensionSet(uint64 oldVoteExtension, uint64 newVoteExtension)
event#
当*lateQuorumVoteExtension*参数发生变化时发出。

### [GovernorStorage](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/extensions/GovernorStorage.sol)
```
import "@openzeppelin/contracts/governance/extensions/GovernorStorage.sol";
```

这个模块是*Governor*的扩展，实现了提案详情的存储。此模块还为提案的可枚举性提供了原语。

此模块的使用场景包括：- 不依赖事件索引就能探索提案状态的用户界面。- 在L2链中，只使用proposalId作为*Governor.queue*和*Governor.execute*函数的参数，因为在这些链中，存储的成本比calldata要便宜。

**FUNCTIONS**
_propose(targets, values, calldatas, description, proposer)

queue(proposalId)

execute(proposalId)

cancel(proposalId)

proposalCount()

proposalDetails(proposalId)

proposalDetailsAt(index)

GOVERNOR
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

proposalEta(proposalId)

proposalNeedsQueuing()

_checkGovernance()

_quorumReached(proposalId)

_voteSucceeded(proposalId)

_getVotes(account, timepoint, params)

_countVote(proposalId, account, support, weight, params)

_defaultParams()

propose(targets, values, calldatas, description)

queue(targets, values, calldatas, descriptionHash)

_queueOperations(, , , , )

execute(targets, values, calldatas, descriptionHash)

_executeOperations(, targets, values, calldatas, )

cancel(targets, values, calldatas, descriptionHash)

_cancel(targets, values, calldatas, descriptionHash)

getVotes(account, timepoint)

getVotesWithParams(account, timepoint, params)

castVote(proposalId, support)

castVoteWithReason(proposalId, support, reason)

castVoteWithReasonAndParams(proposalId, support, reason, params)

castVoteBySig(proposalId, support, voter, signature)

castVoteWithReasonAndParamsBySig(proposalId, support, voter, reason, params, signature)

_castVote(proposalId, account, support, reason)

_castVote(proposalId, account, support, reason, params)

relay(target, value, data)

_executor()

onERC721Received(, , , )

onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

_encodeStateBitmap(proposalState)

_isValidDescriptionForProposer(proposer, description)

clock()

CLOCK_MODE()

votingDelay()

votingPeriod()

quorum(timepoint)

BALLOT_TYPEHASH()

EXTENDED_BALLOT_TYPEHASH()

IGOVERNOR
COUNTING_MODE()

hasVoted(proposalId, account)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
IGOVERNOR
ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)

ProposalQueued(proposalId, etaSeconds)

ProposalExecuted(proposalId)

ProposalCanceled(proposalId)

VoteCast(voter, proposalId, support, weight, reason)

VoteCastWithParams(voter, proposalId, support, weight, reason, params)

IERC5267
EIP712DomainChanged()

**ERRORS**
IGOVERNOR
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

NONCES
InvalidAccountNonce(account, currentNonce)

#### _propose(address[] targets, uint256[] values, bytes[] calldatas, string description, address proposer) → uint256
internal#
连接到提案机制

#### queue(uint256 proposalId)
public#
{IGovernorTimelock-queue}版本，只有proposalId作为参数。

#### execute(uint256 proposalId)
public#
*IGovernor.execute*仅以proposalId作为参数的版本。

#### cancel(uint256 proposalId)
public#
*IGovernor.cancel*的ProposalId版本。

#### proposalCount() → uint256
public#
返回存储提案的数量。

#### proposalDetails(uint256 proposalId) → address[], uint256[], bytes[], bytes32
public#
返回提案Id的详细信息。如果提案Id不是已知的提案，则撤销。

#### proposalDetailsAt(uint256 index) → uint256, address[], uint256[], bytes[], bytes32
public#
返回给定序列索引的提案的详细信息（包括提案Id）。

## Utils

### [Votes](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/utils/Votes.sol)
```
import "@openzeppelin/contracts/governance/utils/Votes.sol";
```

这是一个基础抽象合约，用于跟踪投票单位，这是一种可以转移的投票权力度量，并提供了一个投票委托系统，其中一个账户可以将其投票单位委托给一种“代表”，该代表将从不同的账户中汇集委托的投票单位，然后可以使用它来参与决策投票。实际上，投票单位必须被委托才能计算为实际的投票，如果一个账户希望参与决策并且没有信任的代表，那么它必须将这些投票委托给自己。

这个合约通常与一个代币合约结合使用，使得投票单位对应于代币单位。有关示例，请参见*ERC721Votes*。

完整的代表投票历史在链上进行跟踪，以便治理协议可以考虑在特定区块号分配的投票，以防止闪电贷款和双重投票。可选的代表系统使得这种历史跟踪的成本变得可选。

在使用此模块时，派生合约必须实现*_getVotingUnits*（例如，使其返回*ERC721.balanceOf*），并可以使用*_transferVotingUnits*来跟踪这些单位分配的变化（在前面的例子中，它将包含在*ERC721._update*中）。

**FUNCTIONS**
clock()

CLOCK_MODE()

getVotes(account)

getPastVotes(account, timepoint)

getPastTotalSupply(timepoint)

_getTotalSupply()

delegates(account)

delegate(delegatee)

delegateBySig(delegatee, nonce, expiry, v, r, s)

_delegate(account, delegatee)

_transferVotingUnits(from, to, amount)

_numCheckpoints(account)

_checkpoints(account, pos)

_getVotingUnits()

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

**EVENTS**
IVOTES
DelegateChanged(delegator, fromDelegate, toDelegate)

DelegateVotesChanged(delegate, previousVotes, newVotes)

IERC5267
EIP712DomainChanged()

**ERRORS**
ERC6372InconsistentClock()

ERC5805FutureLookup(timepoint, clock)

IVOTES
VotesExpiredSignature(expiry)

NONCES
InvalidAccountNonce(account, currentNonce)

#### clock() → uint48
public#
用于标记检查点的时钟。可以被覆盖以实现基于时间戳的检查点（和投票），在这种情况下，*CLOCK_MODE*也应该被覆盖以匹配。

#### CLOCK_MODE() → string
public#
EIP-6372中指定的时钟的机器可读描述。

#### getVotes(address account) → uint256
public#
返回该账户当前的投票数量。

#### getPastVotes(address account, uint256 timepoint) → uint256
public#
返回该账户在过去特定时刻的投票数量。如果clock()被配置为使用区块编号，那么这将返回相应区块结束时的值。

要求：
* 时间点必须在过去。如果操作使用区块编号，该区块必须已经被挖出。

#### getPastTotalSupply(uint256 timepoint) → uint256
public#
返回过去某一特定时刻可用的投票总供应量。如果clock()被配置为使用区块编号，那么它将返回相应区块结束时的值。

> NOTE
这个值是所有可用投票的总和，不一定是所有委托投票的总和。尚未被委托的投票仍然是总供应的一部分，尽管它们不会参与投票。

要求：
* 时间点必须在过去。如果操作使用区块编号，区块必须已经被挖出。

#### _getTotalSupply() → uint256
internal#
返回当前的总投票供应量。

#### delegates(address account) → address
public#
返回账户选择的代表。

#### delegate(address delegatee)
public#
将发送者的投票权代理给受托人。

#### delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
public#
将签署者的投票权委托给被委托人。

#### _delegate(address account, address delegatee)
internal#
将账户的所有投票单位委托给`delegatee。

触发事件*IVotes.DelegateChanged*和*IVotes.DelegateVotesChanged*。

#### _transferVotingUnits(address from, address to, uint256 amount)
internal#
转移、铸造或销毁投票单位。要注册铸币，来源应为零。要注册销毁，目标应为零。投票单位的总供应量将随着铸币和销毁进行调整。

#### _numCheckpoints(address account) → uint32
internal#
获取账户的检查点数量。

#### _checkpoints(address account, uint32 pos) → struct Checkpoints.Checkpoint208
internal#
获取账户的第pos个检查点。

#### _getVotingUnits(address) → uint256
internal#
必须返回账户持有的投票单位。

#### ERC6372InconsistentClock()
error#
时间锁被错误地修改了。

#### ERC5805FutureLookup(uint256 timepoint, uint48 clock)
error#
未来投票的查询不可用。

## Timelock
在治理系统中，*TimelockController*合约负责在提案和执行之间引入延迟。它可以与*Governor*一起使用，也可以单独使用。

### [TimelockController](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/governance/TimelockController.sol)
```
import "@openzeppelin/contracts/governance/TimelockController.sol";
```

这个合约模块充当一个时间锁控制器。当被设置为Ownable智能合约的所有者时，它会对所有只有所有者才能进行的维护操作强制执行时间锁。这给了受控合约的用户在可能危险的维护操作被应用之前退出的时间。

默认情况下，这个合约是自我管理的，意味着管理任务必须通过时间锁过程。提议者（或执行者）角色负责提议（或执行）操作。一个常见的用例是将这个*TimelockController*定位为智能合约的所有者，而多签名或DAO则是唯一的提议者。

**MODIFIERS**
onlyRoleOrOpenRole(role)

**FUNCTIONS**
constructor(minDelay, proposers, executors, admin)

receive()

supportsInterface(interfaceId)

isOperation(id)

isOperationPending(id)

isOperationReady(id)

isOperationDone(id)

getTimestamp(id)

getOperationState(id)

getMinDelay()

hashOperation(target, value, data, predecessor, salt)

hashOperationBatch(targets, values, payloads, predecessor, salt)

schedule(target, value, data, predecessor, salt, delay)

scheduleBatch(targets, values, payloads, predecessor, salt, delay)

cancel(id)

execute(target, value, payload, predecessor, salt)

executeBatch(targets, values, payloads, predecessor, salt)

_execute(target, value, data)

updateDelay(newDelay)

_encodeStateBitmap(operationState)

PROPOSER_ROLE()

EXECUTOR_ROLE()

CANCELLER_ROLE()

ERC1155HOLDER
onERC1155Received(, , , , )

onERC1155BatchReceived(, , , , )

ERC721HOLDER
onERC721Received(, , , )

ACCESSCONTROL
hasRole(role, account)

_checkRole(role)

_checkRole(role, account)

getRoleAdmin(role)

grantRole(role, account)

revokeRole(role, account)

renounceRole(role, callerConfirmation)

_setRoleAdmin(role, adminRole)

_grantRole(role, account)

_revokeRole(role, account)

DEFAULT_ADMIN_ROLE()

**EVENTS**
CallScheduled(id, index, target, value, data, predecessor, delay)

CallExecuted(id, index, target, value, data)

CallSalt(id, salt)

Cancelled(id)

MinDelayChange(oldDuration, newDuration)

IACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

**ERRORS**
TimelockInvalidOperationLength(targets, payloads, values)

TimelockInsufficientDelay(delay, minDelay)

TimelockUnexpectedOperationState(operationId, expectedStates)

TimelockUnexecutedPredecessor(predecessorId)

TimelockUnauthorizedCaller(caller)

IACCESSCONTROL
AccessControlUnauthorizedAccount(account, neededRole)

AccessControlBadConfirmation()

#### onlyRoleOrOpenRole(bytes32 role)
modifier#
使函数只能由特定角色调用的修饰符。除了检查发送者的角色，还会考虑地址(0)的角色。将角色授予地址(0)等同于为所有人启用此角色。

#### constructor(uint256 minDelay, address[] proposers, address[] executors, address admin)
public#
该合约以以下参数进行初始化：

* minDelay：操作的初始最小延迟（以秒为单位）

* proposers：被授予提案者和取消者角色的账户

* executors：被授予执行者角色的账户

* admin：可选的被授予管理员角色的账户；通过零地址禁用

> IMPORTANT
可选的管理员可以在部署后帮助初步配置角色，而无需受到延迟的影响，但是，应该随后放弃这种角色，以支持通过时间锁定提案进行管理。此合约的前一版本会自动将此管理员分配给部署者，应该放弃这种角色。

#### receive()
external#
合约可能会在维护过程中接收/持有ETH。

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查阅 *IERC165.supportsInterface*.

#### isOperation(bytes32 id) → bool
public#
返回一个id是否对应于已注册的操作。这包括等待，准备和完成的操作。

#### isOperationPending(bytes32 id) → bool
public#
返回一个操作是否处于待处理状态。请注意，"待处理"的操作也可能是"就绪"的。

#### isOperationReady(bytes32 id) → bool
public#
返回一个操作是否准备好执行。请注意，一个“准备好的”操作也是“待处理的”。

#### isOperationDone(bytes32 id) → bool
public#
返回操作是否已完成。

#### getTimestamp(bytes32 id) → uint256
public#
返回操作准备就绪的时间戳（对于未设置的操作为0，对于完成的操作为1）。

#### getOperationState(bytes32 id) → enum TimelockController.OperationState
public#
返回操作状态。

#### getMinDelay() → uint256
public#
返回操作生效的最小延迟时间（以秒为单位）。

通过执行调用updateDelay的操作，可以更改此值。

#### hashOperation(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt) → bytes32
public#
返回包含单个交易的操作的标识符。

#### hashOperationBatch(address[] targets, uint256[] values, bytes[] payloads, bytes32 predecessor, bytes32 salt) → bytes32
public#
返回包含一批交易的操作的标识符。

#### schedule(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt, uint256 delay)
public#
安排包含单个交易的操作。

如果盐不为零，则发出*CallSalt*，并且发出*CallScheduled*。

要求：
* 调用者必须具有'提议者'角色。

#### scheduleBatch(address[] targets, uint256[] values, bytes[] payloads, bytes32 predecessor, bytes32 salt, uint256 delay)
public#
安排包含一批交易的操作。

如果盐不为零，发出*CallSalt*，每个批次交易发出一个*CallScheduled*事件。

要求：
* 调用者必须具有'提议者'角色。

#### cancel(bytes32 id)
public#
取消一个操作。

要求：
* 调用者必须具有'取消者'角色。

#### execute(address target, uint256 value, bytes payload, bytes32 predecessor, bytes32 salt)
public#
执行包含单个交易的（已准备好的）操作。

发出*CallExecuted*事件。

要求：
* 调用者必须具有'执行者'角色。

#### executeBatch(address[] targets, uint256[] values, bytes[] payloads, bytes32 predecessor, bytes32 salt)
public#

执行包含一批交易的（已准备好的）操作。

每个批次的交易都会发出一个*CallExecuted*事件。

要求：
* 调用者必须具有'执行者'角色。

#### _execute(address target, uint256 value, bytes data)
internal#
执行操作的调用。

#### updateDelay(uint256 newDelay)
external#
更改未来操作的最小时间锁定持续时间。

发出一个*MinDelayChange*事件。

要求：
* 调用者必须是时间锁本身。这只能通过安排并稍后执行一个操作来实现，其中时间锁是目标，数据是对此函数的ABI编码调用。

#### _encodeStateBitmap(enum TimelockController.OperationState operationState) → bytes32
internal#
将OperationState编码为bytes32表示，其中每个启用的位对应于OperationState枚举中的基础位置。例如：

0x000…​1000 ^^----- …​^---- 完成 ^--- 准备 ^-- 等待 ^- 未设置

#### PROPOSER_ROLE() → bytes32
public#

#### EXECUTOR_ROLE() → bytes32
public#

#### CANCELLER_ROLE() → bytes32
public#

#### CallScheduled(bytes32 indexed id, uint256 indexed index, address target, uint256 value, bytes data, bytes32 predecessor, uint256 delay)
event#
当调用作为操作ID的一部分被安排时发出。

#### CallExecuted(bytes32 indexed id, uint256 indexed index, address target, uint256 value, bytes data)
event#
在执行操作ID的一部分时发出的调用。

#### CallSalt(bytes32 indexed id, bytes32 salt)
event#
当新的提案被安排并且含有非零盐值时发出。

#### Cancelled(bytes32 indexed id)
event#
当操作ID被取消时发出。

#### MinDelayChange(uint256 oldDuration, uint256 newDuration)
event#
最小延迟更改(旧持续时间，新持续时间)

#### TimelockInvalidOperationLength(uint256 targets, uint256 payloads, uint256 values)
error#
操作调用的参数长度不匹配。

#### TimelockInsufficientDelay(uint256 delay, uint256 minDelay)
error#
调度操作未达到最小延迟。

#### TimelockUnexpectedOperationState(bytes32 operationId, bytes32 expectedStates)
error#
操作的当前状态不符合要求。expectedStates是一个位图，其中每个OperationState枚举位置从右到左计数的位都被启用。

请参阅*_encodeStateBitmap*。

#### TimelockUnexecutedPredecessor(bytes32 predecessorId)
error#
尚未进行的操作的前驱。

#### TimelockUnauthorizedCaller(address caller)
error#
呼叫者账户未被授权。

## 术语
**Operation（操作）**：是时间锁的主题的交易（或一组交易）。它必须由提议者安排并由执行者执行。时间锁在提议和执行之间强制执行最小延迟（参见*操作生命周期*）。如果操作包含多个交易（批处理模式），它们将以原子方式执行。操作由其内容的哈希标识。

* **Operation status（操作状态）：**

  * **Unset（未设置）：**不是时间锁机制的一部分的操作。

  * **Waiting（等待）：**已安排的操作，在计时器到期之前。

  * **Ready（就绪）：**已安排的操作，在计时器到期后。

  * **Pending（待处理）：**等待或就绪的操作。

* **Done（完成）：**已执行的操作。

* **Predecessor（前驱）**：操作之间的（可选）依赖关系。一个操作可以依赖于另一个操作（其前驱），强制执行这两个操作的顺序。

* **Role（角色）：**

* **Admin（管理员）：**负责授予提议者和执行者角色的地址（智能合约或EOA）。

* **Proposer（提议者）：**负责安排（和取消）操作的地址（智能合约或EOA）。

* **Executor（执行者）：**负责在时间锁到期后执行操作的地址（智能合约或EOA）。这个角色可以给予零地址，以允许任何人执行操作。

## 操作结构
由*TimelockController*执行的操作可以包含一个或多个后续调用。根据你是否需要多个调用被原子性地执行，你可以使用简单或批处理操作。

两种操作都包含：

* Target，时间锁应该操作的智能合约的地址。

* Value，以wei为单位，应该与交易一起发送。大多数情况下这将是0。以太币可以在结束前存入或在执行交易时传递。

* Data，包含编码的函数选择器和调用的参数。这可以使用许多工具生成。例如，授予角色ROLE给ACCOUNT的维护操作可以使用web3js编码如下：
```
const data = timelock.contract.methods.grantRole(ROLE, ACCOUNT).encodeABI()
```

* Predecessor，指定操作之间的依赖关系。这种依赖关系是可选的。如果操作没有任何依赖关系，使用bytes32(0)。

* Salt，用于消除两个完全相同的操作的歧义。这可以是任何随机值。

在批处理操作的情况下，目标，值和数据被指定为数组，它们必须具有相同的长度。

## 操作生命周期
时间锁操作由唯一的id（它们的哈希）标识，并遵循特定的生命周期：

未设置→待处理→待处理+就绪→完成

* 通过调用*schedule*（或*scheduleBatch*），提议者将操作从未设置状态移动到待处理状态。这启动了一个必须超过最小延迟的计时器。计时器在通过*getTimestamp*方法可访问的时间戳到期。

* 一旦计时器到期，操作自动进入就绪状态。此时，它可以被执行。

* 通过调用*execute*（或*executeBatch*），执行者触发操作的底层交易并将其移动到完成状态。如果操作有一个前驱，它必须处于完成状态，此转换才能成功。

* *cancel*允许提议者取消任何待处理的操作。这将操作重置为未设置状态。因此，提议者可以重新安排已被取消的操作。在这种情况下，当操作被重新安排时，计时器重新开始。

可以使用以下函数查询操作状态：

* *isOperationPending(bytes32)*

* *isOperationReady(bytes32)*

* *isOperationDone(bytes32)*

## Roles

### Admin
管理员负责管理提议者和执行者。为了使时间锁自我管理，这个角色应该只给时间锁本身。在部署时，管理员角色可以授予任何地址（除了时间锁本身）。在进一步配置和测试后，这个可选的管理员应该放弃其角色，以便所有进一步的维护操作都必须通过时间锁过程。

### Proposer
提议者负责安排（和取消）操作。这是一个关键角色，应该由治理实体担任。这可以是EOA，多签名，或DAO。

> WARNING
**Proposer fight：**虽然有多个提议者可以在一个变得不可用的情况下提供冗余，但也可能是危险的。因为提议者对所有操作都有发言权，他们可以取消他们不同意的操作，包括移除他们的提议者的操作。

这个角色由**PROPOSER_ROLE**值标识：0xb09aa5aeb3702cfd50b6b62bc4532604938f21248a27a1d5ca736082b6819cc1

### Executor
执行者负责执行提议者安排的操作，一旦时间锁到期。逻辑规定，作为提议者的多签名或DAO应该也是执行者，以保证已经安排的操作最终会被执行。然而，有额外的执行者可以降低成本（执行交易不需要由提议它的多签名或DAO验证），同时确保负责执行的人不能触发没有被提议者安排的操作。另外，可以通过将执行者角色授予零地址，允许任何地址在时间锁到期后执行提议。

这个角色由**EXECUTOR_ROLE**值标识：0xd8aa0f3194971a2a116679f7c2090f6939c8d4e01a2a8d7e41d55e5351469e63

> WARNING
一个没有至少一个提议者和一个执行者的实时合约是锁定的。在部署者放弃其行政权利，以便时间锁合约本身之前，确保这些角色由可靠的实体填充。参见*AccessControl文档*，了解更多关于角色管理的信息。