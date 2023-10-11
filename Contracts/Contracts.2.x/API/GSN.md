# Gas Station Network (GSN)
从v2.4.0版本开始可用。

这组合约提供了通过[Gas Station Network（GSN）](https://gsn.openzeppelin.com/)调用合约所需的所有工具。

> TIP
如果你对GSN还不熟悉，请查看我们关于[该系统的概述](/Learn/Sending%20gasless%20transactions/Sending-gasless-transactions.md)以及[创建支持GSN的合约的基本指南](../Gas-Station-Network/Gas-Station-Network.md)。

接收方必须继承的核心合约是[GSNRecipient](#gsnrecipient)：它包括所有必要的接口，以及一些帮助方法，使与GSN的交互更容易。

使编写[GSN策略](../Gas-Station-Network/Strategies.md#gsn-strategies)变得简单的实用程序可以在[GSNRecipient](#gsnrecipient)中找到，或者你可以直接使用我们预先制作的策略之一：

* [GSNRecipientERC20Fee](#gsnrecipienterc20fee)使用特定应用的[ERC20代币](../Tokens/ERC20/ERC20.md)向最终用户收取gas费用

* [GSNRecipientSignature](#gsnrecipientsignature)接受由受信任的第三方（例如后端中的私钥）签名的所有Relayer 调用

你还可以查看组成GSN协议的两个合约接口：[IRelayRecipient](#irelayrecipient)和[IRelayHub](#irelayhub)，但你不需要直接使用它们。

## 接收方

### GSNRecipient
基本GSN接收方合约：包括[IRelayRecipient](#irelayrecipient)接口，并在继承树中的所有合约上启用GSN支持。

> TIP
此合约是抽象的。[IRelayRecipient.acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)、[_preRelayedCall](#_prerelayedcallbytes-→-bytes32)和[_postRelayedCall](#_postrelayedcallbytes-bool-uint256-bytes32)函数未实现，必须由派生合约提供。有关如何使用预构建的[GSNRecipientSignature](#gsnrecipientsignature)和[GSNRecipientERC20Fee](#gsnrecipienterc20fee)的更多信息，请参阅[GSN策略](../Gas-Station-Network/Strategies.md#gsn-strategies)，或者如何编写你自己的策略。

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

CONTEXT
constructor()

IRELAYRECIPIENT
[acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)

**EVENTS**
[RelayHubChanged(oldRelayHub, newRelayHub)](#relayhubchangedaddress-oldrelayhub-address-newrelayhub)

#### getHubAddr() → address
public#
返回此接收者的[IRelayHub](#irelayhub)合约的地址。

#### _upgradeRelayHub(address newRelayHub)
internal#
切换到新的[IRelayHub](#irelayhub)实例。添加此方法是为了未来的兼容性：没有理由不使用默认实例。

> IMPORTANT
升级后，[GSNRecipient](#gsnrecipient)将无法从旧的[IRelayHub](#irelayhub)实例接收Relayer 调用。此外，所有资金应通过[_withdrawDeposits](#_withdrawdepositsuint256-amount-address-payable-payee)事先提取。

#### relayHubVersion() → string
public#
返回此接收方实现构建的[IRelayHub](#irelayhub)的版本字符串。如果使用[_upgradeRelayHub](#_upgraderelayhubaddress-newrelayhub)，则新的[IRelayHub](#irelayhub)实例应与此版本兼容。

#### _withdrawDeposits(uint256 amount, address payable payee)
internal#
从RelayHub中提取接收者的存款。

派生合约应该在外部接口中以适当的访问控制方式公开此功能。

#### _msgSender() → address payable
internal#
msg.sender的替代品。返回交易的实际发送者：对于常规交易是msg.sender，对于GSNRelayer 调用是最终用户（其中msg.sender实际上是RelayHub）。

> IMPORTANT
从[GSNRecipient](#gsnrecipient)派生的合约不应使用[_msg.sender](#_msgsender-→-address-payable)，而应使用_msgSender。

#### _msgData() → bytes
internal#
替代msg.data。返回交易的实际calldata：对于常规交易，返回msg.data，对于GSNRelayer 调用，返回一个简化版本（其中msg.data包含额外的信息）。

> IMPORTANT
从[GSNRecipient](#gsnrecipient)派生的合约不应使用msg.data，而应使用[_msg.sender](#_msgsender-→-address-payable)。

#### preRelayedCall(bytes context) → bytes32
外部#
请参考IRelayRecipient.preRelayedCall函数。

不应直接重写此函数，而应使用_preRelayedCall。

* 要求：

    * 调用者必须是RelayHub合约。

#### _preRelayedCall(bytes context) → bytes32
外部#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer 调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
外部#
请参阅IRelayRecipient.postRelayedCall。

该函数不应直接被重写，而应使用_postRelayedCall。

* 要求：

    * 调用者必须是RelayHub合约。

#### _postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
internal#
请参阅IRelayRecipient.postRelayedCall。

由GSNRecipient.postRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的Relayer 调用后处理。

#### _approveRelayedCall() → uint256, bytes
internal#
在acceptRelayedCall中返回此内容以继续执行Relayer 调用。请注意，RelayHub将向此合约收取费用。

#### _approveRelayedCall(bytes context) → uint256, bytes
internal#
请查看 GSNRecipient._approveRelayedCall。

此重载将上下文转发给 _preRelayedCall 和 _postRelayedCall。

#### _rejectRelayedCall(uint256 errorCode) → uint256, bytes
internal#
在acceptRelayedCall中返回这个值以阻止执行Relayer 调用。不会收取任何费用。

#### _computeCharge(uint256 gas, uint256 gasPrice, uint256 serviceFee) → uint256
internal#

#### RelayHubChanged(address oldRelayHub, address newRelayHub)
event#
当合约将其[IRelayHub](#irelayhub)合约更改为新合约时发出。

## Strategies

### GSNRecipientSignature
一种[GSN策略](../Gas-Station-Network/Strategies.md#gsn-strategies)，允许通过可信签名者的签名进行Relayer 交易。意图是由一个在链下执行验证的服务器生成此签名。需要注意的是，在此方案中不向用户收取任何费用。因此，服务器应确保在其经济和威胁模型中考虑到这一点。

**FUNCTIONS**
[constructor(trustedSigner)](#constructoraddress-trustedsigner)

[acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, _)](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-maxpossiblecharge-→-uint256-bytes)

[{xref-GSNRecipientSignature-preRelayedCall}[_preRelayedCall()]]

[{xref-GSNRecipientSignature-postRelayedCall}[_postRelayedCall(, _, _, _)]]

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
GSNRECIPIENT
[RelayHubChanged(oldRelayHub, newRelayHub)](#relayhubchangedaddress-oldrelayhub-address-newrelayhub)

#### constructor(address trustedSigner)
public#
设置可信签名者，该签名者将负责批准Relayer 调用的签名。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256) → uint256, bytes
internal#
确保只有带有可信签名的交易才能通过GSNRelayer 。

#### _preRelayedCall(bytes) → bytes32
internal#

#### _postRelayedCall(bytes, bool, uint256, bytes32)
internal#

### GSNRecipientERC20Fee
一种[GSN策略](../Gas-Station-Network/Strategies.md#gsn-strategies)是使用一种特殊的ERC20代币作为交易费，我们称之为gas支付代币。收取的费用金额恰好等于收款方所收取的以太金额。这意味着该代币基本上与以太的价值挂钩。

gas支付代币的分发策略并未在该合约中定义。它是一种可铸造的代币，其唯一的铸币者是收款方，因此策略必须在派生合约中实现，并利用内部的[_mint](#_mintaddress-account-uint256-amountl)函数。

**FUNCTIONS**
[constructor(name, symbol)](#constructorstring-name-string-symbol)

[token()](#token-→-contract-ierc20)

[_mint(account, amount)](#_mintaddress-account-uint256-amount)

[acceptRelayedCall(_, from, _, transactionFee, gasPrice, _, _, _, maxPossibleCharge)](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)

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

[_preRelayedCall(context)](#_prerelayedcallbytes-context-→-bytes32)

[postRelayedCall(context, success, actualCharge, preRetVal)](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval)

[_postRelayedCall(context, success, actualCharge, preRetVal)](#_postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval)

[_approveRelayedCall()](#_approverelayedcall-→-uint256-bytes)

[_approveRelayedCall(context)](#_approverelayedcallbytes-context-→-uint256-bytes)

[_rejectRelayedCall(errorCode)](#_rejectrelayedcalluint256-errorcode-→-uint256-bytes)

[_computeCharge(gas, gasPrice, serviceFee)](#_computechargeuint256-gas-uint256-gasprice-uint256-servicefee-→-uint256)

**EVENTS**
GSNRECIPIENT
[RelayHubChanged(oldRelayHub, newRelayHub)](#relayhubchangedaddress-oldrelayhub-address-newrelayhub)

#### constructor(string name, string symbol)
public#
构造函数的参数是燃料支付代币将具有的细节：名称和符号。小数位数被硬编码为18。

#### token() → contract IERC20
public#
返回gas支付代币。

#### _mint(address account, uint256 amount)
internal#
内部函数，用于铸造gas支付代币。派生合约应该在其公共API中公开此函数，并配备适当的访问控制机制。

#### acceptRelayedCall(address, address from, bytes, uint256 transactionFee, uint256 gasPrice, uint256, uint256, bytes, uint256 maxPossibleCharge) → uint256, bytes
外部#
确保只有拥有足够的gas支付代币余额的用户才能通过GSNRelayer 交易。

#### _preRelayedCall(bytes context) → bytes32
internal#
对用户进行预充值。将从用户的燃料支付代币余额中扣除最大可能的费用（取决于燃料限制、燃料价格和费用）。请注意，这是对实际费用的过估计，这是必要的，因为我们无法预测执行实际上需要多少燃料。剩余费用将在[_postRelayedCall](#_postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval)中返还给用户。

#### _postRelayedCall(bytes context, bool, uint256 actualCharge, bytes32)
internal#
一旦实际执行成本确定，将向用户退还之前多收取的金额。

## Protocol

### IRelayRecipient
一个通过GSN从[IRelayHub](#irelayhub)调用的合约的基本接口。

> TIP
你不需要自己编写实现！请继承自[GSNRecipient](#gsnrecipient)。

**FUNCTIONS**
[getHubAddr()](#gethubaddr-e28692-address-1)

[acceptRelayedCall(relay, from, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, approvalData, maxPossibleCharge)](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)

[preRelayedCall(context)](#prerelayedcallbytes-context-e28692-bytes32-1)

[postRelayedCall(context, success, actualCharge, preRetVal)](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-1)

#### getHubAddr() → address
internal#
返回此收件人与之交互的[IRelayHub](#irelayhub)实例的地址。

#### acceptRelayedCall(address relay, address from, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes approvalData, uint256 maxPossibleCharge) → uint256, bytes
internal#
被[IRelayHub](#irelayhub)调用以验证接收方是否同意为Relayer 调用付费。请注意，无论Relayer 调用的执行结果如何（即是否还原），接收方都将被收费。

Relayer 请求由from发起，并将由relay提供服务。encodedFunction是Relayer 调用的calldata，因此它的前四个字节是函数选择器。Relayer 调用将转发gasLimit gas，并以至少gasPrice的燃料价格执行事务。relay的费用为transactionFee，并且接收方的最大可能费用为maxPossibleCharge（以wei为单位）。nonce是发送方（from）在[IRelayHub](#irelayhub)中用于重放攻击保护的nonce，approvalData是一个可选参数，可用于保存对所有或部分先前值的签名。

返回一个元组，其中第一个值用于指示批准（0）或拒绝（自定义非零错误代码，保留值为1到10），第二个值是要传递给其他[IRelayRecipient](#irelayrecipient)函数的数据。

[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)使用50k gas进行调用：如果在执行过程中用尽，请求将被视为被拒绝。正常的还原也会触发拒绝。

#### preRelayedCall(bytes context) → bytes32
internal#
在[IRelayHub](#irelayhub)上调用已批准的Relayer 调用请求之前，调用preRelayedCall。在执行Relayer 调用之前，这允许例如预先向交易发送方收费。

context是[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)返回的元组的第二个值。

返回一个值以传递给[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-1)。

[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)使用100k gas进行调用：如果在执行过程中用完gas或者发生回滚，Relayer 调用将不会被执行，但是接收方仍将被收取交易费用。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
internal#
在[IRelayHub](#irelayhub)上的已批准的Relayer 调用请求中调用，在Relayer 调用执行后。这允许例如为Relayer 调用的成本向用户收费，从[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)返回任何超额费用，或执行特定于合约的簿记。

context是[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)返回的元组的第二个值。success是Relayer 调用的执行状态。actualCharge是接收方将为交易收取的估计费用，不包括[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-1)本身使用的任何gas。preRetVal是[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)的返回值。

[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-1)使用100k gas进行调用：如果在执行过程中用完gas或以其他方式回滚，则Relayer 调用和对[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)的调用将被回滚，但接收方仍将被收取交易费用。

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

[relayCall(from, to, encodedFunction, transactionFee, gasPrice, gasLimit, nonce, signature, approvalData)](#relayaddedaddress-relay-address-owner-uint256-transactionfee-uint256-stake-uint256-unstakedelay-string-url)

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
外部#
向Relayer 添加股份并设置其解锁延迟。如果Relayer 不存在，则创建该Relayer ，并且调用此函数的调用者成为其所有者。如果Relayer 已经存在，则只有所有者可以调用此函数。Relayer 不能成为自己的所有者。

此函数调用中的所有以太将添加到Relayer 的股份中。其解锁延迟将分配给unstakeDelay，但新值必须大于或等于当前值。

发出[Staked](#stakedaddress-relay-uint256-stake-uint256-unstakedelay)事件。

#### registerRelay(uint256 transactionFee, string url)
外部#
将调用者注册为Relayer 。Relayer 必须进行抵押，并且不能是合约（即此函数必须直接从EOA调用）。

此函数可以多次调用，发出新的[RelayAdded](#relayaddedaddress-relay-address-owner-uint256-transactionfee-uint256-stake-uint256-unstakedelay-string-url)事件。请注意，[relayCall](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)不强制执行接收的transactionFee。

发出[RelayAdded](#relayaddedaddress-relay-address-owner-uint256-transactionfee-uint256-stake-uint256-unstakedelay-string-url)事件。

#### removeRelayByOwner(address relay)
外部#
移除（注销）中继。也可以移除未注册（但已质押）的中继。

只有中继的所有者才能调用此功能。在中继的unstakeDelay过后，[unstake](#unstakeaddress-relay)将可以被调用。

将触发一个[RelayRemoved](#relayremovedaddress-relay-uint256-unstaketime)事件。

#### unstake(address relay)
外部#

#### getRelay(address relay) → uint256 totalStake, uint256 unstakeDelay, uint256 unstakeTime, address payable owner, enum IRelayHub.RelayState state
外部#
返回Relayer 的状态。请注意，Relayer 可能会在解除质押或受到惩罚时被删除，导致此函数返回一个空条目。

#### depositFor(address target)
外部#
为合约存入以太，以便它可以接收（并支付）中继交易。

未使用的余额只能由合约本身通过调用 [withdraw](#withdrawuint256-amount-address-payable-dest)来提取。

发出[Deposited](#depositedaddress-recipient-address-from-uint256-amount)事件。

#### balanceOf(address target) → uint256
外部#
返回一个账户的存款。这些存款可以是合约的资金，也可以是Relayer 所有者的收入。

#### withdraw(uint256 amount, address payable dest)
外部#

#### canRelay(address relay, address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData) → uint256 status, bytes recipientContext
外部#
检查RelayHub是否接受Relayer 操作。为了使此操作成功，必须满足多个条件：- 所有参数必须由发送方（from）签名- 发送方的nonce必须是当前的- 接收方必须接受此交易（通过[acceptRelayedCall](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)）

返回一个PreconditionCheck值（当交易可以被Relayer 时为OK），或者如果[acceptRelayedCall](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)返回一个接收方特定的错误代码，则返回该代码。

#### relayCall(address from, address to, bytes encodedFunction, uint256 transactionFee, uint256 gasPrice, uint256 gasLimit, uint256 nonce, bytes signature, bytes approvalData)
外部#
转发交易。

为了成功转发交易，必须满足多个条件： - [canRelay](#canrelayaddress-relay-address-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata-→-uint256-status-bytes-recipientcontext)必须返回PreconditionCheck.OK - 发送者必须是注册的Relayer  - 交易的gas价格必须大于或等于发送者请求的gas价格 - 如果所有内部交易（对接收者的调用）使用了所有可用的gas，交易必须有足够的gas以防止gas耗尽 - 接收者必须有足够的余额来支付最坏情况下的Relayer 费用（即当所有gas都用完时）

如果满足所有条件，将转发调用并向接收者收费。按照以下顺序调用[preRelayedCall](#prerelayedcallbytes-context-e28692-bytes32-1)、编码函数和[postRelayedCall](#postrelayedcallbytes-context-bool-success-uint256-actualcharge-bytes32-preretval-1)。

参数： - from：发起请求的客户端 - to：目标[IRelayRecipient](#irelayrecipient)合约 - encodedFunction：要转发的函数调用，包括数据 - transactionFee：Relayer 接管的实际gas成本的费用（%） - gasPrice：客户端愿意支付的gas价格 - gasLimit：在调用编码函数时要转发的gas - nonce：客户端的nonce - signature：客户端对所有先前参数的签名，加上Relayer 和RelayHub地址 - approvalData：转发给[acceptRelayedCall](#acceptrelayedcalladdress-relay-address-from-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-approvaldata-uint256-→-uint256-bytes)的特定于dapp的数据。RelayHub不验证此值，但仍然可以用于例如签名。

发出[TransactionRelayed](#transactionrelayedaddress-relay-address-from-address-to-bytes4-selector-enum-irelayhubrelaycallstatus-status-uint256-charge)事件。

#### requiredGas(uint256 relayedCallStipend) → uint256
外部#
在调用[relayCall](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)时，返回应该转发多少gas，以便Relayer 交易最多花费relayedCallStipend个gas。

#### maxPossibleCharge(uint256 relayedCallStipend, uint256 gasPrice, uint256 transactionFee) → uint256
外部#
给定转发的气体数量、气体价格和Relayer 费用，返回最大的接收方费用。

#### penalizeRepeatedNonce(bytes unsignedTx1, bytes signature1, bytes unsignedTx2, bytes signature2)
外部#
惩罚一个Relayer 节点，该节点使用相同的nonce签署了两笔交易（只有第一笔交易有效），但数据不同（gas价格、gas限制等可能不同）。

必须提供两笔交易的（未签名的）交易数据和签名。

#### penalizeIllegalTransaction(bytes unsignedTx, bytes signature)
外部#
对于发送了不针对`RelayHub`的[`registerRelay`](#registerrelayuint256-transactionfee-string-url)或[`relayCall`](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)的事务的Relayer 进行惩罚。

#### getNonce(address from) → uint256
外部#
在RelayHub中返回一个账户的nonce。

#### Staked(address relay, uint256 stake, uint256 unstakeDelay)
event#
当Relayer 的质押或解质押延迟增加时发出

#### RelayAdded(address relay, address owner, uint256 transactionFee, uint256 stake, uint256 unstakeDelay, string url)
event#
当一个继电器被注册或重新注册时发出。通过查看这些事件（并过滤掉[RelayRemoved](#relayremovedaddress-relay-uint256-unstaketime)事件），客户端可以发现可用继电器的列表。

#### RelayRemoved(address relay, uint256 unstakeTime)
event#
当 Relayer 被移除（取消注册）时发出。unstakeTime是可以调用解除质押的时间。

#### Unstaked(address relay, uint256 stake)
event#
当一个Relayer 被解除质押时，包括返回的质押金额时发出。

#### Deposited(address recipient, address from, uint256 amount)
event#
在调用[depositFor](#depositforaddress-target)时发出，包括被资助的金额和账户。

#### Withdrawn(address account, address dest, uint256 amount)
event#
当一个账户从RelayHub中提取资金时发出。

#### CanRelayFailed(address relay, address from, address to, bytes4 selector, uint256 reason)
event#
当Relayer 调用失败时发出。

这可能是由于错误的[relayCall](#relaycalladdress-from-address-to-bytes-encodedfunction-uint256-transactionfee-uint256-gasprice-uint256-gaslimit-uint256-nonce-bytes-signature-bytes-approvaldata)参数或接收方不接受Relayer 调用引起的。实际的Relayer 调用未执行，并且接收方未收费。

reason参数包含一个错误代码：值1-10对应于PreconditionCheck条目，而大于10的值是从[acceptRelayedCall](#acceptrelayedcalladdress-address-from-bytes-uint256-transactionfee-uint256-gasprice-uint256-uint256-bytes-uint256-maxpossiblecharge-→-uint256-bytes)返回的自定义接收方错误代码。

#### TransactionRelayed(address relay, address from, address to, bytes4 selector, enum IRelayHub.RelayCallStatus status, uint256 charge)
event#
当一个交易被Relayer 时发出。在监控Relayer 操作和Relayer 调用合约时非常有用。

请注意，实际编码的函数可能会被还原：这在状态参数中表示。

charge是从接收者的余额中扣除的以太值，支付给Relayer 的所有者。

#### Penalized(address relay, address sender, uint256 amount)
event#
当继电器受到惩罚时发出的信号。