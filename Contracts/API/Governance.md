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
投票开始和投票结束之间的延迟。此持续时间的单位取决于此合约使用的时钟（参见EIP-6372）。

votingDelay可以延迟投票的开始。在设置投票持续时间与投票延迟相比时，必须考虑到这一点。

#### quorum(uint256 timepoint) → uint256
