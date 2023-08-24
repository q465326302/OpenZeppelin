# Writing GSN-capable contracts
[Gas Station Network](https://gsn.openzeppelin.com/)（GSN）允许您构建应用程序，在这些应用程序中，您为用户的交易付费，因此他们无需持有Ether来支付燃气费，从而简化了他们的入门流程。在本指南中，我们将学习如何编写智能合约，以便从GSN接收交易，使用OpenZeppelin合约。

如果您对GSN还不熟悉，您可能首先要查看系统概述，以更清晰地了解如何实现无燃气费交易。否则，准备好了！

## 接收Relayer 调用
编写接收者的第一步是从我们的GSNRecipient合约继承。如果您还从其他合约（如ERC20）继承，这也可以正常工作：添加GSNRecipient将使您所有的代币函数都可通过GSN调用。
```
import "@openzeppelin/contracts/GSN/GSNRecipient.sol";

contract MyContract is GSNRecipient, ... {

}
```

### msg.sender and msg.data
当使用GSN接收合约时，只有一个额外的细节需要注意：您绝不能直接使用msg.sender或msg.data。在Relayer 调用中，msg.sender将是RelayHub而不是您的用户！但这并不意味着您将无法检索用户的地址：GSNRecipient提供了两个函数_msgSender()和_msgData()，它们是msg.sender和msg.data的替代函数，同时处理底层细节。只要您使用这两个函数而不是原始的获取器，您就可以放心使用！

> WARNING
当您与GSNRecipient混合使用时，从您继承的第三方合约可能不使用这些替代函数，这使得它们在使用时不安全。如果有疑问，请前往我们的[支持论坛](https://forum.openzeppelin.com/c/support)。

### 接受和收费
与常规合约函数调用不同，每个Relayer 调用都必须经过额外的一些步骤，这些步骤是RelayHub将在您的合约上调用的IRelayRecipient接口的函数。GSNRecipient包含了这个接口，但没有实现：大部分编写接收者的工作涉及处理这些函数调用。它们被设计为提供灵活性，但基本的接收者可以安全地忽略其中大部分而仍然安全可靠。

OpenZeppelin Contracts提供了一些经过试验和测试的方法供您直接使用，但您仍然应该对底层发生的事情有一个基本的了解。

#### acceptRelayedCall
首先，RelayHub将询问您的接收者合约是否希望接收Relayer 调用。请记住，您将为 Relayer 产生的燃气消耗费用付费，因此您只应该接受您愿意支付的调用！
```
    function acceptRelayedCall(
        address relay,
        address from,
        bytes calldata encodedFunction,
        uint256 transactionFee,
        uint256 gasPrice,
        uint256 gasLimit,
        uint256 nonce,
        bytes calldata approvalData,
        uint256 maxPossibleCharge
    ) external view returns (uint256, bytes memory);
```

有多种方法可以使这个工作，包括：

1. 有一个受信任用户的白名单
2. 只接受对一个入职功能的调用
3. 以代币的形式向用户收费（可能由您发行）
4. 将接受逻辑委托到链下

所有Relayer 调用请求都可以被拒绝，对接收方没有任何费用。

在这个函数中，您返回一个数字，表示您是否接受调用（0）或不接受（其他数字）。您还可以返回一些任意的数据，将其作为执行上下文传递给后续的两个函数（pre和post）。

### pre和postRelayedCall
在接受Relayer 调用后，RelayHub将为您的合约提供两次机会来为用户的调用收费、进行一些簿记等操作：在实际Relayer 调用之前和之后。这些函数分别被称为preRelayedCall和postRelayedCall。
```
function preRelayedCall(bytes calldata context) external returns (bytes32);
```

preRelayedCall将告知您呼叫可能的最高费用，并可用于提前向用户收费。如果用户可能会在呼叫中花费他们的津贴，这将非常有用，因此您可以在此处锁定一些资金。
```
    function postRelayedCall(
        bytes calldata context,
        bool success,
        uint256 actualCharge,
        bytes32 preRetVal
    ) external;
```

postRelayedCall将为您提供准确的交易成本估计，这使其成为向用户收费的自然选择。它还将告诉您Relayer 调用是否恢复。这样，您就可以不收取用户恢复调用的费用，但请记住，您仍然会被Relayer 方收费。

这些功能使您能够实现一种流程，例如，您可以使用自定义代币向用户收取Relayer 交易的费用。您可以在pre中锁定部分他们的代币，并在post中执行实际的收费。这类似于以太坊中的燃气费用工作原理：网络首先锁定足够的ETH以支付交易的燃气限制和燃气价格，然后支付实际花费的金额。

## 进一步阅读
阅读我们在[OpenZeppelin Contracts](./Strategies.md)中预构建和发布的支付策略指南，或查看[GSN基础合约的API参考](../API/GSN.md)。
