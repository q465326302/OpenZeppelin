# Governance
该目录包括用于链上治理的原语。

## Governor
这个模块化的Governor合约系统允许部署类似于[Compound的Governor Alpha和Bravo](https://compound.finance/docs/governance)等链上投票协议，通过能够轻松定制协议的多个方面。

> TIP
对于一个引导式体验，可以使用[Contracts Wizard](https://wizard.openzeppelin.com/#governor)来设置你的Governor合约。
对于一个书面步骤，可以查看我们的[《如何设置链上治理》指南](./Governance.md)。

* [Governor](#governor): 包含所有逻辑和原语的核心合约。它是抽象的，并需要选择下面每个模块中的一个，或者自定义模块。

Votes模块确定投票权的来源，有时也确定法定人数。

* [GovernorVotes](#governorvotes): 从[ERC20Votes](./ERC20.md#erc20votes)或v4.5以后的[ERC721Votes](./ERC721.md#erc721votes)代币中提取投票权重。
* [GovernorVotesComp](#governorvotescomp): 从类似于COMP或[ERC20VotesComp](./ERC20.md#erc20votescomp)代币中提取投票权重。
* [GovernorVotesQuorumFraction](#governorvotesquorumfraction): 与GovernorVotes结合使用，将法定人数设置为总代币供应量的一部分。

Counting模块确定有效的投票选项。

* [GovernorCountingSimple](#governorcountingsimple): 简单的投票机制，有3个投票选项：反对、赞成和弃权。

Timelock扩展为治理决策添加了延迟。工作流程扩展为在执行之前需要一个队列步骤。有了这些模块，提案将由外部的Timelock合约执行，因此Timelock必须持有被治理的资产。

* [GovernorTimelockControl](#governortimelockcontrol): 连接到[TimelockController](#timelockcontroller)的实例。允许多个提案者和执行者，以及Governor本身。

* [GovernorTimelockCompound](#governortimelockcompound): 连接到Compound的[Timelock](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Timelock.sol)合约的实例。

其他扩展可以以多种方式定制行为或接口。

* [GovernorCompatibilityBravo](#governorcompatibilitybravo): 扩展接口以完全兼容GovernorBravo。请注意，无论是否包含此扩展，事件都是兼容的。

* [GovernorSettings](#governorsettings): 以可以通过治理提案更新的方式管理一些设置（投票延迟、投票期间持续时间和提案阈值），而无需进行升级。

* [GovernorPreventLateQuorum](#governorpreventlatequorum): 确保在达到法定人数后有一个最小的投票期限，以作为对大型投票人的安全防护措施。

除了模块和扩展之外，核心合约还需要实现一些虚拟函数来满足你的特定规格要求：

* [votingDelay()](#votingdelay-→-uint256): 提案提交后到固定投票权和开始投票之间的延迟（以EIP-6372时钟为单位）。这可以用来强制在提案发布后用户购买代币或委托他们的投票之前进行延迟。

* [votingPeriod()](#votingperiod-→-uint256): 提案开始后到投票结束之间的延迟（以EIP-6372时钟为单位）。

* [quorum(uint256 timepoint)](#quorumnumeratoruint256-timepoint-→-uint256): 提案成功所需的法定人数。此函数包括一个时间点参数（参见EIP-6372），以便法定人数可以随时间变化，例如跟随代币的totalSupply。

> NOTE
Governor合约的函数不包括访问控制。如果要限制访问，应通过重载特定函数来添加这些检查。其中，[Governor._cancel](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)默认是internal的，如果需要使用此函数，你将需要自己公开它（并使用正确的访问控制机制）。

## 核心

### IGovernor
```
import "@openzeppelin/contracts/governance/IGovernor.sol";
```

[Governor](#governor) 核心的界面。

*从v4.3版本开始可用。*

**FUNCTIONS**
[name()](#name-→-string)
[version()](#version-→-string)
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[COUNTING_MODE()](#counting_mode-→-string)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[state(proposalId)](#stateuint256-proposalid-→-enum-igovernorproposalstate)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-→-uint256)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-→-uint256)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-→-address)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-→-uint256)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256-proposalid)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256-proposalid)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256-proposalid)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256-balance)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256-balance)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256-balance)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256-balance)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256-balance)

IERC165
[supportsInterface(interfaceId)](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool)

**EVENTS**
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

#### name() → string
public#
构建ERC712域分隔符时使用的governor实例的名称。

#### version() → string
public#
governor实例的版本（用于构建ERC712域分隔符）。默认值为“1”。

#### clock() → uint48
public#
请参阅 [IERC6372](./Interfaces.md#ierc6372)

#### CLOCK_MODE() → string
public#
请参阅 EIP-6372.

#### COUNTING_MODE() → string
public#
这是一个描述[castVote](#castvoteuint256-proposalid-uint8-support-→-uint256-balance)可能的支持值以及这些投票如何计算的字符串，用于UI显示正确的投票选项和解释结果。该字符串是一个URL编码的键值对序列，每个键值对描述一个方面，例如support=bravo&quorum=for,abstain。

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
public#
用于从提案详细信息中重新构建提案ID的哈希函数。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
public#
根据Compound的约定，提案的当前状态。

#### proposalSnapshot(uint256 proposalId) → uint256
public#
用于检索用户投票和法定人数的时间点。如果使用块编号（按照Compound的Comp），快照将在该块结束时执行。因此，对该提案的投票将从以下块的开始开始。

#### proposalDeadline(uint256 proposalId) → uint256
public#
投票截止时间点。如果使用块编号，则投票在该块结束时关闭，因此可以在该块期间投票。

#### proposalProposer(uint256 proposalId) → address
public#
创建提案的账户。

#### votingDelay() → uint256
public#
在提案创建和投票开始之间存在延迟。此持续时间的单位取决于此合约使用的时钟（请参阅EIP-6372）。

可以增加此延迟时间，以便用户有时间购买投票权或委托投票权，然后再开始对提案进行投票。

#### votingPeriod() → uint256
public#
投票开始和投票结束之间的延迟。此持续时间的单位取决于此合约使用的时钟（参见EIP-6372）。

> NOTE
[votingDelay](#votingdelay-→-uint256)可以延迟投票的开始。在设置投票持续时间与投票延迟相比时，必须考虑到这一点。

#### quorum(uint256 timepoint) → uint256
public#
成功提案所需的最低投票数。

> NOTE
时间点参数对应于用于计算投票的快照。这允许根据此时间点的总供应量等值来调整法定人数（请参阅[ERC20Votes](./ERC20.md#erc20votes)）。

#### getVotes(address account, uint256 timepoint) → uint256
public#
特定时间点上帐户的投票权力。

注意：这可以通过多种方式实现，例如通过从一个（或多个）[ERC20Votes](./ERC20.md#erc20votes)代币中读取委托余额来实现。

#### getVotesWithParams(address account, uint256 timepoint, bytes params) → uint256
public#
给定额外的编码参数，一个账户在特定时间点的投票权力。

#### hasVoted(uint256 proposalId, address account) → bool
public#
返回帐户是否对提案ID投票。

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256 proposalId
public#

创建一个新提案。投票在[IGovernor.votingDelay](#votingdelay-→-uint256)指定的延迟后开始，并持续[IGovernor.votingPeriod](#votingperiod-→-uint256)指定的持续时间。

发出一个[ProposalCreated](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)事件。

#### execute(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256 proposalId
public#

成功执行提案。这需要达到法定人数，投票成功，并达到截止日期。

发布[ProposalExecuted](#proposalexecuteduint256-proposalid)事件。

注意：某些模块可以修改执行要求，例如添加额外的时间锁。

#### cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256 proposalId
public#
取消提案。提案可以被提出者取消，但只能在待定状态下取消，即在投票开始之前。

发出一个[ProposalCanceled](#proposalcanceleduint256-proposalid)事件。

#### castVote(uint256 proposalId, uint8 support) → uint256 balance
public#
投票

发出一个[VoteCast](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)事件。

#### castVoteWithReason(uint256 proposalId, uint8 support, string reason) → uint256 balance
public#
投票并附上理由。

触发[VoteCast](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)事件。

#### castVoteWithReasonAndParams(uint256 proposalId, uint8 support, string reason, bytes params) → uint256 balance
public#
根据参数的长度，发出[VoteCast](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)或[VoteCastWithParams](#votecastwithparamsaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason-bytes-params)事件。

#### castVoteBySig(uint256 proposalId, uint8 support, uint8 v, bytes32 r, bytes32 s) → uint256 balance
public#
使用用户的密码签名进行投票。

发出一个[VoteCast](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)事件。

#### castVoteWithReasonAndParamsBySig(uint256 proposalId, uint8 support, string reason, bytes params, uint8 v, bytes32 r, bytes32 s) → uint256 balance
public#
使用用户的加密签名进行投票，并附加编码参数进行投票。

根据params的长度发出[VoteCast](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)或[VoteCastWithParams](#votecastwithparamsaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason-bytes-params)事件。

#### ProposalCreated(uint256 proposalId, address proposer, address[] targets, uint256[] values, string[] signatures, bytes[] calldatas, uint256 voteStart, uint256 voteEnd, string description)
event#
当提案被创建时发出。

#### ProposalCanceled(uint256 proposalId)
event#
当一个提案被取消时发出的信号。

#### ProposalExecuted(uint256 proposalId)
event#
当提案执行时发出。

#### VoteCast(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason)
event#
当没有参数投票时发出的信号。

注意：支持值应视为桶。它们的解释取决于所使用的投票模块。

#### VoteCastWithParams(address indexed voter, uint256 proposalId, uint8 support, uint256 weight, string reason, bytes params)
event#
在使用参数进行投票时发出。

注意：支持值应被视为桶。它们的解释取决于所使用的投票模块。参数是额外编码的参数。它们的解释也取决于所使用的投票模块。

### Governor
```
import "@openzeppelin/contracts/governance/Governor.sol";
```

这个治理系统的核心是被设计成可以通过各种模块进行扩展的。

该合约是抽象的，并需要在各个模块中实现几个函数：

* 计数模块必须实现[quorum](#quorumuint256-timepoint-→-uint256)、[_quorumReached](#_quorumreacheduint256-proposalid-→-bool)、[_voteSucceeded](#_votesucceededuint256-proposalid-→-bool)和[_countVote](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)函数。

* 投票模块必须实现[_getVotes](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)函数。

* 此外，还必须实现[votingPeriod](#votingperiod-→-uint256)函数。

*该功能从v4.3版本开始提供。*

**MODIFIERS**
[onlyGovernance()](#onlygovernance)

**FUNCTIONS**
[constructor(name_)](#constructorstring-name_)
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

EVENTS
IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### onlyGovernance()
modifier#
限制一个函数，只能通过治理提案执行。例如，[GovernorSettings](#governorsettings)中的治理参数设置器使用此修饰符进行保护。

治理执行地址可能与Governor自身的地址不同，例如可能是一个时间锁。这可以通过模块自定义，通过重写[_executor](#_executor-→-address)函数来实现。执行者只能在治理程序的[execute](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)函数执行期间调用这些函数，而不能在其他任何情况下调用。因此，例如，附加的时间锁提议者不能在不经过治理协议的情况下更改治理参数（自v4.6起）。

#### constructor(string name_)
internal#
设置[name](#name-e28692-string-1)和[version](#version-e28692-string-1)的值

#### receive()
external#
接收ETH的函数，将由治理合约处理（如果执行者是第三方合约，则禁用）

#### supportsInterface(bytes4 interfaceId) → bool
public#
请参见 [IERC165.supportsInterface](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool)。

#### name() → string
public#
请参见 [IGovernor.name](#name-→-string).

#### version() → string
public#
请参见 [IGovernor.version](#version-→-string).

#### hashProposal(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
请参考 [IGovernor.hashProposal](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)。

提案ID是通过对ABI编码的targets数组、values数组、calldatas数组和descriptionHash（bytes32，它本身是描述字符串的keccak256哈希）进行哈希计算得到的。这个提案ID可以从[ProposalCreated](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)事件的提案数据中生成。甚至可以在提案提交之前提前计算出来。

请注意，chainId和governor地址不是提案ID计算的一部分。因此，如果在多个网络上的多个治理合约提交相同的提案（具有相同的操作和相同的描述），则它们将具有相同的ID。这也意味着为了在同一个治理合约上执行相同的操作两次，提案人必须更改描述以避免提案ID冲突。

#### state(uint256 proposalId) → enum IGovernor.ProposalState
public#
请参考 [IGovernor.state](#stateuint256-proposalid-→-enum-igovernorproposalstate).

#### proposalThreshold() → uint256
public#
Governor Bravo界面的一部分：*“选民成为提案人所需的票数”*。

#### proposalSnapshot(uint256 proposalId) → uint256
public#
请参阅[IGovernor.proposalSnapshot](#proposalsnapshotuint256-proposalid-→-uint256)。

#### proposalDeadline(uint256 proposalId) → uint256
public#
请参阅 [IGovernor.proposalDeadline](#proposaldeadlineuint256-proposalid-→-uint256).

#### proposalProposer(uint256 proposalId) → address
public#
返回创建给定提案的账户。

#### _quorumReached(uint256 proposalId) → bool
internal#
已经投票的数量超过了阈值限制。

#### _voteSucceeded(uint256 proposalId) → bool
internal#
提案成功与否

#### _getVotes(address account, uint256 timepoint, bytes params) → uint256
internal#
获取特定时间点的帐户的投票权重，用params描述的投票方式。

#### _countVote(uint256 proposalId, address account, uint8 support, uint256 weight, bytes params)
internal#

根据给定的支持、投票权重和投票参数，为提案ID的账户注册一次投票。

注意：支持是通用的，根据所使用的投票系统可以代表各种不同的含义。

#### _defaultParams() → bytes
internal#
默认情况下，castVote方法使用的附加编码参数（不包括它们）。

注意：应该由特定的实现来重写以使用适当的值，附加参数的含义在该实现的上下文中。

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256
public#
查看 [IGovernor.propose](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256-proposalid) 函数。该函数包含 opt-in 前置交易保护，其描述在 [_isValidDescriptionForProposer](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool) 中说明。

#### execute(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
请参阅 [IGovernor.execute](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256-proposalid).

#### cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
请参阅 [IGovernor.cancel](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256-proposalid).

#### _execute(uint256, address[] targets, uint256[] values, bytes[] calldatas, bytes32)
internal#
内部执行机制。可以被重写以实现不同的执行机制。

#### _beforeExecute(uint256, address[] targets, uint256[], bytes[] calldatas, bytes32)
internal#
在执行触发之前的 hooks 。

#### _afterExecute(uint256, address[], uint256[], bytes[], bytes32)
internal#
在执行被触发后的 hooks 。

#### _cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
internal#
内部取消机制：锁定提案计时器，防止其重新提交。将其标记为已取消，以便与已执行的提案区分。

发出[IGovernor.ProposalCanceled](#proposalcanceleduint256-proposalid)事件。

#### getVotes(address account, uint256 timepoint) → uint256
public#
请参阅 [IGovernor.getVotes](#getvotesaddress-account-uint256-timepoint-→-uint256).

#### getVotesWithParams(address account, uint256 timepoint, bytes params) → uint256
public#
请参阅 [IGovernor.getVotesWithParams](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-→-uint256).

#### castVote(uint256 proposalId, uint8 support) → uint256
public#
请参阅 [IGovernor.castVote](#castvoteuint256-proposalid-uint8-support-→-uint256-balance).

#### castVoteWithReason(uint256 proposalId, uint8 support, string reason) → uint256
public#
请参阅 [IGovernor.castVoteWithReason](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256-balance).

#### castVoteWithReasonAndParams(uint256 proposalId, uint8 support, string reason, bytes params) → uint256
public#
请参阅 [IGovernor.castVoteWithReasonAndParams](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256-balance).

#### castVoteBySig(uint256 proposalId, uint8 support, uint8 v, bytes32 r, bytes32 s) → uint256
public#
请参阅 [IGovernor.castVoteBySig](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256-balance).

#### castVoteWithReasonAndParamsBySig(uint256 proposalId, uint8 support, string reason, bytes params, uint8 v, bytes32 r, bytes32 s) → uint256
public#
请参阅 [IGovernor.castVoteWithReasonAndParamsBySig](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256-balance).

#### _castVote(uint256 proposalId, address account, uint8 support, string reason) → uint256
internal#

内部投票机制：检查投票是否待定，尚未投票，使用[IGovernor.getVotes](#getvotesaddress-account-uint256-timepoint-→-uint256)检索投票权重，并调用[_countVote](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)内部函数。使用_defaultParams()。

触发[IGovernor.VoteCast](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)事件。

#### _castVote(uint256 proposalId, address account, uint8 support, string reason, bytes params) → uint256
internal#
内部投票投票机制：检查投票是否挂起，是否尚未投票，使用[IGovernor.getVotes](#getvotesaddress-account-uint256-timepoint-→-uint256)检索投票权重，并调用[_countVote](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)内部函数。

发出(#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)事件。

#### relay(address target, uint256 value, bytes data)
external#
将交易或函数调用转发到任意目标。在使用时间锁定等合约作为治理执行者的情况下，可以在治理提案中调用此函数来恢复错误发送到治理合约的代币或以太。请注意，如果执行者只是治理本身，则使用Relayer 是多余的。

#### _executor() → address
internal#
governor执行行动的地址。将通过执行通过另一个合约（如时间锁）的模块来进行重载。

#### onERC721Received(address, address, uint256, bytes) → bytes4
public#
请参阅  [IERC721Receiver.onERC721Received](./ERC721.md#onerc721receivedaddress-operator-address-from-uint256-tokenid-bytes-data-→-bytes4).

####  onERC1155Received(address, address, uint256, uint256, bytes) → bytes4
public#
请参阅 [IERC1155Receiver.onERC1155Received](./ERC1155.md#onerc1155receivedaddress-operator-address-from-uint256-id-uint256-value-bytes-data-→-bytes4).

#### onERC1155BatchReceived(address, address, uint256[], uint256[], bytes) → bytes4
public#
请参阅 [IERC1155Receiver.onERC1155BatchReceived](./ERC1155.md#onerc1155batchreceivedaddress-operator-address-from-uint256-ids-uint256-values-bytes-data-→-bytes4).

#### _isValidDescriptionForProposer(address proposer, string description) → bool
internal#
检查提案人是否被授权提交具有给定描述的提案。

如果提案描述以#proposer=0x???结尾，其中0x???是以十六进制字符串（不区分大小写）表示的地址，则只有该地址被授权提交此提案。

这用于防止前置交易。通过在其提案末尾添加此模式，提案人可以确保没有其他地址可以提交相同的提案。攻击者必须删除或更改该部分，这将导致不同的提案ID。

如果描述不符合此模式，则不受限制，任何人都可以提交。这包括： - 如果0x???部分不是有效的十六进制字符串。 - 如果0x???部分是有效的十六进制字符串，但不包含确切的40个十六进制数字。 - 如果以预期的后缀后跟换行符或其他空格符号结尾。 - 如果以其他类似的后缀结尾，例如#other=abc。 - 如果没有以任何此类后缀结尾。

## 模块

### GovernorCountingSimple
```
import "@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol";
```

对于简单的3个选项，投票计数的延伸，可以使用Governor扩展。
*自V4.3版本启用。*

FUNCTIONS
[COUNTING_MODE()](#counting_mode-e28692-string-1)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-e28692-bool-1)
[proposalVotes(proposalId)](#proposalvotesuint256-proposalid-→-uint256-againstvotes-uint256-forvotes-uint256-abstainvotes)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-e28692-bool-1)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-e28692-bool-1)
[_countVote(proposalId, account, support, weight, )](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes)

GOVERNOR
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

**EVENTS**
IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### COUNTING_MODE() → string
public#
请参阅 [IGovernor.COUNTING_MODE](#counting_mode-→-string).

#### hasVoted(uint256 proposalId, address account) → bool
public#
请参阅 [IGovernor.hasVoted](#hasvoteduint256-proposalid-address-account-→-bool).

#### proposalVotes(uint256 proposalId) → uint256 againstVotes, uint256 forVotes, uint256 abstainVotes
public#
访问内部投票计数器。

#### _quorumReached(uint256 proposalId) → bool
internal#
请参阅 [Governor._quorumReached](#_quorumreacheduint256-proposalid-→-bool).

#### _voteSucceeded(uint256 proposalId) → bool
internal#
请参阅 [Governor._voteSucceeded](#_votesucceededuint256-proposalid-→-bool).
在这个模块中，forVotes必须严格超过againstVotes。

#### _countVote(uint256 proposalId, address account, uint8 support, uint256 weight, bytes)
internal#
请参阅 [Governor._countVote](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
在这个模块中，支持遵循了从Governor Bravo中的VoteType枚举。

#### GovernorVotes
```
import "@openzeppelin/contracts/governance/extensions/GovernorVotes.sol";
```

从[ERC20Votes](./ERC20.md#erc20votes)代币或v4.5之后的[ERC721Votes](./ERC721.md#erc721votes)代币中提取投票权重的[Governor](#governor)扩展。

*从v4.3版本开始可用。*

FUNCTIONS
[constructor(tokenAddress)](#constructorcontract-ivotes-tokenaddress)
[clock()](#clock-e28692-uint48-1)
[CLOCK_MODE()](#clock_mode-e28692-string-1)
[_getVotes(account, timepoint, )](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

GOVERNOR
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

**EVENTS**
IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor(contract IVotes tokenAddress)
internal#

#### clock() → uint48
public#
如果代币没有实施EIP-6372协议，那么Clock（按照EIP-6372规定）将被设置为与代币的时钟匹配，否则将回退到区块号。

#### CLOCK_MODE() → string
public#
在EIP-6372中指定的时钟的机器可读描述。

#### _getVotes(address account, uint256 timepoint, bytes) → uint256
internal#

### GovernorVotesQuorumFraction
```
import "@openzeppelin/contracts/governance/extensions/GovernorVotesQuorumFraction.sol";
```

从[ERC20Votes](./ERC20.md#erc20votes)代币中提取投票权重的[Governor](#governor)扩展和以总供应量的一部分表示的法定人数。

*从v4.3版本开始可用。*

**FUNCTIONS**
[constructor(quorumNumeratorValue)](#constructoruint256-quorumnumeratorvalue)
[quorumNumerator()](#quorumnumerator-→-uint256)
[quorumNumerator(timepoint)](#quorumnumeratoruint256-timepoint-→-uint256)
[quorumDenominator()](#quorumdenominator-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-e28692-uint256-1)
[updateQuorumNumerator(newQuorumNumerator)](#updatequorumnumeratoruint256-newquorumnumerator)
[_updateQuorumNumerator(newQuorumNumerator)](#_updatequorumnumeratoruint256-newquorumnumerator)

GOVERNORVOTES
[clock()](#clock-e28692-uint48-1)
[CLOCK_MODE()](#clock_mode-e28692-string-1)
[_getVotes(account, timepoint, )](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

GOVERNOR
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

**EVENTS**
[QuorumNumeratorUpdated(oldQuorumNumerator, newQuorumNumerator)](#quorumnumeratorupdateduint256-oldquorumnumerator-uint256-newquorumnumerator)

IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor(uint256 quorumNumeratorValue)
internal#
将法定人数初始化为代币总供应量的一部分。

该分数被指定为分子/分母。默认情况下，分母为100，因此法定人数被指定为百分比：分子为10对应于法定人数为总供应量的10％。可以通过重写[quorumDenominator](#quorumdenominator-→-uint256)来自定义分母。

#### quorumNumerator() → uint256
public#
返回当前配额分子。请参见[quorumDenominator](#quorumdenominator-→-uint256).

#### quorumNumerator(uint256 timepoint) → uint256
public#
返回特定时间点的法定人数的分子。参见[quorumDenominator](#quorumdenominator-→-uint256)。

#### quorumDenominator() → uint256
public#
返回法定人数的分母。默认为100，但可以被重写。

#### quorum(uint256 timepoint) → uint256
public#
以投票数量为单位，返回时间点的法定人数：供应量 * 分子 / 分母。

#### updateQuorumNumerator(uint256 newQuorumNumerator)
external#
更改法定人数的分子。

发出一个[QuorumNumeratorUpdated](#quorumnumeratorupdateduint256-oldquorumnumerator-uint256-newquorumnumerator)事件。

要求:
* 必须通过治理提案调用。
* 新的分子必须小于或等于分母。

#### _updateQuorumNumerator(uint256 newQuorumNumerator)
internal#
更改法定人数的分子。

发出一个(#quorumnumeratorupdateduint256-oldquorumnumerator-uint256-newquorumnumerator)事件。

要求：
* 新的分子必须小于或等于分母。

#### QuorumNumeratorUpdated(uint256 oldQuorumNumerator, uint256 newQuorumNumerator)
event#

### GovernorVotesComp
```
import "@openzeppelin/contracts/governance/extensions/GovernorVotesComp.sol";
```

从Comp代币中提取投票权重的[Governor](#governor)扩展

*从v4.3版本开始可用。*

**FUNCTIONS**
[constructor(token_)](#constructorcontract-erc20votescomp-token_)
[clock()](#clock-e28692-uint48-2)
[CLOCK_MODE()](#clock_mode-e28692-string-2)
[_getVotes(account, timepoint, )](#_getvotesaddress-account-uint256-timepoint-bytes-e28692-uint256-1)

GOVERNOR
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

EVENTS
IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor(contract ERC20VotesComp token_)
internal#

#### clock() → uint48
public#
如果代币没有实施EIP-6372，则根据EIP-6372中指定的方式设置时钟。如果代币没有实施EIP-6372，则回退到区块号。

#### CLOCK_MODE() → string
public#
在EIP-6372中指定的时钟的机器可读描述。

#### _getVotes(address account, uint256 timepoint, bytes) → uint256
internal#

## 扩展

### GovernorTimelockControl
```
import "@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol";
```

将执行过程与[TimelockController](#timelockcontroller)实例绑定的[Governor](#governor)扩展。这将为所有成功的提案（除了投票持续时间外）增加由[TimelockController](#timelockcontroller)执行的延迟。为了使[Governor](#governor)正常工作，它需要提案者（最好还有执行者）角色。

使用这个模型意味着提案将由[TimelockController](#timelockcontroller)操作，而不是由[Governor](#governor)操作。因此，资产和权限必须附加到[TimelockController](#timelockcontroller)上。发送到[Governor](#governor)的任何资产将无法访问。

> WARNING
在TimelockController中设置额外的提议者，除了治理合约之外，是非常危险的，因为它赋予了他们一些必须信任或者知道他们不会使用的权力：1）只有[onlyGovernance](#onlygovernance)功能（如[Relayer](#relayaddress-target-uint256-value-bytes-data)）可以通过Timelock进行操作，2）他们可以阻止已批准的治理提案，从而有效地执行拒绝服务攻击。这个风险将在未来的版本中得到缓解。

*从v4.3版本开始提供。*

**FUNCTIONS**
[constructor(timelockAddress)](#constructorcontract-timelockcontroller-timelockaddress)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-e28692-bool-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-2)
[timelock()](#timelock-→-address)
[proposalEta(proposalId)](#proposaletauint256-proposalid-→-uint256)
[queue(targets, values, calldatas, descriptionHash)]
[_execute(, targets, values, calldatas, descriptionHash)]
[_cancel(targets, values, calldatas, descriptionHash)]
[_executor()](#_executor-e28692-address-1)
[updateTimelock(newTimelock)](#updatetimelockcontract-timelockcontroller-newtimelock)

GOVERNOR
[receive()](#receive)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)


EVENTS
[TimelockChange(oldTimelock, newTimelock)](#timelockchangeaddress-oldtimelock-address-newtimelock)

IGOVERNORTIMELOCK
ProposalQueued(proposalId, eta)

IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor(contract TimelockController timelockAddress)
internal#
设置时间锁。

#### supportsInterface(bytes4 interfaceId) → bool
public#
请参阅 [IERC165.supportsInterface](#supportsinterfacebytes4-interfaceid-→-bool).

#### state(uint256 proposalId) → enum IGovernor.ProposalState
public#
重写了[Governor.state](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)函数的版本，增加了对Queued状态的支持。

#### timelock() → address
public#
公共访问器来检查时间锁的地址

#### proposalEta(uint256 proposalId) → uint256
public#
公共访问器用于检查排队提案的预计时间

#### queue(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
将提案排队到时间锁的功能。

#### _execute(uint256, address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash)
internal#
重写的execute函数通过timelock定运行已排队的提案。

#### _cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
internal#
[Governor._cancel](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)函数的重写版本，用于取消已经排队的时间锁提案。

#### _executor() → address
internal#
通过这个地址，governor执行行动。在这种情况下，是指timelock定。

#### updateTimelock(contract TimelockController newTimelock)
external#
公共端点用于更新底层的timelock实例。仅限于timelock本身，因此更新必须通过治理提案进行提议、计划和执行。

> CAUTION
不建议在存在其他排队的治理提案时更改timelock。

#### TimelockChange(address oldTimelock, address newTimelock)
event#
当用于提案执行的时间锁控制器被修改时发出。

### GovernorTimelockCompound
```
import "@openzeppelin/contracts/governance/extensions/GovernorTimelockCompound.sol";
```

对将执行过程绑定到复合时间锁的[Governor](#governor)进行扩展。这将为所有成功的提案（除了投票持续时间之外）添加一个由外部时间锁强制执行的延迟。[Governor](#governor)需要成为时间锁的管理员才能执行任何操作。公开的、无限制的*GovernorTimelockCompound.acceptAdmin*可用于接受时间锁的所有权。

使用这个模型意味着提案将由[TimelockController](#timelockcontroller)操作，而不是由[Governor](#governor)操作。因此，资产和权限必须附加到[TimelockController](#timelockcontroller)上。发送到[Governor](#governor)的任何资产都将无法访问。

*自v4.3起可用。*

**FUNCTIONS**
[constructor(timelockAddress)](#constructorcontract-icompoundtimelock-timelockaddress)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-e28692-bool-2)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-3)
[timelock()](#timelock-e28692-address-1)
[proposalEta(proposalId)](#proposaletauint256-proposalid-e28692-uint256-1)
[queue(targets, values, calldatas, descriptionHash)](#queueaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[_execute(proposalId, targets, values, calldatas, )](#_executeuint256-proposalid-address-targets-uint256-values-bytes-calldatas-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-2)
[_executor()](#_executor-e28692-address-2)
[__acceptAdmin()](#__acceptadmin)
[updateTimelock(newTimelock)](#updatetimelockcontract-icompoundtimelock-newtimelock)

GOVERNOR
[receive()](#receive)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

EVENTS
[TimelockChange(oldTimelock, newTimelock)](#timelockchangeaddress-oldtimelock-address-newtimelock-1)

IGOVERNORTIMELOCK
ProposalQueued(proposalId, eta)

IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor(contract ICompoundTimelock timelockAddress)
internal#
设置时间锁。

#### supportsInterface(bytes4 interfaceId) → bool
public#
请参阅 [IERC165.supportsInterface](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool).

#### state(uint256 proposalId) → enum IGovernor.ProposalState
public#
重写的[Governor.state](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)函数的版本，增加了对Queued和Expired状态的支持。

#### timelock() → address
public#
公共访问器用于检查时间锁的地址

#### proposalEta(uint256 proposalId) → uint256
public#
公共访问者来检查排队提案的预计时间

#### queue(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
将一个提案排队到时间锁定的函数。

#### _execute(uint256 proposalId, address[] targets, uint256[] values, bytes[] calldatas, bytes32)
internal#
重写的execute函数通过timelock定来运行已排队的提案。

#### _cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
internal#
重写版本的 [Governor._cancel](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256) 函数，用于取消已经排队的时间锁定提案。

#### _executor() → address
internal#
行动的执行地址是governor。在这种情况下，时间锁。

#### __acceptAdmin()
public#
接受对时间锁的管理员权限。

#### updateTimelock(contract ICompoundTimelock newTimelock)
external#
公共端点用于更新底层的timelock实例。该端点仅限timelock本身使用，因此更新必须通过治理提案进行提议、计划和执行。

出于安全原因，在设置新的timelock之前，必须将timelock移交给另一个管理员。两个操作（移交timelock）和进行更新可以合并为一个提案。

请注意，如果timelock管理员在之前的操作中已被移交，我们将拒绝通过timelock进行的更新，如果timelock的管理员已经接受并且操作是在治理范围之外执行的。

> CAUTION
不建议在存在其他排队中的治理提案时更改timelock。

#### TimelockChange(address oldTimelock, address newTimelock)
event#

### GovernorSettings
```
import "@openzeppelin/contracts/governance/extensions/GovernorSettings.sol";
```

通过治理设置可以更新的[Governor](#governor)延期

*自v4.4起可用。*

**FUNCTIONS**
[constructor(initialVotingDelay, initialVotingPeriod, initialProposalThreshold)](#constructoruint256-initialvotingdelay-uint256-initialvotingperiod-uint256-initialproposalthreshold)
[votingDelay()](#votingdelay-e28692-uint256-1)
[votingPeriod()]
[proposalThreshold()]
[setVotingDelay(newVotingDelay)]
[setVotingPeriod(newVotingPeriod)]
[setProposalThreshold(newProposalThreshold)]
[_setVotingDelay(newVotingDelay)]
[_setVotingPeriod(newVotingPeriod)]
[_setProposalThreshold(newProposalThreshold)]

GOVERNOR
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[COUNTING_MODE()](#counting_mode-→-string)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

EVENTS
[VotingDelaySet(oldVotingDelay, newVotingDelay)](#votingdelaysetuint256-oldvotingdelay-uint256-newvotingdelay)
[VotingPeriodSet(oldVotingPeriod, newVotingPeriod)](#votingperiodsetuint256-oldvotingperiod-uint256-newvotingperiod)
[ProposalThresholdSet(oldProposalThreshold, newProposalThreshold)](#proposalthresholdsetuint256-oldproposalthreshold-uint256-newproposalthreshold)

IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor(uint256 initialVotingDelay, uint256 initialVotingPeriod, uint256 initialProposalThreshold)
internal#
初始化治理参数。

#### votingDelay() → uint256
public#
请参阅 [IGovernor.votingDelay](#votingdelay-→-uint256).

#### votingPeriod() → uint256
public#
请参阅 [IGovernor.votingPeriod](#votingperiod-→-uint256).

#### proposalThreshold() → uint256
public#
请参阅 [Governor.proposalThreshold](#proposalthreshold-→-uint256).

#### setVotingDelay(uint256 newVotingDelay)
public#
更新投票延迟。此操作只能通过治理提案完成。

发送 [VotingDelaySet](#votingdelaysetuint256-oldvotingdelay-uint256-newvotingdelay)事件。

#### setVotingPeriod(uint256 newVotingPeriod)
public#
更新投票期限。此操作只能通过治理提案进行。

发出一个[VotingPeriodSet](#votingperiodsetuint256-oldvotingperiod-uint256-newvotingperiod)事件。

#### setProposalThreshold(uint256 newProposalThreshold)
public#
更新提案门槛。此操作只能通过治理提案进行。

发出[ProposalThresholdSet](#proposalthresholdsetuint256-oldproposalthreshold-uint256-newproposalthreshold)事件。

#### _setVotingDelay(uint256 newVotingDelay)
internal#
为投票延迟设置一个内部的setter函数。

触发一个[VotingDelaySet](#votingdelaysetuint256-oldvotingdelay-uint256-newvotingdelay)事件。

#### _setVotingPeriod(uint256 newVotingPeriod)
internal#
设置投票期的内部设置器。

发出一个[VotingPeriodSet](#votingperiodsetuint256-oldvotingperiod-uint256-newvotingperiod)事件。

#### _setProposalThreshold(uint256 newProposalThreshold)
内部设定提案阈值。

发出[ProposalThresholdSet](#proposalthresholdsetuint256-oldproposalthreshold-uint256-newproposalthreshold)事件。

#### VotingDelaySet(uint256 oldVotingDelay, uint256 newVotingDelay)
event#

#### VotingPeriodSet(uint256 oldVotingPeriod, uint256 newVotingPeriod)
event#

#### ProposalThresholdSet(uint256 oldProposalThreshold, uint256 newProposalThreshold)

### GovernorPreventLateQuorum
```
import "@openzeppelin/contracts/governance/extensions/GovernorPreventLateQuorum.sol";
```

一个模块，确保在达到法定人数后有最低的投票期限。这样可以防止一个大选民在最后一刻扭转选票并引发法定人数，通过确保其他选民始终有时间做出反应并试图反对决策。

如果一项投票导致达到法定人数，提案的投票期限可以延长，以确保至少经过指定的时间后才结束（“投票延长”参数）。此参数可以通过治理提案进行设置。

*自v4.5版本起可用。*

**FUNCTIONS**
[constructor(initialVoteExtension)](#constructoruint64-initialvoteextension)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-2)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-e28692-uint256-1)
[lateQuorumVoteExtension()](#latequorumvoteextension-→-uint64)
[setLateQuorumVoteExtension(newVoteExtension)](#setlatequorumvoteextensionuint64-newvoteextension)
[_setLateQuorumVoteExtension(newVoteExtension)](#_setlatequorumvoteextensionuint64-newvoteextension)

GOVERNOR
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

EVENTS
[ProposalExtended(proposalId, extendedDeadline)](#proposalextendeduint256-indexed-proposalid-uint64-extendeddeadline)
[LateQuorumVoteExtensionSet(oldVoteExtension, newVoteExtension)](#latequorumvoteextensionsetuint64-oldvoteextension-uint64-newvoteextension)

IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor(uint64 initialVoteExtension)
internal#
初始化投票扩展参数：从提案达到法定人数到投票期结束所需的时间，以块数或秒数（取决于议长时钟模式）计算。如有需要，投票期限将延长超过提案创建时设定的期限。

#### proposalDeadline(uint256 proposalId) → uint256
public#
如果提案在投票期结束时才达到法定人数，那么返回的提案截止日期可能会超过提案创建时设定的截止日期。请参阅[Governor.proposalDeadline](#proposaldeadlineuint256-proposalid-e28692-uint256-1)。

#### _castVote(uint256 proposalId, address account, uint8 support, string reason, bytes params) → uint256
internal#
投票并检测是否导致法定人数达到，有可能延长投票期。参见[Governor._castVote](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)。

可能会发出[ProposalExtended](#proposalextendeduint256-indexed-proposalid-uint64-extendeddeadline)事件。

#### lateQuorumVoteExtension() → uint64
public#
返回投票扩展参数的当前值：从提案达到法定人数直到其投票期结束所需通过的区块数量。

#### setLateQuorumVoteExtension(uint64 newVoteExtension)
public#
更改*lateQuorumVoteExtension*。此操作只能由治理执行者执行，通常是通过治理提案进行。

发出*LateQuorumVoteExtensionSet*事件。

#### _setLateQuorumVoteExtension(uint64 newVoteExtension)
internal#
更改[lateQuorumVoteExtension](#latequorumvoteextension-→-uint64)。这是一个内部函数，如果需要另一个访问控制机制，则可以在公共函数中公开，如[setLateQuorumVoteExtension](#setlatequorumvoteextensionuint64-newvoteextension)。

发出[LateQuorumVoteExtensionSet](#latequorumvoteextensionsetuint64-oldvoteextension-uint64-newvoteextension)事件。

#### ProposalExtended(uint256 indexed proposalId, uint64 extendedDeadline)
event#
当提案的截止日期因在投票期间达到法定人数而被推迟时发出。

#### LateQuorumVoteExtensionSet(uint64 oldVoteExtension, uint64 newVoteExtension)
event#
当[lateQuorumVoteExtension](#latequorumvoteextension-→-uint64)参数被更改时发出。

### GovernorCompatibilityBravo
```
import "@openzeppelin/contracts/governance/compatibility/GovernorCompatibilityBravo.sol";
```

GovernorBravo是一种[Governor](#governor)的兼容性层，它在Governor之上实现GovernorBravo的兼容性。

该兼容性层包括一个投票系统，并通过继承要求添加一个兼容*IGovernorTimelock*的模块。它不包括代币绑定，也不包括任何变量升级模式。

在使用此模块时，你可能需要启用Solidity优化器，以避免达到合约大小限制。

*自v4.3版本起可用。*

**FUNCTIONS**
[COUNTING_MODE()](#counting_mode-e28692-string-2)
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-e28692-uint256-1)
[propose(targets, values, signatures, calldatas, description)](#proposeaddress-targets-uint256-values-string-signatures-bytes-calldatas-string-description-→-uint256)
[queue(proposalId)](#queueuint256-proposalid)
[execute(proposalId)](#executeuint256-proposalid)
[cancel(proposalId)](#canceluint256-proposalid)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[proposals(proposalId)](#proposalsuint256-proposalid-→-uint256-id-address-proposer-uint256-eta-uint256-startblock-uint256-endblock-uint256-forvotes-uint256-againstvotes-uint256-abstainvotes-bool-canceled-bool-executed)
[getActions(proposalId)](#getactionsuint256-proposalid-→-address-targets-uint256-values-string-signatures-bytes-calldatas)
[getReceipt(proposalId, voter)](#getreceiptuint256-proposalid-address-voter-→-struct-igovernorcompatibilitybravoreceipt)
[quorumVotes()](#quorumvotes-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-e28692-bool-2)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-e28692-bool-2)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-e28692-bool-2)
[_countVote(proposalId, account, support, weight, )](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-1)

**GOVERNOR**
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_defaultParams()](#_defaultparams-→-bytes)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNORTIMELOCK
timelock()
proposalEta(proposalId)
queue(targets, values, calldatas, descriptionHash)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

EVENTS
IGOVERNORTIMELOCK
ProposalQueued(proposalId, eta)

IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

### COUNTING_MODE() → string
public#
这是一个用于解释支持和解释[castVote](#castvoteuint256-proposalid-uint8-support-→-uint256)结果的字符串，旨在被UI界面使用以显示正确的投票选项和解释结果。该字符串是一个URL编码的键值对序列，每个键值对描述一个方面，例如support=bravo&quorum=for,abstain。

有两个标准键：support和quorum。
* support=bravo指的是投票选项0 = 反对，1 = 赞成，2 = 弃权，就像GovernorBravo中的投票选项。
* quorum=bravo表示只有赞成票计入法定人数。
* quorum=for,abstain表示赞成票和弃权票都计入法定人数。

如果计数模块使用了编码参数，则应将其包含在一个具有描述行为的唯一名称的params键下。例如：
* params=fractional可能指的是一种方案，其中投票在赞成/反对/弃权之间按比例分配。
* params=erc721可能指的是一种方案，其中特定的NFT被委托进行投票。

> NOTE
这个字符串可以通过标准的[URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) JavaScript类进行解码。

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256
public#
请参阅 [IGovernor.propose](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256-proposalid).

#### propose(address[] targets, uint256[] values, string[] signatures, bytes[] calldatas, string description) → uint256
public#
请参阅 [IGovernorCompatibilityBravo.propose].

#### queue(uint256 proposalId)
public#
请参阅 [IGovernorCompatibilityBravo.queue].

#### execute(uint256 proposalId)
public#
请参阅 [IGovernorCompatibilityBravo.execute].

#### cancel(uint256 proposalId)
public#
取消与GovernorBravo逻辑的提案。

#### cancel(address[] targets, uint256[] values, bytes[] calldatas, bytes32 descriptionHash) → uint256
public#
使用GovernorBravo逻辑取消提案。在任何时刻，提案可以被取消，无论是由提案人取消，还是由第三方取消，前提是提案人的投票权已经降低到提案阈值以下。

#### proposals(uint256 proposalId) → uint256 id, address proposer, uint256 eta, uint256 startBlock, uint256 endBlock, uint256 forVotes, uint256 againstVotes, uint256 abstainVotes, bool canceled, bool executed
public#
请参阅 [IGovernorCompatibilityBravo.proposals].

#### getActions(uint256 proposalId) → address[] targets, uint256[] values, string[] signatures, bytes[] calldatas
public#
请参阅 [IGovernorCompatibilityBravo.getActions].

#### getReceipt(uint256 proposalId, address voter) → struct IGovernorCompatibilityBravo.Receipt
public#
请参阅 [IGovernorCompatibilityBravo.getReceipt].

#### quorumVotes() → uint256
public#
请参阅 [IGovernorCompatibilityBravo.quorumVotes].

#### hasVoted(uint256 proposalId, address account) → bool
public#
请参阅 [IGovernor.hasVoted](#hasvoteduint256-proposalid-address-account-→-bool).

#### _quorumReached(uint256 proposalId) → bool
internal#
请参阅 [Governor._quorumReached](#_quorumreacheduint256-proposalid-→-bool)。在此模块中，只有对投票进行计数才能达到法定人数。

#### _voteSucceeded(uint256 proposalId) → bool
internal#
请参阅 [Governor._voteSucceeded](#_votesucceededuint256-proposalid-→-bool)。在这个模块中，支持票必须严格超过反对票。

#### _countVote(uint256 proposalId, address account, uint8 support, uint256 weight, bytes)
internal#
请参阅 [Governor._countVote](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params).在这个模块中，支持者们追随着 Governor Bravo。

### Deprecated

#### GovernorProposalThreshold
```
import "@openzeppelin/contracts/governance/extensions/GovernorProposalThreshold.sol";
```

对提案限制的[Governor](#governor)扩展，仅限于持有最低余额的代币持有者。

*从v4.3开始可用。从v4.4开始已弃用。*

**FUNCTIONS**
[propose(targets, values, calldatas, description)](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-e28692-uint256-2)

GOVERNOR
[receive()](#receive)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[name()](#name-e28692-string-1)
[version()](#version-e28692-string-1)
[hashProposal(targets, values, calldatas, descriptionHash)](#hashproposaladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-e28692-uint256-1)
[state(proposalId)](#stateuint256-proposalid-e28692-enum-igovernorproposalstate-1)
[proposalThreshold()](#proposalthreshold-→-uint256)
[proposalSnapshot(proposalId)](#proposalsnapshotuint256-proposalid-e28692-uint256-1)
[proposalDeadline(proposalId)](#proposaldeadlineuint256-proposalid-e28692-uint256-1)
[proposalProposer(proposalId)](#proposalproposeruint256-proposalid-e28692-address-1)
[_quorumReached(proposalId)](#_quorumreacheduint256-proposalid-→-bool)
[_voteSucceeded(proposalId)](#_votesucceededuint256-proposalid-→-bool)
[_getVotes(account, timepoint, params)](#_getvotesaddress-account-uint256-timepoint-bytes-params-→-uint256)
[_countVote(proposalId, account, support, weight, params)](#_countvoteuint256-proposalid-address-account-uint8-support-uint256-weight-bytes-params)
[_defaultParams()](#_defaultparams-→-bytes)
[execute(targets, values, calldatas, descriptionHash)](#executeaddress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[cancel(targets, values, calldatas, descriptionHash)](#canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[_execute(, targets, values, calldatas, )](#_executeuint256-address-targets-uint256-values-bytes-calldatas-bytes32)
[_beforeExecute(, targets, , calldatas, )](#_beforeexecuteuint256-address-targets-uint256-bytes-calldatas-bytes32)
[_afterExecute(, , , , )](#_afterexecuteuint256-address-uint256-bytes-bytes32)
[_cancel(targets, values, calldatas, descriptionHash)](#_canceladdress-targets-uint256-values-bytes-calldatas-bytes32-descriptionhash-→-uint256)
[getVotes(account, timepoint)](#getvotesaddress-account-uint256-timepoint-e28692-uint256-1)
[getVotesWithParams(account, timepoint, params)](#getvoteswithparamsaddress-account-uint256-timepoint-bytes-params-e28692-uint256-1)
[castVote(proposalId, support)](#castvoteuint256-proposalid-uint8-support-→-uint256)
[castVoteWithReason(proposalId, support, reason)](#castvotewithreasonuint256-proposalid-uint8-support-string-reason-→-uint256)
[castVoteWithReasonAndParams(proposalId, support, reason, params)](#castvotewithreasonandparamsuint256-proposalid-uint8-support-string-reason-bytes-params-→-uint256)
[castVoteBySig(proposalId, support, v, r, s)](#castvotebysiguint256-proposalid-uint8-support-uint8-v-bytes32-r-bytes32-s-→-uint256)
[castVoteWithReasonAndParamsBySig(proposalId, support, reason, params, v, r, s)](#castvotewithreasonandparamsbysiguint256-proposalid-uint8-support-string-reason-bytes-params-uint8-v-bytes32-r-bytes32-s-→-uint256)
[_castVote(proposalId, account, support, reason)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-→-uint256)
[_castVote(proposalId, account, support, reason, params)](#_castvoteuint256-proposalid-address-account-uint8-support-string-reason-bytes-params-→-uint256)
[relay(target, value, data)](#relayaddress-target-uint256-value-bytes-data)
[_executor()](#_executor-→-address)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-→-bytes4)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-→-bytes4)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-→-bytes4)
[_isValidDescriptionForProposer(proposer, description)](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)

IGOVERNOR
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)
[COUNTING_MODE()](#counting_mode-→-string)
[votingDelay()](#votingdelay-→-uint256)
[votingPeriod()](#votingperiod-→-uint256)
[quorum(timepoint)](#quorumuint256-timepoint-→-uint256)
[hasVoted(proposalId, account)](#hasvoteduint256-proposalid-address-account-→-bool)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

EVENTS
IGOVERNOR
[ProposalCreated(proposalId, proposer, targets, values, signatures, calldatas, voteStart, voteEnd, description)](#proposalcreateduint256-proposalid-address-proposer-address-targets-uint256-values-string-signatures-bytes-calldatas-uint256-votestart-uint256-voteend-string-description)
[ProposalCanceled(proposalId)](#proposalcanceleduint256-proposalid)
[ProposalExecuted(proposalId)](#proposalexecuteduint256-proposalid)
[VoteCast(voter, proposalId, support, weight, reason)](#votecastaddress-indexed-voter-uint256-proposalid-uint8-support-uint256-weight-string-reason)
[VoteCastWithParams(voter, proposalId, support, weight, reason, params)](#_getvotesaddress-account-uint256-timepoint-bytes-→-uint256)

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### propose(address[] targets, uint256[] values, bytes[] calldatas, string description) → uint256
public#
请查看[IGovernor.propose](#proposeaddress-targets-uint256-values-bytes-calldatas-string-description-→-uint256-proposalid)函数。该函数具有opt-in的前运行保护，其描述在[_isValidDescriptionForProposer](#_isvaliddescriptionforproposeraddress-proposer-string-description-→-bool)中解释。

## 实用程序

### Votes
```
import "@openzeppelin/contracts/governance/utils/Votes.sol";
```

这是一个基本的抽象合约，用于跟踪投票单位，投票单位是可以转让的投票权力的度量，并提供投票委托系统，其中一个账户可以将其投票单位委托给一种“代表”，该代表将从不同账户汇集委托的投票单位，然后可以用它来投票决策。实际上，只有委托了投票单位才能算作实际投票，如果一个账户希望参与决策但没有可信的代表，它必须将这些投票委托给自己。

这个合约通常与一个代币合约结合使用，使得投票单位对应于代币单位。例如，参见[ERC721Votes](./ERC721.md#erc721votes)。

委托投票的完整历史记录被跟踪在链上，以便治理协议可以将投票视为在特定区块号分布的，以防止闪电贷和双重投票。选择性的委托系统使得跟踪历史记录的成本可选。

在使用这个模块时，派生合约必须实现[_getVotingUnits](#_getvotingunitsaddress-→-uint256)（例如，使其返回[ERC721.balanceOf](./ERC721.md#balanceofaddress-owner-→-uint256)），并可以使用[_transferVotingUnits](#_transfervotingunitsaddress-from-address-to-uint256-amount)来跟踪这些单位分布的变化（在前面的例子中，它将包含在[ERC721._beforeTokenTransfer](./ERC721.md#_beforetokentransferaddress-from-address-to-uint256-firsttokenid-uint256-batchsize)中）。

*自v4.5起可用。*

**FUNCTIONS**
[clock()](#clock-e28692-uint48-3)
[CLOCK_MODE()](#clock_mode-e28692-string-3)
[getVotes(account)](#getvotesaddress-account-→-uint256)
[getPastVotes(account, timepoint)](#getpastvotesaddress-account-uint256-timepoint-→-uint256)
[getPastTotalSupply(timepoint)](#getpasttotalsupplyuint256-timepoint-→-uint256)
[_getTotalSupply()](#_gettotalsupply-→-uint256)
[delegates(account)](#delegatesaddress-account-→-address)
[delegate(delegatee)](#delegateaddress-delegatee)
[delegateBySig(delegatee, nonce, expiry, v, r, s)](#delegatebysigaddress-delegatee-uint256-nonce-uint256-expiry-uint8-v-bytes32-r-bytes32-s)
[_delegate(account, delegatee)](#_delegateaddress-account-address-delegatee)
[_transferVotingUnits(from, to, amount)](#_transfervotingunitsaddress-from-address-to-uint256-amount)
[_useNonce(owner)](#_usenonceaddress-owner-→-uint256-current)
[nonces(owner)](#noncesaddress-owner-→-uint256)
[DOMAIN_SEPARATOR()](#domain_separator-→-bytes32)
[_getVotingUnits()](#_getvotingunitsaddress-→-uint256)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

**EVENTS**
IVOTES
[DelegateChanged(delegator, fromDelegate, toDelegate)]
[DelegateVotesChanged(delegate, previousBalance, newBalance)]

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### clock() → uint48
public#
用于标记检查点的时钟。可以被重写以实现基于时间戳的检查点（和投票），在这种情况下，[CLOCK_MODE](#clock_mode-e28692-string-3)也应该被重写以匹配。

#### CLOCK_MODE() → string
public#

EIP-6372规定的时钟的机器可读描述。

#### getVotes(address account) → uint256
public#
返回该账户当前的投票数量。

#### getPastVotes(address account, uint256 timepoint) → uint256
public#
返回特定时刻过去账户的投票数量。如果时钟（clock()）配置为使用区块号，则会返回相应区块结束时的值。

要求：
* timepoint 必须在过去。如果使用区块号进行操作，则该区块必须已经被挖掘。

#### getPastTotalSupply(uint256 timepoint) → uint256
public#
在过去的特定时刻返回可用的总票数。如果时钟（clock()）配置为使用区块号，这将返回相应区块结束时的值。

> NOTE
该值是所有可用票数的总和，不一定是所有委派票数的总和。尚未委派的票数仍然是总供应的一部分，即使它们不参与投票。

要求：
* timepoint必须在过去。如果使用区块号进行操作，则该区块必须已经被挖掘。

#### _getTotalSupply() → uint256
internal#
返回当前总投票供应量。

#### delegates(address account) → address
public#
返回账户选择的代表。

#### delegate(address delegatee)
public#
将发件人的选票委托给受托人。

#### delegateBySig(address delegatee, uint256 nonce, uint256 expiry, uint8 v, bytes32 r, bytes32 s)
public#
将签名者的选票授予代表。

#### _delegate(address account, address delegatee)
internal#
将账户的所有投票单位委托给委托人。

触发事件[IVotes.DelegateChanged]和[IVotes.DelegateVotesChanged]。

#### _transferVotingUnits(address from, address to, uint256 amount)
internal#
转让、铸造或销毁投票单位。要注册一个铸造，from 应该是零。要注册一个销毁，to 应该是零。铸造和销毁将调整投票单位的总供应量。

#### _useNonce(address owner) → uint256 current
internal#
消耗一个随机数。

返回当前值并增加随机数。

#### nonces(address owner) → uint256
public#
返回一个地址的nonce。

#### DOMAIN_SEPARATOR() → bytes32
external#
返回合约[的EIP712](./Utils.md#eip712)域分隔符。

#### _getVotingUnits(address) → uint256
internal#
必须返回一个账户持有的投票单位。

## 时间锁定
在一种治理系统中，[TimelockController](#timelockcontroller)合约负责在提案和执行之间引入延迟。它可以与[Governor](#governor)一起使用，也可以单独使用。

### TimelockController
```
import "@openzeppelin/contracts/governance/TimelockController.sol";
```

合约模块充当一个timelock定的控制器。当设置为可拥有智能合约的所有者时，它会对所有仅所有者维护操作进行时间锁定。这样，在应用潜在危险的维护操作之前，用户可以有时间退出受控合约。

默认情况下，该合约是自我管理的，这意味着管理任务必须通过时间锁定流程进行。建议人（执行人）角色负责提出（执行）操作。一个常见的用例是将该[TimelockController](#timelockcontroller)定位为智能合约的所有者，由多重签名或DAO作为唯一的建议人。

*自v3.3起可用。*

**MODIFIERS**
[onlyRoleOrOpenRole(role)](#onlyroleoropenrolebytes32-role)

**FUNCTIONS**
[constructor(minDelay, proposers, executors, admin)](#constructoruint256-mindelay-address-proposers-address-executors-address-admin)
[receive()](#receive-1)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-e28692-bool-3)
[isOperation(id)](#isoperationbytes32-id-→-bool)
[isOperationPending(id)](#isoperationpendingbytes32-id-→-bool)
[isOperationReady(id)](#isoperationreadybytes32-id-→-bool)
[isOperationDone(id)](#isoperationdonebytes32-id-→-bool)
[getTimestamp(id)](#gettimestampbytes32-id-→-uint256)
[getMinDelay()](#getmindelay-→-uint256)
[hashOperation(target, value, data, predecessor, salt)](#hashoperationaddress-target-uint256-value-bytes-data-bytes32-predecessor-bytes32-salt-→-bytes32)
[hashOperationBatch(targets, values, payloads, predecessor, salt)](#hashoperationbatchaddress-targets-uint256-values-bytes-payloads-bytes32-predecessor-bytes32-salt-→-bytes32)
[schedule(target, value, data, predecessor, salt, delay)](#scheduleaddress-target-uint256-value-bytes-data-bytes32-predecessor-bytes32-salt-uint256-delay)
[scheduleBatch(targets, values, payloads, predecessor, salt, delay)](#schedulebatchaddress-targets-uint256-values-bytes-payloads-bytes32-predecessor-bytes32-salt-uint256-delay)
[cancel(id)](#cancelbytes32-id)
[execute(target, value, payload, predecessor, salt)](#executeaddress-target-uint256-value-bytes-payload-bytes32-predecessor-bytes32-salt)
[executeBatch(targets, values, payloads, predecessor, salt)](#executebatchaddress-targets-uint256-values-bytes-payloads-bytes32-predecessor-bytes32-salt)
[_execute(target, value, data)](#_executeaddress-target-uint256-value-bytes-data)
[updateDelay(newDelay)](#updatedelayuint256-newdelay)
[onERC721Received(, , , )](#onerc721receivedaddress-address-uint256-bytes-e28692-bytes4-1)
[onERC1155Received(, , , , )](#onerc1155receivedaddress-address-uint256-uint256-bytes-e28692-bytes4-1)
[onERC1155BatchReceived(, , , , )](#onerc1155batchreceivedaddress-address-uint256-uint256-bytes-e28692-bytes4-1)

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
[_grantRole(role, account)](./Access.md#_grantrolebytes32-role-address-account)
[_revokeRole(role, account)](./Access.md#_revokerolebytes32-role-address-account)

**EVENTS**
[CallScheduled(id, index, target, value, data, predecessor, delay)](#callscheduledbytes32-indexed-id-uint256-indexed-index-address-target-uint256-value-bytes-data-bytes32-predecessor-uint256-delay)
[CallExecuted(id, index, target, value, data)](#callexecutedbytes32-indexed-id-uint256-indexed-index-address-target-uint256-value-bytes-data)
[CallSalt(id, salt)](#callsaltbytes32-indexed-id-bytes32-salt)
[Cancelled(id)](#cancelledbytes32-indexed-id)
[MinDelayChange(oldDuration, newDuration)](#mindelaychangeuint256-oldduration-uint256-newduration)

IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](./Access.md#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](./Access.md#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](./Access.md#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### onlyRoleOrOpenRole(bytes32 role)
modifier#
对函数进行修改，使其只能被特定角色调用。除了检查发送者的角色外，还要考虑 address(0) 的角色。将角色授予 address(0) 相当于为所有人启用此角色。

#### constructor(uint256 minDelay, address[] proposers, address[] executors, address admin)
public#
使用以下参数初始化合约：
* minDelay: 操作的初始最小延迟
* proposers: 被授予提案人和取消人角色的账户
* executors: 被授予执行者角色的账户
* admin: 可选的账户，被授予管理员角色；使用零地址来禁用

> IMPORTANT
可选的管理员可以在部署后帮助进行角色的初始配置，而不受延迟的限制，但是应该在之后放弃此角色，改为通过时间锁定的提案进行管理。之前的合约版本会自动将此管理员分配给部署者，也应该放弃此角色。

#### receive()
external#
合约可能会在维护过程中收到/持有ETH。

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查阅 [IERC165.supportsInterface](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool).

#### isOperation(bytes32 id) → bool
public#
返回一个id是否对应于注册的操作。这包括待处理、准备好和完成的操作。

#### isOperationPending(bytes32 id) → bool
public#
返回操作是否挂起。请注意，“挂起”操作也可能是“就绪”的。

#### isOperationReady(bytes32 id) → bool
public#
返回操作是否准备好执行。请注意，“准备就绪”的操作也是“待处理”的。

#### isOperationDone(bytes32 id) → bool
public#
返回操作是否已完成。

#### getTimestamp(bytes32 id) → uint256
public#
返回操作准备就绪的时间戳（未设置操作为0，完成操作为1）。

#### getMinDelay() → uint256
public#
返回操作变得有效所需的最小延迟。

通过执行调用updateDelay的操作可以更改该值。

#### hashOperation(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt) → bytes32
public#
返回包含单个交易的操作的标识符。

#### hashOperationBatch(address[] targets, uint256[] values, bytes[] payloads, bytes32 predecessor, bytes32 salt) → bytes32
public#
返回包含一批交易的操作的标识符。

#### schedule(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt, uint256 delay)
public#
安排一个包含单个交易的操作。

如果盐值不为零，则发出[CallSalt](#callsaltbytes32-indexed-id-bytes32-salt)；同时发出[CallScheduled](#callscheduledbytes32-indexed-id-uint256-indexed-index-address-target-uint256-value-bytes-data-bytes32-predecessor-uint256-delay)。
要求：
* 调用者必须具有“提议者”角色。

#### scheduleBatch(address[] targets, uint256[] values, bytes[] payloads, bytes32 predecessor, bytes32 salt, uint256 delay)
public#
安排一个包含一批交易的操作。

如果盐不为零，则发出[CallSalt](#callsaltbytes32-indexed-id-bytes32-salt)，如果批处理中的每个交易发出一个[CallScheduled](#callscheduledbytes32-indexed-id-uint256-indexed-index-address-target-uint256-value-bytes-data-bytes32-predecessor-uint256-delay)事件。
要求：
* 调用者必须具有“提议者”角色。

#### cancel(bytes32 id)
public#
取消一个操作。

要求：
* 呼叫者必须拥有“取消者”角色。

#### execute(address target, uint256 value, bytes payload, bytes32 predecessor, bytes32 salt)
public#
执行一个包含单个事务的（准备好的）操作。

发出[CallExecuted](#callexecutedbytes32-indexed-id-uint256-indexed-index-address-target-uint256-value-bytes-data)事件。
要求：
* 调用者必须具有“executor”角色。

#### executeBatch(address[] targets, uint256[] values, bytes[] payloads, bytes32 predecessor, bytes32 salt)

执行一个包含一批交易的（准备好的）操作。

每个交易在批处理中发出一个[CallExecuted](#callexecutedbytes32-indexed-id-uint256-indexed-index-address-target-uint256-value-bytes-data)事件。
要求：
* 调用者必须具有'executor'角色。
  
#### _execute(address target, uint256 value, bytes data)
internal#
执行一个操作的调用。

#### updateDelay(uint256 newDelay)
external#
更改未来操作的最小时间锁定持续时间。

发出[MinDelayChange](#mindelaychangeuint256-oldduration-uint256-newduration)事件。

要求：
* 调用者必须是时间锁本身。只有通过安排并稍后执行一个将时间锁定设为目标，数据为ABI编码调用此函数的操作，才能实现这一要求。

#### onERC721Received(address, address, uint256, bytes) → bytes4

public#
请参阅 [IERC721Receiver.onERC721Received](./ERC721.md#onerc721receivedaddress-operator-address-from-uint256-tokenid-bytes-data-→-bytes4).

#### onERC1155Received(address, address, uint256, uint256, bytes) → bytes4
public#
请参阅 [IERC1155Receiver.onERC1155Received](./ERC1155.md#onerc1155receivedaddress-operator-address-from-uint256-id-uint256-value-bytes-data-→-bytes4).

#### onERC1155BatchReceived(address, address, uint256[], uint256[], bytes) → bytes4
请参阅 [IERC1155Receiver.onERC1155BatchReceived](./ERC1155.md#onerc1155batchreceivedaddress-operator-address-from-uint256-ids-uint256-values-bytes-data-→-bytes4).

#### CallScheduled(bytes32 indexed id, uint256 indexed index, address target, uint256 value, bytes data, bytes32 predecessor, uint256 delay)
event#
当一个调用被安排作为操作ID的一部分时发出。

#### CallExecuted(bytes32 indexed id, uint256 indexed index, address target, uint256 value, bytes data)
event#
当作为操作id的一部分执行调用时发出。

#### CallSalt(bytes32 indexed id, bytes32 salt)
event#
当使用非零盐值调度新提案时发出。

#### Cancelled(bytes32 indexed id)
event# 
当操作ID被取消时发出。

#### MinDelayChange(uint256 oldDuration, uint256 newDuration)
event# 
当未来操作的最小延迟被修改时发出。

## 术语
* **Operation**：一个交易（或一组交易），是timelock的主题。它必须由提议者安排并由执行者执行。timelock强制在提议和执行之间有最小延迟（参见operation lifecycle）。如果操作包含多个交易（批处理模式），它们将以原子方式执行。操作通过其内容的哈希值进行标识。

* **操作状态**：
    * **Unset**：不是timelock机制的一部分的操作。
    * **Pending**：已安排但计时器尚未到期的操作。
    * **Ready**：已安排且计时器已到期的操作。
    * **Done**：已执行的操作。

* **前置操作**：操作之间的（可选）依赖关系。一个操作可以依赖于另一个操作（其前置操作），从而强制这两个操作的执行顺序。

* **角色**：
    * **Admin**：负责授予提议者和执行者角色的地址（智能合约或EOA）。
    * **提议者**：负责安排（和取消）操作的地址（智能合约或EOA）。
    * **执行者**：负责在timelock过期后执行操作的地址（智能合约或EOA）。可以将此角色授予零地址，以允许任何人执行操作。

### 操作结构
由[TimelockController](#timelockcontroller)执行的操作可以包含一个或多个后续调用。根据是否需要原子执行多个调用，可以使用简单操作或批处理操作。

两种操作都包含：
* **目标**：timelock应该操作的智能合约的地址。
* **价值**：以wei为单位，应该与交易一起发送的价值。大多数情况下，这将为0。在执行交易时可以在结束前存入以太或传递。
* **数据**：包含调用的编码函数选择器和参数。可以使用多种工具生成。例如，使用web3js可以如下编码授予账户ROLE角色的维护操作：

```
const data = timelock.contract.methods.grantRole(ROLE, ACCOUNT).encodeABI()
```

* **Predecessor**，用于指定操作之间的依赖关系。此依赖关系是可选的。如果操作没有任何依赖关系，则使用bytes32(0)。
* **Salt**，用于消除两个否则相同的操作之间的歧义。可以是任意随机值。
对于批量操作，目标（target）、值（value）和数据（data）被指定为数组，这些数组必须具有相同的长度。

### 操作生命周期
具有时间锁定的操作通过唯一的id（其哈希）进行识别，并遵循特定的生命周期：

未设置 → 待处理 → 待处理 + 准备完成 → 已完成

* 通过调用[schedule](#scheduleaddress-target-uint256-value-bytes-data-bytes32-predecessor-bytes32-salt-uint256-delay)（或[scheduleBatch](#schedulebatchaddress-targets-uint256-values-bytes-payloads-bytes32-predecessor-bytes32-salt-uint256-delay)），提议者将操作从未设置状态移至待处理状态。这将启动一个计时器，该计时器的时间必须超过最小延迟。计时器在一个可以通过[getTimestamp](#gettimestampbytes32-id-→-uint256)方法访问的时间戳到期。
* 一旦计时器到期，操作会自动进入准备完成状态。此时，可以执行该操作。
* 通过调用execute（或[executeBatch](#executebatchaddress-targets-uint256-values-bytes-payloads-bytes32-predecessor-bytes32-salt)），执行者触发操作的基础交易，并将其移至已完成状态。如果操作有前任操作，则前任操作必须处于已完成状态，此次转换才会成功。
* [cancel]允许提议者取消任何待处理操作。这将将操作重置为未设置状态。因此，提议者可以重新安排已取消的操作。在这种情况下，操作重新安排时计时器重新启动。

可以使用以下函数查询操作的状态：
* [isOperationPending(bytes32)](#isoperationpendingbytes32-id-→-bool)
* [isOperationReady(bytes32)](#isoperationreadybytes32-id-→-bool)
* [isOperationDone(bytes32)](#isoperationdonebytes32-id-→-bool)

### 角色

#### 管理员
管理员负责管理提议者和执行者。为了使时间锁能够自我管理，该角色只应该授予时间锁本身。在部署之后，管理员角色可以授予任何地址（除了时间锁本身）。在进一步配置和测试之后，这个可选的管理员应该放弃其角色，以便所有后续的维护操作都必须通过时间锁流程进行。

该角色由**TIMELOCK_ADMIN_ROLE**值标识：0x5f58e3a2316349923ce3780f8d587db2d72378aed66a8261c916544fa6846ca5

#### 提议者
提议者负责安排（和取消）操作。这是一个关键的角色，应该授予治理实体。这可以是一个EOA，一个多签名账户或一个DAO。

> WARNING
**提议者之争**：拥有多个提议者可以在一个提议者不可用时提供冗余，但这可能是危险的。由于提议者对所有操作都有发言权，他们可以取消他们不同意的操作，包括删除他们作为提议者的操作。

该角色由**PROPOSER_ROLE**值标识：0xb09aa5aeb3702cfd50b6b62bc4532604938f21248a27a1d5ca736082b6819cc1

#### 执行者
执行者负责在时间锁到期后执行提议者安排的操作。逻辑上讲，多签名账户或DAO作为提议者也应该是执行者，以确保已安排的操作最终会被执行。然而，拥有额外的执行者可以降低成本（执行交易不需要多签名账户或DAO的验证），同时确保负责执行的人不能触发未经提议者安排的操作。另外，也可以通过将执行者角色授予零地址，允许任何地址在时间锁到期后执行提案。

该角色由**EXECUTOR_ROLE**值标识：0xd8aa0f3194971a2a116679f7c2090f6939c8d4e01a2a8d7e41d55e5351469e63

> WARNING
没有至少一个提议者和一个执行者的活跃合约是被锁定的。在部署者放弃其管理权利，将其转交给时间锁合约本身之前，请确保这些角色由可靠的实体填充。请参阅[AccessControl](./Access.md)文档以了解更多关于角色管理的信息。
