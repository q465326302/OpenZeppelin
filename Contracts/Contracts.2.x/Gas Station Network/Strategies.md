# GSN Strategies
这个指南向您展示了使用OpenZeppelin Contracts接受通过加油站网络（GSN）中继呼叫的不同策略。

首先，我们将介绍什么是“GSN策略”，然后展示如何使用两种最常见的策略。最后，我们将介绍如何创建自定义策略。

如果您仍在学习Gas Station Network的基础知识，您应该先阅读*GSN指南*。

## 解释GSN策略
**GSN策略**决定哪些中继呼叫被批准，哪些被拒绝。策略是GSN中的关键概念。Dapp需要一种策略来防止恶意用户使用Dapp的资金进行中继呼叫费用。

正如我们在GSN指南中所看到的，为了启用*GSN*，您的合约需要扩展自*GSNRecipient*。

GSN接收方合约需要以下内容才能工作：

1. 它需要在其RelayHub上存入资金。

2. 它需要以不同的方式处理msg.sender和msg.data。

3. 它需要决定如何批准和拒绝中继呼叫。

可以通过[GSN Dapp](https://gsn.openzeppelin.com/recipients)工具或使用*OpenZeppelin GSN Helpers*以编程方式存入GSN接收方合约的资金。

实际用户的msg.sender和msg.data可以通过*GSNRecipient*的*_msgSender()*和*_msgData()*安全地获取。

决定如何批准和拒绝中继呼叫要复杂一些。您很可能希望选择哪些用户可以通过GSN使用您的合约，并可能向他们收费，就像夜总会的门卫一样。我们称这些为GSN策略。

基本的*GSNRecipient*合约不包括策略，因此您必须使用其中一个预构建的策略或编写自己的策略。我们将首先介绍使用包含的策略：*GSNRecipientSignature*和*GSNRecipientERC20Fee*。

### GSNRecipientSignature
*GSNRecipientSignature*允许用户通过GSN中继呼叫到您的接收方合约（向您收费），如果他们能够证明您信任的帐户已经批准他们这样做。他们通过签名的方式来实现这一点。

中继呼叫必须包含由相同帐户对中继呼叫参数进行签名。如果不是相同的帐户，则GSNRecipientSignature不会接受中继呼叫。

这意味着您需要建立一个系统，您信任的帐户对中继呼叫参数进行签名，然后将其包含在中继呼叫中，只要它们是有效用户（根据您的业务逻辑）。

有效用户的定义取决于您的系统，但一个示例是已通过某种[OAuth](https://en.wikipedia.org/wiki/OAuth)和验证完成注册的用户，例如通过验证码或验证其电子邮件地址。您还可以进一步限制，并允许新用户发送特定数量的中继呼叫（例如，限制为通过GSN进行5次中继呼叫，此后用户需要创建钱包）。或者，您可以通过线下方式（例如通过信用卡）向用户收费，为其系统上的信用创建中继呼叫，直到信用用尽。

这种设置的好处是，如果您想要更改业务规则，您的合约不需要更改。您只需更改用户免费使用您的合约的后端逻辑条件。另一方面，您需要一个后端服务器、微服务或Lambda函数来完成此操作。

### GSNRecipientSignature如何工作？
GSNRecipientSignature根据包含的签名决定是否接受中继呼叫。

acceptRelayedCall实现从approvalData中的中继呼叫参数的签名中恢复地址，并与信任的签名者进行比较。如果包含的签名与信任的签名者匹配，则批准中继呼叫。另一方面，当包含的签名与信任的签名者不匹配时，中继呼叫会被拒绝，并返回INVALID_SIGNER错误代码。

### 如何使用GSNRecipientSignature
您需要创建一个离线服务（例如后端服务器、微服务或Lambda函数），您的Dapp调用该服务，以使用您信任的签名者帐户对中继呼叫参数进行签名（或不签名，根据您的业务逻辑）。然后，在中继呼叫的approvalData中包含该签名。

您的GSN接收方合约将继承自GSNRecipientSignature，而不是直接使用GSNRecipient，同时在GSNRecipientSignature的构造函数中设置信任的签名者，如下面的示例代码所示：

```
import "@openzeppelin/contracts/GSN/GSNRecipientSignature";

contract MyContract is GSNRecipientSignature {
    constructor(address trustedSigner) public GSNRecipientSignature(trustedSigner) {
    }
}
```

> TIP
我们编写了一份详细指南，介绍如何设置与GSNRecipientSignature配合使用的签名服务器，[快来看看吧](https://forum.openzeppelin.com/t/advanced-gsn-gsnrecipientsignature-sol/1414)！

### GSNRecipientERC20Fee
*GSNRecipientERC20Fee*要复杂一些（但不要担心，我们已经为您编写好了！）。与GSNRecipientSignature不同，GSNRecipientERC20Fee不需要任何离线服务。您将为用户提供专用的ERC20代币，这些代币用于支付转发到您的接收合约的调用。只要用户拥有足够的代币支付，他们的转发调用就会自动获得批准，并且接收合约将支付他们的交易费用！

每个接收合约都有自己的专用代币。代币与以太币的兑换比率为1:1，因为代币用于支付使用GSN时的合约的燃气费用。

GSNRecipientERC20Fee有一个内部的*_mint*函数。首先，您需要设置一种调用它的方式（例如，添加一个具有某种形式的*访问控制*（例如*onlyMinter*）的公共函数）。然后，根据您的业务逻辑向用户发放代币。例如，您可以向新用户铸造一定数量的代币，在用户离线购买代币时铸造代币，根据用户的订阅情况赠送代币等。

> NOTE
用户**不需要为他们的代币调用approve方法**，以便您的接收合约使用它们。它们是修改过的ERC20变体，允许接收合约检索它们。

### GSNRecipientERC20Fee如何工作？
GSNRecipientERC20Fee根据用户代币余额决定是否批准或拒绝转发调用。

acceptRelayedCall函数的实现检查用户的代币余额。如果用户的代币不足，转发调用将被拒绝，并返回INSUFFICIENT_BALANCE错误。如果代币足够，转发调用将获得批准，并返回最终用户地址、maxPossibleCharge、transactionFee和gasPrice数据，以便在_preRelayedCall和_postRelayedCall中使用。

在_preRelayedCall函数中，将maxPossibleCharge数量的代币转移到接收合约中。假设转发调用将使用所有可用的燃气，转移所需的最大代币数量。然后，在_postRelayedCall函数中，计算实际金额，包括接收合约实现和ERC20代币转移，并退还差额。

在_preRelayedCall中转移所需的最大代币数量是为了保护合约免受攻击（这与以太坊交易中锁定以太币的方式非常相似）。

> NOTE
燃气费用估算并不是100%准确，我们可能会在日后进行进一步调整。

> NOTE
始终使用_preRelayedCall和_postRelayedCall函数。内部使用_preRelayedCall和_postRelayedCall函数，而不是公共的preRelayedCall和postRelayedCall函数，因为非RelayHub合约不能调用公共函数。

### 如何使用GSNRecipientERC20Fee
您的GSN接收合约需要继承自GSNRecipientERC20Fee，并具有适当的*访问控制*（用于代币铸造），在GSNRecipientERC20Fee的构造函数中设置代币详细信息，并创建一个公共的铸币函数，以适当的访问控制保护，如以下示例代码所示（使用*MinterRole*）：
```
import "@openzeppelin/contracts/GSN/GSNRecipientERC20Fee";

contract MyContract is GSNRecipientERC20Fee, MinterRole {
    constructor() public GSNRecipientERC20Fee("FeeToken", "FEE") {
    }

    function mint(address account, uint256 amount) public onlyMinter {
        _mint(account, amount);
    }
}
```

## 自定义策略
如果包含的策略不完全符合您的业务需求，您也可以编写自己的策略！例如，您的自定义策略可以使用指定的代币按自定义的汇率支付中继调用费用。或者，您可以向订阅您的dapp的用户发行ERC721代币，持有订阅代币的账户可以作为订阅的一部分免费使用您的合约。有很多潜在的选择！

要编写自定义策略，只需继承自GSNRecipient并实现acceptRelayedCall、_preRelayedCall和_postRelayedCall函数。

您的acceptRelayedCall实现决定是否接受中继调用：返回_approveRelayedCall以接受，返回_rejectRelayedCall与错误代码以拒绝。

并非所有GSN策略都使用_preRelayedCall和_postRelayedCall（尽管它们仍然必须实现，例如GSNRecipientSignature将它们留空），但在您的策略涉及向终端用户收费时非常有用。

_preRelayedCall应该接受最大可能的费用，而_postRelayedCall在中继调用完成后退还实际费用与最大可能费用之间的差额。当返回_approveRelayedCall以批准中继调用时，还可以返回终端用户地址、maxPossibleCharge、transactionFee和gasPrice数据，以便在_preRelayedCall和_postRelayedCall中使用这些数据。请参考[GSNRecipientERC20Fee](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v2.4.0/contracts/GSN/GSNRecipientERC20Fee.sol)的代码作为示例实现。

一旦您的策略准备就绪，您的GSN接收方只需继承它即可：
```
contract MyContract is MyCustomGSNStrategy {
    constructor() public MyCustomGSNStrategy() {
    }
}
```