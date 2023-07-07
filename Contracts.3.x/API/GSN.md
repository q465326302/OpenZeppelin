# Gas Station Network (GSN)
这套合同提供了通过[Gas Station Network（GSN）](https://gsn.openzeppelin.com/)调用合同所需的所有工具。

> TIP
如果您对GSN还不熟悉，请查看我们关于*该系统的概述*和*创建GSN兼容合同的基本指南*。

接收方必须继承的核心合同是*GSNRecipient*：它包括所有必要的接口，以及一些帮助方法，使与GSN的交互更容易。

使编写*GSN策略*变得简单的实用程序可在*GSNRecipient*中使用，或者您可以直接使用我们预先制作的策略之一：

* *GSNRecipientERC20Fee* 使用特定应用程序的*ERC20代币*为最终用户收取燃气费用

* *GSNRecipientSignature* 接受已由可信第三方（例如后端中的私钥）签名的所有中继调用

您还可以查看构成GSN协议的两个合同接口：*IRelayRecipient*和*IRelayHub*，但您不需要直接使用它们。

## 接收方

### GSNRecipient

基本GSN接收方合同：包括*IRelayRecipient*接口，并在继承树中的所有合同上启用GSN支持。

> TIP
此合同是抽象的。未实现*IRelayRecipient.acceptRelayedCall*、*_preRelayedCall*和*_postRelayedCall*函数，必须由派生合同提供。有关如何使用预构建的*GSNRecipientSignature*和*GSNRecipientERC20Fee*的更多信息，请参阅*GSN策略*，或者如何编写您自己的策略。

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
升级后，*GSNRecipient*将无法从旧的*IRelayHub*实例接收中继调用。此外，所有资金应通过*_withdrawDeposits*提前提取。

#### relayHubVersion() → string
公开#
返回此接收方实现版本的*IRelayHub*版本字符串。如果使用*_upgradeRelayHub*，则新的*IRelayHub*实例应与此版本兼容。

#### _withdrawDeposits(uint256 amount, address payable payee)
内部#
从RelayHub中撤回收款人的存款。

衍生合约应该以外部接口的形式公开这一操作，并进行适当的访问控制。

#### _msgSender() → address payable
内部#
替代msg.sender的方法。返回交易的实际发送者：对于普通交易，返回msg.sender，对于GSN中继调用，返回最终用户（其中msg.sender实际上是RelayHub）。

> IMPORTANT
从*GSNRecipient*派生的合约不应该使用msg.sender，而应该使用*_msgSender*。

#### _msgData() → bytes
替代msg.data。返回交易的实际calldata：对于常规交易，返回msg.data，对于GSN中继调用（其中msg.data包含其他信息），返回简化版本。

> 从*GSNRecipient*派生的合约不应使用msg.data，而应使用*_msgData*。

#### preRelayedCall(bytes context) → bytes32
公开#
请查看IRelayRecipient.preRelayedCall。

不应直接覆盖此函数，请使用_preRelayedCall代替。

* 要求：
    * 调用者必须是RelayHub合约。

#### _preRelayedCall(bytes context) → bytes32
内部#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，用于断言调用者是RelayHub合约。派生合约必须实现此函数以执行任何他们希望进行的中继调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
公开#
请参阅IRelayRecipient.preRelayedCall。

由GSNRecipient.preRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的中继调用预处理。

#### postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
公开#
请参阅IRelayRecipient.postRelayedCall。

此函数不应直接被覆盖，而应使用_postRelayedCall。

* 要求：
    * 调用者必须是RelayHub合约。

#### _postRelayedCall(bytes context, bool success, uint256 actualCharge, bytes32 preRetVal)
内部#
请参阅IRelayRecipient.postRelayedCall。

通过GSNRecipient.postRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的中继调用后处理。

#### _approveRelayedCall() → uint256, bytes
内部#
返回此值以继续执行中继调用。请注意，RelayHub将向此合约收取费用。

#### _approveRelayedCall(bytes context) → uint256, bytes
内部#
请参阅 GSNRecipient._approveRelayedCall。

这个重载将上下文传递给 _preRelayedCall 和 _postRelayedCall。

#### _rejectRelayedCall(uint256 errorCode) → uint256, bytes
内部#
在acceptRelayedCall中返回此内容以阻止执行中继调用。不会收取任何费用。

#### _computeCharge(uint256 gas, uint256 gasPrice, uint256 serviceFee) → uint256
内部#

#### RelayHubChanged(address oldRelayHub, address newRelayHub)
事件#
当合约将其 *IRelayHub* 合约更改为新的合约时发出。

## Strategies
###