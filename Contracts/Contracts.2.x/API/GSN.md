# Gas Station Network (GSN)
从v2.4.0版本开始可用。

这组合同提供了通过[Gas Station Network（GSN）](https://gsn.openzeppelin.com/)调用合同所需的所有工具。

> TIP
如果您对GSN还不熟悉，请查看我们关于*该系统的概述*以及*创建支持GSN的合同的基本指南*。

接收方必须继承的核心合同是*GSNRecipient*：它包括所有必要的接口，以及一些帮助方法，使与GSN的交互更容易。

使编写*GSN策略*变得简单的实用程序可以在*GSNRecipient*中找到，或者您可以直接使用我们预先制作的策略之一：

* *GSNRecipientERC20Fee*使用特定应用的*ERC20代币*向最终用户收取燃气费用

* *GSNRecipientSignature*接受由受信任的第三方（例如后端中的私钥）签名的所有Relayer调用

您还可以查看组成GSN协议的两个合同接口：*IRelayRecipient*和*IRelayHub*，但您不需要直接使用它们。

## 接收方

### GSNRecipient
基本GSN接收方合同：包括*IRelayRecipient*接口，并在继承树中的所有合同上启用GSN支持。

> TIP
此合同是抽象的。*IRelayRecipient.acceptRelayedCall*、*_preRelayedCall*和*_postRelayedCall*函数未实现，必须由派生合同提供。有关如何使用预构建的*GSNRecipientSignature*和*GSNRecipientERC20Fee*的更多信息，请参阅*GSN策略*，或者如何编写您自己的策略。

**FUNCTIONS**
getHubAddr()

_upgradeRelayHub(newRelayHub)

relayHubVersion()

_withdrawDeposits(amount, payee)

_msgSender()

_msgData()

preRelayedCall(context)

_preRelayedCall(context)

postRelayedCall(context, success, actualCharge, preRetVal)

_postRelayedCall(context, success, actualCharge, preRetVal)

_approveRelayedCall()

_approveRelayedCall(context)

_rejectRelayedCall(errorCode)

_computeCharge(gas, gasPrice, serviceFee)

CONTEXT
constructor()

IRELAYRECIPIENT
acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)

**EVENTS**
RelayHubChanged(oldRelayHub, newRelayHub)

#### getHubAddr() → address
公开#
返回此接收者的*IRelayHub*合约的地址。

#### _upgradeRelayHub(address newRelayHub)
内部#
切换到新的*IRelayHub*实例。添加此方法是为了未来的兼容性：没有理由不使用默认实例。

> IMPORTANT
升级后，*GSNRecipient*将无法从旧的*IRelayHub*实例接收Relayer调用。此外，所有资金应通过*_withdrawDeposits*事先提取。

#### relayHubVersion() → string
公开#
返回此接收方实现构建的*IRelayHub*的版本字符串。如果使用*_upgradeRelayHub*，则新的*IRelayHub*实例应与此版本兼容。

#### _withdrawDeposits(uint256 amount, address payable payee)
内部#
从RelayHub中提取接收者的存款。

派生合约应该在外部接口中以适当的访问控制方式公开此功能。

#### _msgSender() → address payable
内部#
msg.sender的替代品。返回交易的实际发送者：对于常规交易是msg.sender，对于GSNRelayer调用是最终用户（其中msg.sender实际上是RelayHub）。

> IMPORTANT
从*GSNRecipient*派生的合约不应使用*msg.sender*，而应使用_msgSender。

#### _msgData() → bytes
内部#
替代msg.data。返回交易的实际calldata：对于常规交易，返回msg.data，对于GSNRelayer调用，返回一个简化版本（其中msg.data包含额外的信息）。

> IMPORTANT
从GSNRecipient派生的合约不应使用msg.data，而应使用_msgData。

#### preRelayedCall(bytes context) → bytes32
外部#
请参考IRelayRecipient.preRelayedCall函数。

不应直接覆盖此函数，而应使用_preRelayedCall。

* 要求：

    * 调用者必须是RelayHub合约。

#### _preRelayedCall(bytes context) → bytes32
外部#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
外部#
请参阅IRelayRecipient.postRelayedCall。

该函数不应直接被覆盖，而应使用_postRelayedCall。

* 要求：

    * 调用者必须是RelayHub合约。

#### _postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
内部#
请参阅IRelayRecipient.postRelayedCall。

由GSNRecipient.postRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer调用后处理。

#### _approveRelayedCall() → uint256, bytes
内部#
在acceptRelayedCall中返回此内容以继续执行Relayer调用。请注意，RelayHub将向此合约收取费用。

#### _approveRelayedCall(bytes context) → uint256, bytes
内部#
请查看 GSNRecipient._approveRelayedCall。

此重载将上下文转发给 _preRelayedCall 和 _postRelayedCall。

#### _rejectRelayedCall(uint256 errorCode) → uint256, bytes
内部#
在acceptRelayedCall中返回这个值以阻止执行Relayer调用。不会收取任何费用。

#### _computeCharge(uint256 gas, uint256 gasPrice, uint256 serviceFee) → uint256
内部#

#### RelayHubChanged(address oldRelayHub, address newRelayHub)
事件#
当合约将其*IRelayHub*合约更改为新合约时发出。

## Strategies

### GSNRecipientSignature
一种*GSN策略*，允许通过可信签名者的签名进行Relayer交易。意图是由一个在链下执行验证的服务器生成此签名。需要注意的是，在此方案中不向用户收取任何费用。因此，服务器应确保在其经济和威胁模型中考虑到这一点。

**FUNCTIONS**
constructor(trustedSigner)

acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, _)

{xref-GSNRecipientSignature-preRelayedCall}[_preRelayedCall()]

{xref-GSNRecipientSignature-postRelayedCall}[_postRelayedCall(, _, _, _)]

GSNRECIPIENT
getHubAddr()

_upgradeRelayHub(newRelayHub)

relayHubVersion()

_withdrawDeposits(amount, payee)

_msgSender()

_msgData()

preRelayedCall(context)

postRelayedCall(context, success, actualCharge, preRetVal)

_approveRelayedCall()

_approveRelayedCall(context)

_rejectRelayedCall(errorCode)

_computeCharge(gas, gasPrice, serviceFee)

**EVENTS**
GSNRECIPIENT
RelayHubChanged(oldRelayHub, newRelayHub)

#### constructor(address trustedSigner)
公开#
设置可信签名者，该签名者将负责批准Relayer调用的签名。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256) → uint256, bytes
内部#
确保只有带有可信签名的交易才能通过GSNRelayer。

#### _preRelayedCall(bytes) → bytes32
内部#

#### _postRelayedCall(bytes, bool, uint256, bytes32)
内部#

### GSNRecipientERC20Fee
一种GSN策略是使用一种特殊的ERC20代币作为交易费，我们称之为燃气支付代币。收取的费用金额恰好等于收款方所收取的以太币金额。这意味着该代币基本上与以太币的价值挂钩。

燃气支付代币的分发策略并未在该合约中定义。它是一种可铸造的代币，其唯一的铸币者是收款方，因此策略必须在派生合约中实现，并利用内部的_mint函数。

**FUNCTIONS**
constructor(name, symbol)

token()

_mint(account, amount)

acceptRelayedCall(_, from, _, transactionFee, gasPrice, _, _, _, maxPossibleCharge)

_preRelayedCall(context)

_postRelayedCall(context, _, actualCharge, _)

GSNRECIPIENT
getHubAddr()

_upgradeRelayHub(newRelayHub)

relayHubVersion()

_withdrawDeposits(amount, payee)

_msgSender()

_msgData()

preRelayedCall(context)

postRelayedCall(context, success, actualCharge, preRetVal)

_approveRelayedCall()

_approveRelayedCall(context)

_rejectRelayedCall(errorCode)

_computeCharge(gas, gasPrice, serviceFee)

**EVENTS**
GSNRECIPIENT
RelayHubChanged(oldRelayHub, newRelayHub)

#### constructor(string name, string symbol)
公开#
构造函数的参数是燃料支付代币将具有的细节：名称和符号。小数位数被硬编码为18。

#### token() → contract IERC20
公开#
返回燃气支付代币。

#### _mint(address account, uint256 amount)
内部#
内部函数，用于铸造燃气支付令牌。派生合约应该在其公共API中公开此函数，并配备适当的访问控制机制。

#### acceptRelayedCall(address, address from, bytes, uint256 transactionFee, uint256 gasPrice, uint256, uint256, bytes, uint256 maxPossibleCharge) → uint256, bytes
外部#
确保只有拥有足够的燃气支付代币余额的用户才能通过GSNRelayer交易。

#### _preRelayedCall(bytes context) → bytes32
内部#
对用户进行预充值。将从用户的燃料支付令牌余额中扣除最大可能的费用（取决于燃料限制、燃料价格和费用）。请注意，这是对实际费用的过估计，这是必要的，因为我们无法预测执行实际上需要多少燃料。剩余费用将在*_postRelayedCall*中返还给用户。

#### _postRelayedCall(bytes context, bool, uint256 actualCharge, bytes32)
内部#
一旦实际执行成本确定，将向用户退还之前多收取的金额。

## Protocol

### IRelayRecipient
一个通过GSN从*IRelayHub*调用的合约的基本接口。

> TIP
您不需要自己编写实现！请继承自*GSNRecipient*。

**FUNCTIONS**
getHubAddr()

acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)

preRelayedCall(context)

postRelayedCall(context, success, actualCharge, preRetVal)

#### getHubAddr() → address
内部#
返回此收件人与之交互的*IRelayHub*实例的地址。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256 maxPossibleCharge) → uint256, bytes
内部#
被*IRelayHub*调用以验证接收方是否同意为Relayer调用付费。请注意，无论Relayer调用的执行结果如何（即是否还原），接收方都将被收费。

Relayer请求由from发起，并将由relay提供服务。encodedFunction是Relayer调用的calldata，因此它的前四个字节是函数选择器。Relayer调用将转发gasLimit gas，并以至少gasPrice的燃料价格执行事务。relay的费用为transactionFee，并且接收方的最大可能费用为maxPossibleCharge（以wei为单位）。nonce是发送方（from）在*IRelayHub*中用于重放攻击保护的nonce，approvalData是一个可选参数，可用于保存对所有或部分先前值的签名。

返回一个元组，其中第一个值用于指示批准（0）或拒绝（自定义非零错误代码，保留值为1到10），第二个值是要传递给其他*IRelayRecipient*函数的数据。

*acceptRelayedCall*使用50k gas进行调用：如果在执行过程中用尽，请求将被视为被拒绝。正常的还原也会触发拒绝。

#### preRelayedCall(bytes context) → bytes32
内部#
在*IRelayHub*上调用已批准的Relayer调用请求之前，调用preRelayedCall。在执行Relayer调用之前，这允许例如预先向交易发送方收费。

context是*acceptRelayedCall*返回的元组的第二个值。

返回一个值以传递给*postRelayedCall*。

*preRelayedCall*使用100k gas进行调用：如果在执行过程中用完gas或者发生回滚，Relayer调用将不会被执行，但是接收方仍将被收取交易费用。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
内部#
在*IRelayHub*上的已批准的Relayer调用请求中调用，在Relayer调用执行后。这允许例如为Relayer调用的成本向用户收费，从*preRelayedCall*返回任何超额费用，或执行特定于合约的簿记。

context是*acceptRelayedCall*返回的元组的第二个值。success是Relayer调用的执行状态。actualCharge是接收方将为交易收取的估计费用，不包括*postRelayedCall*本身使用的任何gas。preRetVal是*preRelayedCall*的返回值。

*postRelayedCall*使用100k gas进行调用：如果在执行过程中用完gas或以其他方式回滚，则Relayer调用和对*preRelayedCall*的调用将被回滚，但接收方仍将被收取交易费用。

### IRelayHub
RelayHub是GSN的核心合约，用户不需要直接与该合约进行交互。

有关如何在本地测试网络上部署RelayHub实例的更多信息，请参阅[OpenZeppelin GSN助手](https://github.com/OpenZeppelin/openzeppelin-gsn-helpers)。

**FUNCTIONS**
stake(relayaddr, unstakeDelay)

registerRelay(transactionFee, url)

removeRelayByOwner(relay)

unstake(relay)

getRelay(relay)

depositFor(target)

balanceOf(target)

withdraw(amount, dest)

canRelay(relay, from, to, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, signature, approvalData)

relayCall(from, to, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, signature, approvalData)

requiredGas(relayedCallStipend)

maxPossibleCharge(relayedCallStipend, gasPrice, transactionFee)

penalizeRepeatedNonce(unsignedTx1, signature1, unsignedTx2, signature2)

penalizeIllegalTransaction(unsignedTx, signature)

getNonce(from)

**EVENTS**
Staked(relay, stake, unstakeDelay)

RelayAdded(relay, owner, transactionFee, stake, unstakeDelay, url)

RelayRemoved(relay, unstakeTime)

Unstaked(relay, stake)

Deposited(recipient, from, amount)

Withdrawn(account, dest, amount)

CanRelayFailed(relay, from, to, selector, reason)

TransactionRelayed(relay, from, to, selector, status, charge)

Penalized(relay, sender, amount)

#### stake(address relayaddr, uint256 unstakeDelay)
外部#
向Relayer添加股份并设置其解锁延迟。如果Relayer不存在，则创建该Relayer，并且调用此函数的调用者成为其所有者。如果Relayer已经存在，则只有所有者可以调用此函数。Relayer不能成为自己的所有者。

此函数调用中的所有以太币将添加到Relayer的股份中。其解锁延迟将分配给unstakeDelay，但新值必须大于或等于当前值。

发出*Staked*事件。

#### registerRelay(uint256 transactionFee, string url)
外部#
将调用者注册为Relayer。Relayer必须进行抵押，并且不能是合约（即此函数必须直接从EOA调用）。

此函数可以多次调用，发出新的*RelayAdded*事件。请注意，*relayCall*不强制执行接收的transactionFee。

发出*RelayAdded*事件。

#### removeRelayByOwner(address relay)
外部#
移除（注销）一个Relayer。已注销（但已抵押）的Relayer也可以被移除。

只能由Relayer的所有者调用。在Relayer的解押延迟过去后，可以调用解押。

发出一个*RelayRemoved*事件。

#### unstake(address relay)
外部#

#### getRelay(address relay) → uint256 totalStake, uint256 unstakeDelay, uint256 unstakeTime, address payable owner, enum IRelayHub.RelayState state
外部#
返回Relayer的状态。请注意，Relayer可能会在解除质押或受到惩罚时被删除，导致此函数返回一个空条目。

#### depositFor(address target)
外部#

#### balanceOf(address target) → uint256
外部#
返回一个账户的存款。这些存款可以是合约的资金，也可以是Relayer所有者的收入。

#### withdraw(uint256 amount, address payable dest)
外部#

#### canRelay(address relay, address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData) → uint256 status, bytes recipientContext
外部#
检查RelayHub是否接受Relayer操作。为了使此操作成功，必须满足多个条件：- 所有参数必须由发送方（from）签名- 发送方的nonce必须是当前的- 接收方必须接受此交易（通过*acceptRelayedCall*）

返回一个PreconditionCheck值（当交易可以被Relayer时为OK），或者如果*acceptRelayedCall*返回一个接收方特定的错误代码，则返回该代码。

#### relayCall(address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData)
外部#
转发交易。

为了成功转发交易，必须满足多个条件： - *canRelay*必须返回PreconditionCheck.OK - 发送者必须是注册的Relayer - 交易的燃气价格必须大于或等于发送者请求的燃气价格 - 如果所有内部交易（对接收者的调用）使用了所有可用的燃气，交易必须有足够的燃气以防止燃气耗尽 - 接收者必须有足够的余额来支付最坏情况下的Relayer费用（即当所有燃气都用完时）

如果满足所有条件，将转发调用并向接收者收费。按照以下顺序调用*preRelayedCall*、编码函数和*postRelayedCall*。

参数： - from：发起请求的客户端 - to：目标*IRelayRecipient*合约 - encodedFunction：要转发的函数调用，包括数据 - transactionFee：Relayer接管的实际燃气成本的费用（%） - gasPrice：客户端愿意支付的燃气价格 - gasLimit：在调用编码函数时要转发的燃气 - nonce：客户端的nonce - signature：客户端对所有先前参数的签名，加上Relayer和RelayHub地址 - approvalData：转发给*acceptRelayedCall*的特定于dapp的数据。RelayHub不验证此值，但仍然可以用于例如签名。

发出*TransactionRelayed*事件。

#### requiredGas(uint256 relayedCallStipend) → uint256
外部#
在调用*relayCall*时，返回应该转发多少gas，以便Relayer交易最多花费relayedCallStipend个gas。

#### maxPossibleCharge(uint256 relayedCallStipend, uint256 gasPrice, uint256 transactionFee) → uint256
外部#
给定转发的气体数量、气体价格和Relayer费用，返回最大的接收方费用。

#### penalizeRepeatedNonce(bytes unsignedTx1, bytes signature1, bytes unsignedTx2, bytes signature2)
外部#
惩罚一个Relayer节点，该节点使用相同的nonce签署了两笔交易（只有第一笔交易有效），但数据不同（燃气价格、燃气限制等可能不同）。

必须提供两笔交易的（未签名的）交易数据和签名。

#### penalizeIllegalTransaction(bytes unsignedTx, bytes signature)
外部#
对于发送了不针对`RelayHub`的*`registerRelay`*或*`relayCall`*的事务的Relayer进行惩罚。

#### getNonce(address from) → uint256
外部#
在RelayHub中返回一个账户的nonce。

#### Staked(address relay, uint256 stake, uint256 unstakeDelay)
事件#
当Relayer的质押或解质押延迟增加时发出

#### RelayAdded(address relay, address owner, uint256 transactionFee, uint256 stake, uint256 unstakeDelay, string url)
事件#
当一个继电器被注册或重新注册时发出。通过查看这些事件（并过滤掉*RelayRemoved*事件），客户端可以发现可用继电器的列表。

#### RelayRemoved(address relay, uint256 unstakeTime)
事件#
当Relayer器被移除（取消注册）时发出。unstakeTime是可以调用解除质押的时间。

#### Unstaked(address relay, uint256 stake)
事件#
当一个Relayer被解除质押时，包括返回的质押金额时发出。

#### Deposited(address recipient, address from, uint256 amount)
事件#
在调用*depositFor*时发出，包括被资助的金额和账户。

#### Withdrawn(address account, address dest, uint256 amount)
事件#
当一个账户从RelayHub中提取资金时发出。

#### CanRelayFailed(address relay, address from, address to, bytes4 selector, uint256 reason)
事件#
当Relayer调用失败时发出。

这可能是由于错误的*relayCall*参数或接收方不接受Relayer调用引起的。实际的Relayer调用未执行，并且接收方未收费。

reason参数包含一个错误代码：值1-10对应于PreconditionCheck条目，而大于10的值是从*acceptRelayedCall*返回的自定义接收方错误代码。

#### TransactionRelayed(address relay, address from, address to, bytes4 selector, enum IRelayHub.RelayCallStatus status, uint256 charge)
事件#
当一个交易被Relayer时发出。在监控Relayer操作和Relayer调用合约时非常有用。

请注意，实际编码的函数可能会被还原：这在状态参数中表示。

charge是从接收者的余额中扣除的以太币值，支付给Relayer的所有者。

#### Penalized(address relay, address sender, uint256 amount)
事件#
当继电器受到惩罚时发出的信号。