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
提供的签名对预期的投票者无效。如果投票者是合同，那么使用IERC1271.isValidSignature的签名无效。

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
接收ETH的函数将由管理员处理（如果执行者是第三方合同，则禁用）

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
将交易或函数调用转发到任意目标。在治理执行者是除了调控器本身之外的一些合同的情况下，例如在使用时间锁时，可以在治理提案中调用此功能来恢复错误地发送到调控器合同的代币或以太币。请注意，如果执行者只是调控器本身，那么使用中继是多余的。

#### _executor() → address
internal#
通过该地址，调控器执行操作。将被通过其他合同执行操作的模块重载，例如时间锁。

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
州长通过该地址执行操作。在这种情况下，是时间锁。

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