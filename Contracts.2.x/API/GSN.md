# Gas Station Network (GSN)
从v2.4.0版本开始可用。

这组合同提供了通过[Gas Station Network（GSN）](https://gsn.openzeppelin.com/)调用合同所需的所有工具。

> TIP
如果您对GSN还不熟悉，请查看我们关于*该系统的概述*以及*创建支持GSN的合同的基本指南*。

接收方必须继承的核心合同是*GSNRecipient*：它包括所有必要的接口，以及一些帮助方法，使与GSN的交互更容易。

使编写*GSN策略*变得简单的实用程序可以在*GSNRecipient*中找到，或者您可以直接使用我们预先制作的策略之一：

* *GSNRecipientERC20Fee*使用特定应用的*ERC20代币*向最终用户收取燃气费用

* *GSNRecipientSignature*接受由受信任的第三方（例如后端中的私钥）签名的所有中继调用

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
升级后，*GSNRecipient*将无法从旧的*IRelayHub*实例接收中继调用。此外，所有资金应通过*_withdrawDeposits*事先提取。

#### relayHubVersion() → string
公开#
返回此接收方实现构建的*IRelayHub*的版本字符串。如果使用*_upgradeRelayHub*，则新的*IRelayHub*实例应与此版本兼容。

#### _withdrawDeposits(uint256 amount, address payable payee)
内部#
从RelayHub中提取接收者的存款。

派生合约应该在外部接口中以适当的访问控制方式公开此功能。

#### _msgSender() → address payable
内部#
msg.sender的替代品。返回交易的实际发送者：对于常规交易是msg.sender，对于GSN中继调用是最终用户（其中msg.sender实际上是RelayHub）。

> IMPORTANT
从*GSNRecipient*派生的合约不应使用*msg.sender*，而应使用_msgSender。

#### _msgData() → bytes
内部#
替代msg.data。返回交易的实际calldata：对于常规交易，返回msg.data，对于GSN中继调用，返回一个简化版本（其中msg.data包含额外的信息）。

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

由GSNRecipient.preRelayedCall调用，该函数断言调用者是RelayHub合约。派生合约必须实现此函数，并进行任何它们希望进行的中继调用预处理。