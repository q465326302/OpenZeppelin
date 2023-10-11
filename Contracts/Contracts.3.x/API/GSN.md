# Gas Station Network (GSN)
这套合约提供了通过[Gas Station Network（GSN）](https://gsn.openzeppelin.com/)调用合约所需的所有工具。

> TIP
如果你对GSN还不熟悉，请查看我们关于该[系统的概述](../../../Learn/Sending%20gasless%20transactions/Sending-gasless-transactions.md)和[创建GSN兼容合约的基本指南](../Gas-Station-Network/gsn.md)。

接收方必须继承的核心合约是[GSNRecipient](#gsnrecipient)：它包括所有必要的接口，以及一些帮助方法，使与GSN的交互更容易。

使编写[GSN strategies](../Gas-Station-Network/Strategies.md)变得简单的实用程序可在[GSNRecipient](#gsnrecipient)中使用，或者你可以直接使用我们预先制作的策略之一：

* [GSNRecipientERC20Fee](#gsnrecipienterc20fee) 使用特定应用程序的*ERC20代币*为最终用户收取gas费用

* [GSNRecipientSignature](#gsnrecipientsignature) 接受已由可信第三方（例如后端中的私钥）签名的所有Relayer 调用

你还可以查看构成GSN协议的两个合约接口：[IRelayRecipient](#irelayrecipient)和[IRelayHub](#irelayhub)，但你不需要直接使用它们。

## Recipient

### GSNRecipient

基本GSN接收方合约：包括[IRelayRecipient](#irelayrecipient)接口，并在继承树中的所有合约上启用GSN支持。

> TIP
此合约是抽象的。未实现[IRelayRecipient.acceptRelayedCall](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)、[_preRelayedCall](#_prerelayedcallbytes-context-→-bytes32)和_postRelayedCall函数，必须由派生合约提供。有关如何使用预构建的*GSNRecipientSignature*和*GSNRecipientERC20Fee*的更多信息，请参阅*GSN策略*，或者如何编写你自己的策略。

**FUNCTIONS**
[getHubAddr()](#gethubaddr-→-address)

[_upgradeRelayHub(newRelayHub)](#_upgraderelayhubaddress-newrelayhub)

[relayHubVersion()](#relayhubversion-→-string)

[_withdrawDeposits(amount, payee)](#_withdrawdepositsuint256-amount-address-payable-payee)

[_msgSender()](#_msgsender-→-address-payable)

[_msgData()](#_msgdata-→-bytes)

[preRelayedCall(context)](#prerelayedcallbytes-context-→-bytes32)

[_preRelayedCall(context)](#_prerelayedcallbytes-context-→-bytes32)

[postRelayedCall(context, success, actualCharge, preRetVal)](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval)

[_postRelayedCall(context, success, actualCharge, preRetVal)](#_postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval)

[_approveRelayedCall()](#_approverelayedcall-→-uint256-bytes)

[_approveRelayedCall(context)](#_approverelayedcallbytes-context-→-uint256-bytes)

[_rejectRelayedCall(errorCode)](#_rejectrelayedcalluint256-errorcode-→-uint256-bytes)

[_computeCharge(gas, gasPrice, serviceFee)](#_computechargeuint256-gas-uint256-gasprice-uint256-servicefee-→-uint256)

IRELAYRECIPIENT
[acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)

**EVENTS**
[RelayHubChanged(oldRelayHub, newRelayHub)](#relayhubchangedaddress-oldrelayhub-address-newrelayhub)

#### getHubAddr() → address
public#
返回此收件人的[IRelayHub](#irelayhub)合约的地址。

#### _upgradeRelayHub(address newRelayHub)
internal#
切换到新的[IRelayHub](#irelayhub)实例。此方法是为了未来的保护措施：没有理由不使用默认实例。

> IMPORTANT
升级后，[GSNRecipient](#gsnrecipient)将无法从旧的[IRelayHub](#irelayhub)实例接收Relayer 调用。此外，所有资金应通过[_withdrawDeposits](#_withdrawdepositsuint256-amount-address-payable-payee)提前提取。

#### relayHubVersion() → string
public#
返回此接收方实现版本的[IRelayHub](#irelayhub)版本字符串。如果使用[_upgradeRelayHub](#_upgraderelayhubaddress-newrelayhub)，则新的[IRelayHub](#irelayhub)实例应与此版本兼容。

#### _withdrawDeposits(uint256 amount, address payable payee)
internal#
从RelayHub中撤回收款人的存款。

衍生合约应该以外部接口的形式公开这一操作，并进行适当的访问控制。

#### _msgSender() → address payable
internal#
替代msg.sender的方法。返回交易的实际发送者：对于普通交易，返回msg.sender，对于GSNRelayer 调用，返回最终用户（其中msg.sender实际上是RelayHub）。

> IMPORTANT
从[GSNRecipient](#gsnrecipient)派生的合约不应该使用msg.sender，而应该使用[_msgSender](#_msgsender-→-address-payable)。

#### _msgData() → bytes
替代msg.data。返回交易的实际calldata：对于常规交易，返回msg.data，对于GSNRelayer 调用（其中msg.data包含其他信息），返回简化版本。

> 从*GSNRecipient*派生的合约不应使用msg.data，而应使用*_msgData*。

#### preRelayedCall(bytes context) → bytes32
public#
请查看IRelayRecipient.preRelayedCall。

不应直接重写此函数，请使用_preRelayedCall代替。

* 要求：
    * 调用者必须是RelayHub合约。

#### _preRelayedCall(bytes context) → bytes32
internal#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，用于断言调用者是RelayHub合约。派生合约必须实现此函数以执行任何他们希望进行的Relayer 调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
public#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer 调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
public#
请参阅IRelayRecipient.postRelayedCall。

此函数不应直接被重写，而应使用_postRelayedCall。

* 要求：
    * 调用者必须是RelayHub合约。

#### _postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
internal#
请参阅IRelayRecipient.postRelayedCall。

通过GSNRecipient.postRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer 调用后处理。

#### _approveRelayedCall() → uint256, bytes
internal#
返回此值以继续执行Relayer 调用。请注意，RelayHub将向此合约收取费用。

#### _approveRelayedCall(bytes context) → uint256, bytes
internal#
请参阅 GSNRecipient._approveRelayedCall。

这个重载将上下文传递给 _preRelayedCall 和 _postRelayedCall。

#### _rejectRelayedCall(uint256 errorCode) → uint256, bytes
internal#
在acceptRelayedCall中返回此内容以阻止执行Relayer 调用。不会收取任何费用。

#### _computeCharge(uint256 gas, uint256 gasPrice, uint256 serviceFee) → uint256
internal#

#### RelayHubChanged(address oldRelayHub, address newRelayHub)
event#
当合约将其 [IRelayHub](#irelayhub) 合约更改为新的合约时发出。

## Strategies
### GSNRecipientSignature
一种[GSN策略](../Gas-Station-Network/Strategies.md)，允许在伴有受信任签名者签名的情况下进行Relayer 交易。这个签名的生成是由一个在链下执行验证的服务器完成的。需要注意的是，在这个方案中用户不会被收费。因此，服务器在经济和威胁模型中应该考虑到这一点。

**FUNCTIONS**
[constructor(trustedSigner)](#constructoraddress-trustedsigner)

[acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, _)](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)

[_preRelayedCall(_)](#_prerelayedcallbytes-→-bytes32)

[_postRelayedCall(_, _, _, _)](#_postrelayedcallbytes-bool-uint256-bytes32)

GSNRECIPIENT
[getHubAddr()](#gethubaddr-→-address)

[_upgradeRelayHub(newRelayHub)](#_upgraderelayhubaddress-newrelayhub)

[relayHubVersion()](#relayhubversion-→-string)

[_withdrawDeposits(amount, payee)](#_withdrawdepositsuint256-amount-address-payable-payee)

[_msgSender()](#_msgsender-→-address-payable)

[_msgData()](#_msgdata-→-bytes)

[preRelayedCall(context)](#prerelayedcallbytes-context-→-bytes32)

[postRelayedCall(context, success, actualCharge, preRetVal)](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval)

[_approveRelayedCall()](#_approverelayedcall-→-uint256-bytes)

[_approveRelayedCall(context)](#_approverelayedcallbytes-context-→-uint256-bytes)

[_rejectRelayedCall(errorCode)](#_rejectrelayedcalluint256-errorcode-→-uint256-bytes)

[_computeCharge(gas, gasPrice, serviceFee)](#_computechargeuint256-gas-uint256-gasprice-uint256-servicefee-→-uint256)

**EVENTS**
[RelayHubChanged(oldRelayHub, newRelayHub)](#relayhubchangedaddress-oldrelayhub-address-newrelayhub)

#### constructor(address trustedSigner)
public#
设置可信的签名者，该签名者将会产生用于批准Relayer 调用的签名。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256) → uint256, bytes
public#
确保只有带有可信签名的交易才能通过GSNRelayer 。

#### _preRelayedCall(bytes) → bytes32
internal#

#### _postRelayedCall(bytes, bool, uint256, bytes32)
internal#

### GSNRecipientERC20Fee
一种[GSN策略](../Gas-Station-Network/Strategies.md)是以一种特殊目的的ERC20代币收取交易费用，我们将其称为gas支付代币。所收取的金额恰好等于收款方收取的以太金额。这意味着该代币基本上与以太的价值挂钩。

gas支付代币向用户的分发策略没有在该合约中定义。它是一种可铸造的代币，其唯一的铸造者是收款方，因此该策略必须在派生合约中实施，利用内部的[_mint](#_mintaddress-account-uint256-amount)函数。

**FUNCTIONS**
[constructor(name, symbol)](#constructorstring-name-string-symbol)

[token()](#token-→-contract-__unstable__erc20owned)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[acceptRelayedCall(_, from, _, transactionFee, gasPrice, _, _, _, maxPossibleCharge)](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)

[_preRelayedCall(context)](#_prerelayedcallbytes-context-e28692-bytes32-1)

[_postRelayedCall(context, _, actualCharge, _)](#_postrelayedcallbytes-context-bool-uint256-actualcharge-bytes32)

GSNRECIPIENT
[getHubAddr()](#gethubaddr-→-address)

[_upgradeRelayHub(newRelayHub)](#_upgraderelayhubaddress-newrelayhub)

[relayHubVersion()](#relayhubversion-→-string)

[_withdrawDeposits(amount, payee)](#_withdrawdepositsuint256-amount-address-payable-payee)

[_msgSender()](#_msgsender-→-address-payable)

[_msgData()](#_msgdata-→-bytes)

[preRelayedCall(context)](#prerelayedcallbytes-context-→-bytes32)

[postRelayedCall(context, success, actualCharge, preRetVal)](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval)

[_approveRelayedCall()](#_approverelayedcall-→-uint256-bytes)

[_approveRelayedCall(context)](#_approverelayedcallbytes-context-→-uint256-bytes)

[_rejectRelayedCall(errorCode)](#_rejectrelayedcalluint256-errorcode-→-uint256-bytes)

[_computeCharge(gas, gasPrice, serviceFee)](#_computechargeuint256-gas-uint256-gasprice-uint256-servicefee-→-uint256)

**EVENTS**
[RelayHubChanged(oldRelayHub, newRelayHub)](#relayhubchangedaddress-oldrelayhub-address-newrelayhub)

#### constructor(string name, string symbol)
public#
构造函数的参数是燃料支付代币的详细信息：名称和符号。小数位数被硬编码为18。

#### token() → contract __unstable__ERC20Owned
public#
返回gas支付代币。

#### _mint(address account, uint256 amount)
internal#
内部函数用于铸造gas支付代币。派生合约应该在其公共API中公开此函数，并配备适当的访问控制机制。

#### acceptRelayedCall(address, address from, bytes, uint256 transactionFee, uint256 gasPrice, uint256, uint256, bytes, uint256 maxPossibleCharge) → uint256, bytes
public#
确保只有拥有足够的Gas支付代币余额的用户才能通过GSNRelayer 交易。

#### _preRelayedCall(bytes context) → bytes32
internal#
对用户进行预付费。最大可能的费用（取决于gas限制、gas价格和费用）将从用户的gas支付代币余额中扣除。请注意，这是对实际费用的过估计，这是因为我们无法预测执行实际需要多少gas。剩余部分将在[_postRelayedCall](#_postrelayedcallbytes-context-bool-uint256-actualcharge-bytes32)中退还给用户。

#### _postRelayedCall(bytes context, bool, uint256 actualCharge, bytes32)
internal#
一旦实际执行成本知道，将向用户返还之前多收取的金额。

## Protocol

### IRelayRecipient
一个合约的基本接口，将通过[IRelayHub](#irelayhub)从GSN调用。

> TIP
你不需要自己编写实现！请继承自[GSNRecipient](#gsnrecipient)。

**FUNCTIONS**
[getHubAddr()](#gethubaddr-e28692-address-1)

[acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)

[preRelayedCall(context)](#prerelayedcallbytes-context-e28692-bytes32-1)

[postRelayedCall(context, success, actualCharge, preRetVal)](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-2)

#### getHubAddr() → address
外部#
返回此收件人与之交互的[IRelayHub](#irelayhub)实例的地址。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256 maxPossibleCharge) → uint256, bytes
外部#
[IRelayHub](#irelayhub)调用acceptRelayedCall方法来验证接收者是否同意为Relayer 调用支付费用。需要注意的是，无论Relayer 调用的执行结果如何（即是否回滚），接收者都将被收取费用。

Relayer 请求由from发起，并将由relay提供服务。encodedFunction是Relayer 调用的calldata，因此它的前四个字节是函数选择器。Relayer 调用将转发gasLimit gas，并以至少gasPrice的燃料价格执行事务。relay的费用是transactionFee，接收者的最大可能费用为maxPossibleCharge（以wei为单位）。nonce是发送者（from）在[IRelayHub](#irelayhub)中用于防止重放攻击的nonce，approvalData是一个可选参数，可以用于保存对所有或部分先前值的签名。

返回一个元组，其中第一个值用于表示批准（0）或拒绝（自定义非零错误代码，值为1到10保留），第二个值是要传递给其他[IRelayRecipient](#irelayrecipient)函数的数据。

[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)方法使用50k gas进行调用：如果在执行过程中用尽，则该请求将被视为被拒绝。正常的回滚也会触发拒绝。

#### preRelayedCall(bytes context) → bytes32
外部#
在被[IRelayHub](#irelayhub)批准的Relayer 调用请求上调用，在执行Relayer 调用之前。这允许例如预先收取交易发送者的费用。

上下文是通过[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)返回的元组的第二个值。

返回一个值以传递给[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-2)。

[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)使用100k gas进行调用：如果在执行过程中用尽gas或者发生回滚，Relayer 调用将不会被执行，但是收件人仍然需要支付交易的费用。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
外部#
在[IRelayHub](#irelayhub)对已批准的Relayer 调用请求进行调用后，Relayer 调用执行完成。这允许例如收取用户Relayer 调用费用、返回[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)的超额收费或执行与合约特定的簿记。

context是通过[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)返回的元组的第二个值。success是Relayer 调用的执行状态。actualCharge是接收者将为交易支付的估计费用，不包括[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-2)本身使用的任何gas。preRetVal是[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)的返回值。

[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-2)使用100k gas进行调用：如果在执行过程中gas耗尽或发生其他还原，Relayer 调用和对[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)的调用将被还原，但接收者仍然需要支付交易的费用。

### IRelayHub
RelayHub是GSN的核心合约，用户不需要直接与该合约进行交互。

有关如何在本地测试网络上部署RelayHub实例的更多信息，请参阅[OpenZeppelin GSN助手](https://github.com/OpenZeppelin/openzeppelin-gsn-helpers)。

**FUNCTIONS**
[stake(relayaddr, unstakeDelay)](#stakeaddress-relayaddr-uint256-unstakedelay)

[registerRelay(transactionFee, url)](#registerrelayuint256-transactionfee-string-url)

[removeRelayByOwner(relay)](#removerelaybyowneraddress-relay)

[unstake(relay)](#unstakeaddress-relay)

[getRelay(relay)](#getrelayaddress-relay-→-uint256-totalstake-uint256-unstakedelay-uint256-unstaketime-address-payable-owner-enum-irelayhubrelaystate-state)

[depositFor(target)](#depositforaddress-target)

[balanceOf(target)](#balanceofaddress-target-→-uint256)

[withdraw(amount, dest)](#withdrawuint256-amount-address-payable-dest)

[canRelay(relay, from, to, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, signature, approvalData)](#canrelayaddress-relay-address-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata-→-uint256-status-bytes-recipientcontext)

[relayCall(from, to, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, signature, approvalData)](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)

[requiredGas(relayedCallStipend)](#requiredgasuint256-relayedcallstipend-→-uint256)

[maxPossibleCharge(relayedCallStipend, gasPrice, transactionFee)](#maxpossiblechargeuint256-relayedcallstipend-uint256-gasprice-uint256-transactionfee-→-uint256)

[penalizeRepeatedNonce(unsignedTx1, signature1, unsignedTx2, signature2)](#penalizerepeatednoncebytes-unsignedtx1-bytes-signature1-bytes-unsignedtx2-bytes-signature2)

[penalizeIllegalTransaction(unsignedTx, signature)](#penalizeillegaltransactionbytes-unsignedtx-bytes-signature)

[getNonce(from)](#getnonceaddress-from-→-uint256)

**EVENTS**
[Staked(relay, stake, unstakeDelay)](#stakedaddress-relay-uint256-stake-uint256-unstakedelay)

[RelayAdded(relay, owner, transactionFee, stake, unstakeDelay, url)](#relayaddedaddress-relay-address-owner-uint256-transactionfee-uint256-stake-uint256-unstakedelay-string-url)

[RelayRemoved(relay, unstakeTime)](#relayremovedaddress-relay-uint256-unstaketime)

[Unstaked(relay, stake)](#unstakedaddress-relay-uint256-stake)

[Deposited(recipient, from, amount)](#depositedaddress-recipient-address-from-uint256-amount)

[Withdrawn(account, dest, amount)](#withdrawnaddress-account-address-dest-uint256-amount)

[CanRelayFailed(relay, from, to, selector, reason)](#canrelayfailedaddress-relay-address-from-address-to-bytes4-selector-uint256-reason)

[TransactionRelayed(relay, from, to, selector, status, charge)](#transactionrelayedaddress-relay-address-from-address-to-bytes4-selector-enum-irelayhubrelaycallstatus-status-uint256-charge)

[Penalized(relay, sender, amount)](#penalizedaddress-relay-address-sender-uint256-amount)

#### stake(address relayaddr, uint256 unstakeDelay)
external#
向Relayer 添加股份并设置其解锁延迟。如果Relayer 不存在，则创建它，并且调用此函数的调用者成为其所有者。如果Relayer 已经存在，则只有所有者可以调用此函数。Relayer 不能成为自己的所有者。

此函数调用中的所有以太将被添加到Relayer 的股份中。其解锁延迟将分配给unstakeDelay，但新值必须大于或等于当前值。

触发一个[Staked](#stakedaddress-relay-uint256-stake-uint256-unstakedelay事件。

#### registerRelay(uint256 transactionFee, string url)
external#
将调用者注册为Relayer 。Relayer 必须进行抵押，并且不能是合约（即必须直接从EOA调用此函数）。

此函数可以被多次调用，每次调用都会发出新的[RelayAdded](#relayaddedaddress-relay-address-owner-uint256-transactionfee-uint256-stake-uint256-unstakedelay-string-url)事件。注意，[relayCall](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)不强制执行接收到的transactionFee。

发出一个[RelayAdded](#relayaddedaddress-relay-address-owner-uint256-transactionfee-uint256-stake-uint256-unstakedelay-string-url)事件。

#### removeRelayByOwner(address relay)
external#
移除（注销）一个Relayer 。已注销（但已抵押）的Relayer 也可以被移除。

只能由Relayer 的所有者调用。在Relayer 的解除锁定延迟过后，可以调用解锁（[unstake](#unstakeaddress-relay)）函数。

发出一个[RelayRemoved](#relayremovedaddress-relay-uint256-unstaketime)事件。

#### unstake(address relay)
external#

#### getRelay(address relay) → uint256 totalStake, uint256 unstakeDelay, uint256 unstakeTime, address payable owner, enum IRelayHub.RelayState state
external#
返回Relayer 的状态。请注意，Relayer 在解除抵押或受到惩罚时可以被删除，导致此函数返回一个空条目。

#### depositFor(address target)
external#
存入以太到合约中，以便能够接收（和支付）Relayer 的交易。

未使用的余额只能通过合约本身调用[withdraw](#withdrawuint256-amount-address-payable-dest)来提取。

触发一个[Deposited](#depositedaddress-recipient-address-from-uint256-amount)事件。

#### balanceOf(address target) → uint256
external#
返回一个账户的存款。这些存款可以是合约的资金，也可以是Relayer 所有者的收入。

#### withdraw(uint256 amount, address payable dest)
external#

#### canRelay(address relay, address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData) → uint256 status, bytes recipientContext
外部#
检查RelayHub是否接受Relayer 操作。为了实现这一点，必须满足多个条件： - 所有参数必须由发送者（from）签名 - 发送者的nonce必须是当前的nonce - 收件人必须接受此交易（通过[acceptRelayedCall](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)）

返回PreconditionCheck值（如果可以Relayer 交易则返回OK），或者如果在[acceptRelayedCall](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)中返回了收件人特定的错误代码，则返回该错误代码。

#### relayCall(address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData)
外部#
转发一个交易。

为了成功，必须满足多个条件：- [canRelay](#canrelayaddress-relay-address-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata-→-uint256-status-bytes-recipientcontext)必须返回PreconditionCheck.OK- 发送方必须是注册的Relayer - 交易的gas价格必须大于或等于发送方请求的gas价格- 如果所有内部交易（对接收方的调用）使用了它们可用的所有gas，则交易必须有足够的gas，以免gas耗尽- 接收方必须有足够的余额来支付最坏情况下的Relayer 费用（即当所有gas都用尽时）

如果所有条件都满足，将转发调用并向接收方收费。按顺序调用[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)、编码函数和[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-2)。

参数：- from：发起请求的客户端- to：目标[IRelayRecipient](#irelayrecipient)合约- encodedFunction：要转发的函数调用，包括数据- transactionFee：Relayer 接管的实际gas成本的费用（%）- gasPrice：客户端愿意支付的gas价格- gasLimit：调用编码函数时要转发的gas- nonce：客户端的nonce- signature：客户端对所有先前参数的签名，加上Relayer 和RelayHub地址- approvalData：转发到[acceptRelayedCall](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)的特定于dapp的数据。RelayHub不验证此值，但仍然可以用于例如签名。

发出[TransactionRelayed](#transactionrelayedaddress-relay-address-from-address-to-bytes4-selector-enum-irelayhubrelaycallstatus-status-uint256-charge)事件。

#### requiredGas(uint256 relayedCallStipend) → uint256
外部#
返回应向[relayCall](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)的调用转发多少gas，以便Relayer 事务可以花费多达relayedCallStipend gas。

#### maxPossibleCharge(uint256 relayedCallStipend, uint256 gasPrice, uint256 transactionFee) → uint256
外部#
根据转发的燃料量、燃料价格和Relayer 费用，返回最大的接收者费用。

#### penalizeRepeatedNonce(bytes unsignedTx1, bytes signature1, bytes unsignedTx2, bytes signature2)
外部#
对使用相同nonce签署了两个交易的Relayer 进行惩罚（只有第一个交易有效），并且数据不同（例如，燃料价格、燃料限制等可能不同）。

必须提供两个交易的（未签名的）交易数据和签名。

#### penalizeIllegalTransaction(bytes unsignedTx, bytes signature)
外部#
对于发送了未针对RelayHub的[registerRelay](#registerrelayuint256-transactionfee-string-url)或[relayCall](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)的交易的Relayer 进行惩罚。

#### getNonce(address from) → uint256
外部#
在RelayHub中返回账户的nonce。

#### Staked(address relay, uint256 stake, uint256 unstakeDelay)
event# 
当 Relayer 的质押或取消质押延迟增加时发出

#### RelayAdded(address relay, address owner, uint256 transactionFee, uint256 stake, uint256 unstakeDelay, string url)
event# 
当一个Relayer 被注册或重新注册时发出。查看这些事件（并过滤掉[RelayRemoved](#relayremovedaddress-relay-uint256-unstaketime)事件）可以让客户端发现可用的Relayer 列表。

#### RelayRemoved(address relay, uint256 unstakeTime)
event# 
当继电器被移除（取消注册）时发出。unstakeTime是解除质押可调用的时间。

#### Unstaked(address relay, uint256 stake)
event# 
当一个Relay取消质押时，包括返回的质押金额时发出。

#### Deposited(address recipient, address from, uint256 amount)
event#

当调用[depositFor](#depositforaddress-target)时发出，包括所存入的金额和账户。

#### Withdrawn(address account, address dest, uint256 amount)
event#
当一个账户从RelayHub中提取资金时发出。

#### CanRelayFailed(address relay, address from, address to, bytes4 selector, uint256 reason)
event#
当尝试Relayer 呼叫失败时发出。

这可能是由于错误的[relayCall](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)参数或接收方不接受Relayer 呼叫。实际的Relayer 呼叫未执行，并且接收方未收费。

reason参数包含一个错误代码：值1-10对应于PreconditionCheck条目，而大于10的值是从[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)返回的自定义接收方错误代码。

#### TransactionRelayed(address relay, address from, address to, bytes4 selector, enum IRelayHub.RelayCallStatus status, uint256 charge)
event#
当一个交易被Relayer 时发出。在监控Relayer 的操作和Relayer 调用合约时非常有用。

请注意，实际的编码函数可能会被还原：这在状态参数中表示。

charge 是从接收者的余额中扣除的以太价值，支付给Relayer 的所有者。

#### Penalized(address relay, address sender, uint256 amount)
event#
当继电器受到惩罚时发出的信号。