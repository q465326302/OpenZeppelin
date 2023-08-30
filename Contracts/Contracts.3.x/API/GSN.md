# Gas Station Network (GSN)
这套合约提供了通过[Gas Station Network（GSN）](https://gsn.openzeppelin.com/)调用合约所需的所有工具。

> TIP
如果您对GSN还不熟悉，请查看我们关于该[系统的概述](../../../Learn和*创建GSN兼容合约的基本指南*。

接收方必须继承的核心合约是*GSNRecipient*：它包括所有必要的接口，以及一些帮助方法，使与GSN的交互更容易。

使编写*GSN策略*变得简单的实用程序可在*GSNRecipient*中使用，或者您可以直接使用我们预先制作的策略之一：

* *GSNRecipientERC20Fee* 使用特定应用程序的*ERC20代币*为最终用户收取燃气费用

* *GSNRecipientSignature* 接受已由可信第三方（例如后端中的私钥）签名的所有Relayer 调用

您还可以查看构成GSN协议的两个合约接口：*IRelayRecipient*和*IRelayHub*，但您不需要直接使用它们。

## 接收方

### GSNRecipient

基本GSN接收方合约：包括*IRelayRecipient*接口，并在继承树中的所有合约上启用GSN支持。

> TIP
此合约是抽象的。未实现*IRelayRecipient.acceptRelayedCall*、*_preRelayedCall*和*_postRelayedCall*函数，必须由派生合约提供。有关如何使用预构建的*GSNRecipientSignature*和*GSNRecipientERC20Fee*的更多信息，请参阅*GSN策略*，或者如何编写您自己的策略。

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

IRELAYRECIPIENT
acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)

**EVENTS**
RelayHubChanged(oldRelayHub, newRelayHub)

#### getHubAddr() → address
公开#
返回此收件人的*IRelayHub*合约的地址。

#### _upgradeRelayHub(address newRelayHub)
内部#
切换到新的*IRelayHub*实例。此方法是为了未来的保护措施：没有理由不使用默认实例。

> IMPORTANT
升级后，*GSNRecipient*将无法从旧的*IRelayHub*实例接收Relayer 调用。此外，所有资金应通过*_withdrawDeposits*提前提取。

#### relayHubVersion() → string
公开#
返回此接收方实现版本的*IRelayHub*版本字符串。如果使用*_upgradeRelayHub*，则新的*IRelayHub*实例应与此版本兼容。

#### _withdrawDeposits(uint256 amount, address payable payee)
内部#
从RelayHub中撤回收款人的存款。

衍生合约应该以外部接口的形式公开这一操作，并进行适当的访问控制。

#### _msgSender() → address payable
内部#
替代msg.sender的方法。返回交易的实际发送者：对于普通交易，返回msg.sender，对于GSNRelayer 调用，返回最终用户（其中msg.sender实际上是RelayHub）。

> IMPORTANT
从*GSNRecipient*派生的合约不应该使用msg.sender，而应该使用*_msgSender*。

#### _msgData() → bytes
替代msg.data。返回交易的实际calldata：对于常规交易，返回msg.data，对于GSNRelayer 调用（其中msg.data包含其他信息），返回简化版本。

> 从*GSNRecipient*派生的合约不应使用msg.data，而应使用*_msgData*。

#### preRelayedCall(bytes context) → bytes32
公开#
请查看IRelayRecipient.preRelayedCall。

不应直接重写此函数，请使用_preRelayedCall代替。

* 要求：
    * 调用者必须是RelayHub合约。

#### _preRelayedCall(bytes context) → bytes32
内部#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，用于断言调用者是RelayHub合约。派生合约必须实现此函数以执行任何他们希望进行的Relayer 调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
公开#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer 调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
公开#
请参阅IRelayRecipient.postRelayedCall。

此函数不应直接被重写，而应使用_postRelayedCall。

* 要求：
    * 调用者必须是RelayHub合约。

#### _postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
内部#
请参阅IRelayRecipient.postRelayedCall。

通过GSNRecipient.postRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer 调用后处理。

#### _approveRelayedCall() → uint256, bytes
内部#
返回此值以继续执行Relayer 调用。请注意，RelayHub将向此合约收取费用。

#### _approveRelayedCall(bytes context) → uint256, bytes
内部#
请参阅 GSNRecipient._approveRelayedCall。

这个重载将上下文传递给 _preRelayedCall 和 _postRelayedCall。

#### _rejectRelayedCall(uint256 errorCode) → uint256, bytes
内部#
在acceptRelayedCall中返回此内容以阻止执行Relayer 调用。不会收取任何费用。

#### _computeCharge(uint256 gas, uint256 gasPrice, uint256 serviceFee) → uint256
内部#

#### RelayHubChanged(address oldRelayHub, address newRelayHub)
事件#
当合约将其 *IRelayHub* 合约更改为新的合约时发出。

## Strategies
### GSNRecipientSignature
一种*GSN策略*，允许在伴有受信任签名者签名的情况下进行Relayer 交易。这个签名的生成是由一个在链下执行验证的服务器完成的。需要注意的是，在这个方案中用户不会被收费。因此，服务器在经济和威胁模型中应该考虑到这一点。

**FUNCTIONS**
constructor(trustedSigner)

acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, _)

_preRelayedCall(_)

_postRelayedCall(_, _, _, _)

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
设置可信的签名者，该签名者将会产生用于批准Relayer 调用的签名。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256) → uint256, bytes
公开#
确保只有带有可信签名的交易才能通过GSNRelayer 。

#### _preRelayedCall(bytes) → bytes32
内部#

#### _postRelayedCall(bytes, bool, uint256, bytes32)
内部#

### GSNRecipientERC20Fee
一种*GSN策略*是以一种特殊目的的ERC20代币收取交易费用，我们将其称为燃气支付代币。所收取的金额恰好等于收款方收取的以太币金额。这意味着该代币基本上与以太币的价值挂钩。

燃气支付代币向用户的分发策略没有在该合约中定义。它是一种可铸造的代币，其唯一的铸造者是收款方，因此该策略必须在派生合约中实施，利用内部的*_mint*函数。

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
构造函数的参数是燃料支付代币的详细信息：名称和符号。小数位数被硬编码为18。

#### token() → contract __unstable__ERC20Owned
公开#
返回燃气支付代币。

#### _mint(address account, uint256 amount)
内部#
内部函数用于铸造燃气支付代币。派生合约应该在其公共API中公开此函数，并配备适当的访问控制机制。

#### acceptRelayedCall(address, address from, bytes, uint256 transactionFee, uint256 gasPrice, uint256, uint256, bytes, uint256 maxPossibleCharge) → uint256, bytes
公开#
确保只有拥有足够的Gas支付代币余额的用户才能通过GSNRelayer 交易。

#### _preRelayedCall(bytes context) → bytes32
内部#
对用户进行预付费。最大可能的费用（取决于燃气限制、燃气价格和费用）将从用户的燃气支付代币余额中扣除。请注意，这是对实际费用的过估计，这是因为我们无法预测执行实际需要多少燃气。剩余部分将在*_postRelayedCall*中退还给用户。

#### _postRelayedCall(bytes context, bool, uint256 actualCharge, bytes32)
内部#
一旦实际执行成本知道，将向用户返还之前多收取的金额。

## 协议

### IRelayRecipient
一个合约的基本接口，将通过*IRelayHub*从GSN调用。

> TIP
您不需要自己编写实现！请继承自*GSNRecipient*。

**FUNCTIONS**
getHubAddr()

acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)

preRelayedCall(context)

postRelayedCall(context, success, actualCharge, preRetVal)

#### getHubAddr() → address
外部#
返回此收件人与之交互的*IRelayHub*实例的地址。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256 maxPossibleCharge) → uint256, bytes
外部#
*IRelayHub*调用acceptRelayedCall方法来验证接收者是否同意为Relayer 调用支付费用。需要注意的是，无论Relayer 调用的执行结果如何（即是否回滚），接收者都将被收取费用。

Relayer 请求由from发起，并将由relay提供服务。encodedFunction是Relayer 调用的calldata，因此它的前四个字节是函数选择器。Relayer 调用将转发gasLimit gas，并以至少gasPrice的燃料价格执行事务。relay的费用是transactionFee，接收者的最大可能费用为maxPossibleCharge（以wei为单位）。nonce是发送者（from）在*IRelayHub*中用于防止重放攻击的nonce，approvalData是一个可选参数，可以用于保存对所有或部分先前值的签名。

返回一个元组，其中第一个值用于表示批准（0）或拒绝（自定义非零错误代码，值为1到10保留），第二个值是要传递给其他*IRelayRecipient*函数的数据。

*acceptRelayedCall*方法使用50k gas进行调用：如果在执行过程中用尽，则该请求将被视为被拒绝。正常的回滚也会触发拒绝。

#### preRelayedCall(bytes context) → bytes32
外部#
在被*IRelayHub*批准的Relayer 调用请求上调用，在执行Relayer 调用之前。这允许例如预先收取交易发送者的费用。

上下文是通过*acceptRelayedCall*返回的元组的第二个值。

返回一个值以传递给*postRelayedCall*。

*preRelayedCall*使用100k gas进行调用：如果在执行过程中用尽gas或者发生回滚，Relayer 调用将不会被执行，但是收件人仍然需要支付交易的费用。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
外部#
在*IRelayHub*对已批准的Relayer 调用请求进行调用后，Relayer 调用执行完成。这允许例如收取用户Relayer 调用费用、返回*preRelayedCall*的超额收费或执行与合约特定的簿记。

context是通过*acceptRelayedCall*返回的元组的第二个值。success是Relayer 调用的执行状态。actualCharge是接收者将为交易支付的估计费用，不包括*postRelayedCall*本身使用的任何gas。preRetVal是*preRelayedCall*的返回值。

*postRelayedCall*使用100k gas进行调用：如果在执行过程中gas耗尽或发生其他还原，Relayer 调用和对*preRelayedCall*的调用将被还原，但接收者仍然需要支付交易的费用。

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
向Relayer 添加股份并设置其解锁延迟。如果Relayer 不存在，则创建它，并且调用此函数的调用者成为其所有者。如果Relayer 已经存在，则只有所有者可以调用此函数。Relayer 不能成为自己的所有者。

此函数调用中的所有以太币将被添加到Relayer 的股份中。其解锁延迟将分配给unstakeDelay，但新值必须大于或等于当前值。

触发一个*Staked*事件。

#### registerRelay(uint256 transactionFee, string url)
外部# 
将调用者注册为Relayer 。Relayer 必须进行抵押，并且不能是合约（即必须直接从EOA调用此函数）。

此函数可以被多次调用，每次调用都会发出新的*RelayAdded*事件。注意，*relayCall*不强制执行接收到的transactionFee。

发出一个*RelayAdded*事件。

#### removeRelayByOwner(address relay)
外部# 
移除（注销）一个Relayer 。已注销（但已抵押）的Relayer 也可以被移除。

只能由Relayer 的所有者调用。在Relayer 的解除锁定延迟过后，可以调用解锁（*unstake*）函数。

发出一个*RelayRemoved*事件。

#### unstake(address relay)
外部# 

#### getRelay(address relay) → uint256 totalStake, uint256 unstakeDelay, uint256 unstakeTime, address payable owner, enum IRelayHub.RelayState state
外部# 
返回Relayer 的状态。请注意，Relayer 在解除抵押或受到惩罚时可以被删除，导致此函数返回一个空条目。

#### depositFor(address target)
外部# 
存入以太币到合约中，以便能够接收（和支付）Relayer 的交易。

未使用的余额只能通过合约本身调用*withdraw*来提取。

触发一个*Deposited*事件。

#### balanceOf(address target) → uint256
外部# 
返回一个账户的存款。这些存款可以是合约的资金，也可以是Relayer 所有者的收入。

#### withdraw(uint256 amount, address payable dest)
外部# 

#### canRelay(address relay, address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData) → uint256 status, bytes recipientContext
外部#
检查RelayHub是否接受Relayer 操作。为了实现这一点，必须满足多个条件： - 所有参数必须由发送者（from）签名 - 发送者的nonce必须是当前的nonce - 收件人必须接受此交易（通过*acceptRelayedCall*）

返回PreconditionCheck值（如果可以Relayer 交易则返回OK），或者如果在*acceptRelayedCall*中返回了收件人特定的错误代码，则返回该错误代码。

#### relayCall(address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData)
外部#
转发一个交易。

为了成功，必须满足多个条件：- *canRelay*必须返回PreconditionCheck.OK- 发送方必须是注册的Relayer - 交易的燃气价格必须大于或等于发送方请求的燃气价格- 如果所有内部交易（对接收方的调用）使用了它们可用的所有燃气，则交易必须有足够的燃气，以免燃气耗尽- 接收方必须有足够的余额来支付最坏情况下的Relayer 费用（即当所有燃气都用尽时）

如果所有条件都满足，将转发调用并向接收方收费。按顺序调用*preRelayedCall*、编码函数和*postRelayedCall*。

参数：- from：发起请求的客户端- to：目标*IRelayRecipient*合约- encodedFunction：要转发的函数调用，包括数据- transactionFee：Relayer 接管的实际燃气成本的费用（%）- gasPrice：客户端愿意支付的燃气价格- gasLimit：调用编码函数时要转发的燃气- nonce：客户端的nonce- signature：客户端对所有先前参数的签名，加上Relayer 和RelayHub地址- approvalData：转发到*acceptRelayedCall*的特定于dapp的数据。RelayHub不验证此值，但仍然可以用于例如签名。

发出*TransactionRelayed*事件。

#### requiredGas(uint256 relayedCallStipend) → uint256
外部#
返回应向*relayCall*的调用转发多少gas，以便Relayer 事务可以花费多达relayedCallStipend gas。

#### maxPossibleCharge(uint256 relayedCallStipend, uint256 gasPrice, uint256 transactionFee) → uint256
外部#
根据转发的燃料量、燃料价格和Relayer 费用，返回最大的接收者费用。

#### penalizeRepeatedNonce(bytes unsignedTx1, bytes signature1, bytes unsignedTx2, bytes signature2)
外部#
对使用相同nonce签署了两个交易的Relayer 进行惩罚（只有第一个交易有效），并且数据不同（例如，燃料价格、燃料限制等可能不同）。

必须提供两个交易的（未签名的）交易数据和签名。

#### penalizeIllegalTransaction(bytes unsignedTx, bytes signature)
外部#
对于发送了未针对RelayHub的*registerRelay*或*relayCall*的交易的Relayer 进行惩罚。

#### getNonce(address from) → uint256
外部#
在RelayHub中返回账户的nonce。

#### Staked(address relay, uint256 stake, uint256 unstakeDelay)
事件# 
当 Relayer 的质押或取消质押延迟增加时发出

#### RelayAdded(address relay, address owner, uint256 transactionFee, uint256 stake, uint256 unstakeDelay, string url)
事件# 
当一个Relayer 被注册或重新注册时发出。查看这些事件（并过滤掉*RelayRemoved*事件）可以让客户端发现可用的Relayer 列表。

#### RelayRemoved(address relay, uint256 unstakeTime)
事件# 
当继电器被移除（取消注册）时发出。unstakeTime是解除质押可调用的时间。

#### Unstaked(address relay, uint256 stake)
事件# 
当一个Relay取消质押时，包括返回的质押金额时发出。

#### Deposited(address recipient, address from, uint256 amount)
事件#

当调用*depositFor*时发出，包括所存入的金额和账户。

#### Withdrawn(address account, address dest, uint256 amount)
事件#
当一个账户从RelayHub中提取资金时发出。

#### CanRelayFailed(address relay, address from, address to, bytes4 selector, uint256 reason)
事件#
当尝试Relayer 呼叫失败时发出。

这可能是由于错误的*relayCall*参数或接收方不接受Relayer 呼叫。实际的Relayer 呼叫未执行，并且接收方未收费。

reason参数包含一个错误代码：值1-10对应于PreconditionCheck条目，而大于10的值是从*acceptRelayedCall*返回的自定义接收方错误代码。

#### TransactionRelayed(address relay, address from, address to, bytes4 selector, enum IRelayHub.RelayCallStatus status, uint256 charge)
事件#
当一个交易被Relayer 时发出。在监控Relayer 的操作和Relayer 调用合约时非常有用。

请注意，实际的编码函数可能会被还原：这在状态参数中表示。

charge 是从接收者的余额中扣除的以太币价值，支付给Relayer 的所有者。

#### Penalized(address relay, address sender, uint256 amount)
事件#
当继电器受到惩罚时发出的信号。