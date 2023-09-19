# GSN Strategies
此指南向您展示了使用OpenZeppelin Contracts接受通过Gas Station Network (GSN)Relayer 调用的不同策略。

首先，我们将介绍什么是“GSN策略”，然后展示如何使用两种最常见的策略。最后，我们将介绍如何创建自定义策略。

如果您还在学习Gas Station Network的基础知识，您应该首先阅读[GSN指南](./gsn.md)。

## 解释GSN策略
**GSN策略**决定哪些Relayer 调用被批准，哪些被拒绝。策略是GSN中的一个关键概念。Dapp需要一个策略来防止恶意用户使用Dapp的资金进行Relayer 调用费用。

正如我们在[GSN指南](./gsn.md)中所看到的，为了使您的合约支持GSN，您的合约需要从[GSNRecipient](../API/GSN.md#gsnrecipient)继承。

GSN接收合约需要以下内容才能正常工作：

1. 它需要在RelayHub上存入资金。

2. 它需要以不同的方式处理msg.sender和msg.data。

3. 它需要决定如何批准和拒绝Relayer 调用。

可以通过[GSN Dapp工具](https://gsn.openzeppelin.com/recipients)或使用[OpenZeppelin GSN Helpers](https://docs.openzeppelin.com/gsn-helpers/0.2/api#javascript_interface)以编程方式为GSN接收合约存入资金。

实际用户的msg.sender和msg.data可以通过[GSNRecipient](../API/GSN.md#gsnrecipient)的[_msgSender()](../API/GSN.md#_msgsender-→-address-payable)和[_msgData()](../API/GSN.md#_msgdata-→-bytes)方法安全地获取。

决定如何批准和拒绝Relayer 调用要复杂一些。您可能希望选择哪些用户可以通过GSN使用您的合约，并可能向他们收费，就像夜总会的门卫一样。我们将这些称为GSN策略。

基本的[GSNRecipient](../API/GSN.md#gsnrecipient)合约不包含策略，因此您必须使用预先构建的策略之一或编写自己的策略。我们将首先介绍如何使用已包含的策略：[GSNRecipientSignature](../API/GSN.md#gsnrecipientsignature)和[GSNRecipientERC20Fee](../API/GSN.md#gsnrecipienterc20fee)。

## GSNRecipientSignature
如果用户能够证明您信任的帐户已经批准他们这样做，[GSNRecipientSignature](../API/GSN.md#gsnrecipientsignature)允许用户通过GSN将调用Relayer 到您的接收合约（并向您收费）。他们这样做的方式是通过签名。

Relayer 调用必须包含由作为受信任签名者添加到合约的同一帐户对Relayer 调用参数的签名。如果不是同一个帐户，GSNRecipientSignature将不接受Relayer 调用。

这意味着您需要建立一个系统，其中您信任的帐户对Relayer 调用参数进行签名，然后将其包含在Relayer 调用中，只要它们是有效用户（根据您的业务逻辑）。

有效用户的定义取决于您的系统，但一个示例是已通过某种[OAuth](https://en.wikipedia.org/wiki/OAuth)和验证完成注册的用户，例如通过了验证码或验证了他们的电子邮件地址。您还可以进一步限制，让新用户发送特定数量的Relayer 调用（例如，限制为通过GSN进行5次Relayer 调用，此后用户需要创建钱包）。或者，您可以在链下向用户收费（例如，通过信用卡）以获得系统上的信用，并允许他们创建Relayer 调用，直到信用用尽。

这种设置的好处是，如果您想更改业务规则，**您的合约无需更改**。您只需更改用户免费使用您合约的后端逻辑条件。另一方面，您需要拥有一个后端服务器、微服务或Lambda函数来实现此目的。

### GSNRecipientSignature如何工作？
GSNRecipientSignature根据包含的签名来决定是否接受Relayer 调用。

acceptRelayedCall方法从approvalData中的Relayer 调用参数的签名中恢复地址，并将其与信任的签名者进行比较。如果包含的签名与信任的签名者匹配，则批准Relayer 调用。另一方面，当包含的签名与信任的签名者不匹配时，Relayer 调用将被拒绝，并返回INVALID_SIGNER错误代码。

### 如何使用GSNRecipientSignature
您需要创建一个链下服务（例如后端服务器、微服务或Lambda函数），您的Dapp调用该服务来使用您信任的签名者帐户对Relayer 调用参数进行签名（根据您的业务逻辑决定是否签名）。然后，在Relayer 调用中将签名包含在approvalData中。

您的GSN接收合约将不再直接使用GSNRecipient，而是继承自GSNRecipientSignature，并在GSNRecipientSignature的构造函数中设置信任的签名者，示例如下：
```
import "@openzeppelin/contracts/GSN/GSNRecipientSignature.sol";

contract MyContract is GSNRecipientSignature {
    constructor(address trustedSigner) public GSNRecipientSignature(trustedSigner) {
    }
}
```

> TIP
我们编写了一份详细指南，介绍了如何设置与GSNRecipientSignature配合工作的签名服务器，[快来看看吧](https://forum.openzeppelin.com/t/advanced-gsn-gsnrecipientsignature-sol/1414)！

## GSNRecipientERC20Fee
[GSNRecipientERC20Fee](../API/GSN.md#gsnrecipienterc20fee)略微复杂一些（但不用担心，我们已经为您编写好了！）。与GSNRecipientSignature不同，GSNRecipientERC20Fee不需要任何离链服务。您将向用户提供特殊用途的ERC20代币，而不是对每个Relayer 调用进行离链批准。这些代币用于支付用户Relayer 调用的费用，只要用户拥有足够的代币支付，他们的Relayer 调用就会自动获得批准，并且接收合约将重写他们的交易费用！

每个接收合约都有自己的特殊用途代币。代币与以太坊之间的兑换比率为1:1，因为这些代币用于支付您的合约在使用GSN时的gas费用。

GSNRecipientERC20Fee具有内部的[_mint](../API/GSN.md#_mintaddress-account-uint256-amount)函数。首先，您需要设置一种调用它的方式（例如，添加一个具有某种形式的[访问控制](../Access-Control.md)的公共函数，例如[onlyMinter](../API/Access.md)）。然后，根据您的业务逻辑向用户发行代币。例如，您可以向新用户铸造一定数量的代币，当用户在离链上购买代币时铸造代币，根据用户的订阅情况赠送代币等。

> NOTE
**用户不需要调用approve**来使用他们的代币。它们是修改过的ERC20变体，允许接收合约检索它们。

### GSNRecipientERC20Fee如何工作？
GSNRecipientERC20Fee根据用户代币余额决定是否批准或拒绝Relayer 调用。

acceptRelayedCall函数的实现检查用户的代币余额。如果用户没有足够的代币，则Relayer 调用将被拒绝，并返回INSUFFICIENT_BALANCE错误。如果有足够的代币，则Relayer 调用将被批准，并返回最终用户地址、maxPossibleCharge、transactionFee和gasPrice数据，以便在_preRelayedCall和_postRelayedCall中使用。

在_preRelayedCall函数中，将转移maxPossibleCharge数量的代币到接收合约。假设Relayer 调用将使用所有可用的gas，转移所需的最大数量的代币。然后，在_postRelayedCall函数中，计算实际金额，包括接收合约实现和ERC20代币转移，并退还差额。

在_preRelayedCall中转移所需的最大数量的代币是为了保护合约免受攻击（这与以太坊交易中锁定以太的方式非常相似）。

> NOTE
gas费用估算并不是100%准确，我们可能会在以后进行微调。

> NOTE
始终使用_preRelayedCall和_postRelayedCall函数。内部使用_preRelayedCall和_postRelayedCall函数，而不是公共的preRelayedCall和postRelayedCall函数，因为非RelayHub合约无法调用公共函数。

### 如何使用GSNRecipientERC20Fee
您的GSN接收合约需要继承GSNRecipientERC20Fee，并具有适当的[访问控制](../Access-Control.md)（用于代币铸造），在GSNRecipientERC20Fee的构造函数中设置代币详细信息，并根据以下示例代码（使用[AccessControl](../API/Access.md)）创建一个适当受保护的公共铸币函数。
```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/GSN/GSNRecipientERC20Fee.sol";
import "@openzeppelin/contracts/access/AccessControl.sol";

contract MyContract is GSNRecipientERC20Fee, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor() public GSNRecipientERC20Fee("FeeToken", "FEE") {
        _setupRole(MINTER_ROLE, _msgSender());
    }

    function _msgSender() internal view override(Context, GSNRecipient) returns (address payable) {
        return GSNRecipient._msgSender();
    }

    function _msgData() internal view override(Context, GSNRecipient) returns (bytes memory) {
        return GSNRecipient._msgData();
    }

    function mint(address account, uint256 amount) public {
        require(hasRole(MINTER_ROLE, _msgSender()), "Caller is not a minter");
        _mint(account, amount);
    }
}
```

## 自定义策略
如果包含的策略不完全符合您的业务需求，您也可以编写自己的策略！例如，您的自定义策略可以使用指定的代币以自定义的汇率支付Relayer 调用费用。或者您可以向订阅您的dapp的用户发行ERC721代币，并且持有订阅代币的账户可以作为订阅的一部分免费使用您的合约。有很多潜在的选择！

要编写自定义策略，只需从GSNRecipient继承并实现acceptRelayedCall、_preRelayedCall和_postRelayedCall函数。

您的acceptRelayedCall实现决定是否接受Relayer 调用：返回_approveRelayedCall以接受，返回带有错误代码的_rejectRelayedCall以拒绝。

并非所有GSN策略都使用_preRelayedCall和_postRelayedCall（尽管它们仍然必须实现，例如GSNRecipientSignature将它们留空），但在涉及向终端用户收费时很有用。

_preRelayedCall应该考虑到最大可能的费用，而_postRelayedCall在Relayer 调用完成后退还任何实际费用的差额。当返回_approveRelayedCall以批准Relayer 调用时，还可以返回终端用户的地址、最大可能的费用、交易费和gas价格数据，以便在_preRelayedCall和_postRelayedCall中使用这些数据。有关示例实现，请参阅[GSNRecipientERC20Fee的代码](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.0.0/contracts/GSN/GSNRecipientERC20Fee.sol)。

一旦您的策略准备就绪，您的GSN接收者只需从Relayer 继承它即可：
```
contract MyContract is MyCustomGSNStrategy {
    constructor() public MyCustomGSNStrategy() {
    }
}
```
