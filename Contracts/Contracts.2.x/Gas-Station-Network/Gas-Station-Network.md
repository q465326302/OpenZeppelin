# Writing GSN-capable contracts
[Gas Station Network](https://gsn.openzeppelin.com/)（GSN）允许您构建应用程序，其中您为用户的交易付费，因此他们无需持有Ether来支付燃气费，从而简化了他们的入门流程。在本指南中，我们将学习如何编写智能合约，以便可以从GSN接收交易，并使用OpenZeppelin Contracts。

如果您对GSN还不熟悉，您可能首先要查看该[系统的概述](/Learn/Sending%20gasless%20transactions/Sending-gasless-transactions.md)，以更清楚地了解如何实现无燃气交易。否则，请准备好！

## 接收Relayer 调用
编写接收者的第一步是继承我们的GSNRecipient合约。如果您还继承了其他合约，例如ERC20或Crowdsale，这也没问题：将GSNRecipient添加到您的所有代币或众筹函数中将使它们成为GSN可调用的函数。
```
import "@openzeppelin/contracts/GSN/GSNRecipient.sol";

contract MyContract is GSNRecipient, ... {

}
```

### msg.sender and msg.data
在处理GSN接收者合约时，只有一个额外的细节需要注意：永远不要直接使用msg.sender或msg.data。在Relayer 调用中，msg.sender将是RelayHub，而不是您的用户！但这并不意味着您将无法获取您的用户地址：GSNRecipient提供了两个函数，_msgSender()和_msgData()，它们是msg.sender和msg.data的替代函数，同时处理了底层细节。只要您使用这两个函数而不是原始的获取器，您就可以放心使用！

> WARNING
您从第三方合约继承的合约可能不使用这些替代函数，因此在与GSNRecipient混合使用时可能不安全。如果有疑问，请前往我们的[支持论坛](https://forum.openzeppelin.com/c/support)。

### Accepting and Charging
与常规合约函数调用不同，每个Relayer 调用都需要经过额外的步骤，这些步骤是IRelayRecipient接口在您的合约上调用的函数。GSNRecipient包括此接口，但没有实现：编写接收者的大部分工作涉及处理这些函数调用。它们被设计为提供灵活性，但基本的接收者可以安全地忽略其中大部分，同时仍然安全可靠。

OpenZeppelin Contracts提供了一些经过验证的方法供您直接使用，但您仍然应该对底层发生的事情有一个基本的了解。

#### acceptRelayedCall
首先，RelayHub将询问您的接收者合约是否希望接收Relayer 调用。请记住， Relayer 将向您收取产生的燃气费用，因此您应该只接受您愿意支付的调用！
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

有多种方法可以使其工作，包括：

1. 拥有可信用户的白名单

2. 仅接受对一个入职功能的调用

3. 使用代币（可能由您发行）向用户收费

4. 将接受逻辑委托给链下

所有Relayer 调用请求都可以被拒绝，对接收者没有任何费用。

在此函数中，您返回一个数字来指示您是否接受调用（0）或不接受（任何其他数字）。您还可以返回一些传递给以下两个函数（pre和post）的任意数据作为执行上下文。

### preRelayedCall和postRelayedCall
在接受Relayer 调用后，RelayHub将为您的合约提供两次机会来向用户收取调用费用，执行一些簿记等操作：在实际Relayer 调用之前和之后。这些函数分别称为preRelayedCall和postRelayedCall。

```
function preRelayedCall(bytes calldata context) external returns (bytes32);
```

preRelayedCall会告知您通话可能的最高费用，并可以用于提前向用户收费。如果用户可能在通话过程中使用他们的津贴，这将非常有用，因此您可以在此处锁定一些资金。

```
function postRelayedCall(
  bytes calldata context,
  bool success,
  uint actualCharge,
  bytes32 preRetVal
) external;
```

postRelayedCall将为您提供交易成本的准确估计，这使其成为向用户收费的自然选择。它还可以让您了解Relayer 调用是否已还原。这使您可以选择不向还原的调用收费，但请记住，您仍将被Relayer 方收费。

这些函数使您能够实现一种流程，例如，您可以使用自定义代币向用户收取Relayer 交易的费用。您可以在pre中锁定一些他们的代币，并在post中执行实际的收费。这类似于以太坊中的燃料费用的工作原理：网络首先锁定足够的ETH以支付交易的燃料限制和燃料价格，然后支付实际花费的费用。

## 进一步阅读
阅读我们在OpenZeppelin Contracts中预构建和发布的[付款策略指南](./Strategies.md)，或查看GSN[基础合约的API参考](../API/GSN.md)。