# Crowdsales
众售是以太坊的一种流行用途；它们允许您以各种方式将代币分配给网络参与者，主要是以Ether作为交换。它们有各种形式和口味，因此让我们看看OpenZeppelin Contracts中提供的各种类型以及如何使用它们。

众售有许多不同的属性，但以下是一些重要的属性：

* 价格和汇率配置

* 您的众售以固定价格出售代币吗？

* 价格是否随时间变化或根据需求变化？

* 发行

* 这些代币实际上是如何发送给参与者的？

* 验证 - 谁可以购买代币？

* 是否进行KYC / AML检查？

* 代币是否有最大限额？

* 如果该限额是针对每个参与者的？

* 是否有开始和结束时间范围？

* 分销

* 资金的分配是在众售后实时进行还是？

* 如果目标未达成，参与者是否可以退款？

为了管理所有不同的众售组合和口味，*Contracts*提供了一个高度可配置的众售基础合约，可以与各种其他功能结合使用，构建定制的众售。

## 众售汇率
了解众售汇率非常重要，这里的错误是常见的错误来源。

✨ 等一下，这很重要 ✨
首先，**所有货币数学运算都是在该货币的最小单位上进行的，并在显示货币时转换为正确的小数位数**。

这意味着当您在智能合约中进行数学运算时，您需要理解您正在添加、除以和乘以货币的最小金额（如wei），而不是常用的显示货币值（以太币）。

在以太币中，货币的最小单位是wei，1 ETH === 10^18 wei。在代币中，过程非常类似：1 TKN === 10^(decimals) TKNbits。

* 代币的最小单位是“bits”或TKNbits。

* 代币的显示值是TKN，即TKNbits * 10^(decimals)

人们通常称为“一个代币”的实际上是一堆TKNbits，显示为1 TKN。这与以太币和wei之间的关系相同。您始终在进行的计算是TKNbits和wei。

因此，如果您想要为每2 wei发行“一个代币”，并且您的小数位数为18，则您的汇率为0.5e18。然后，当我向您发送2 wei时，您的众售会向我发行2 * 0.5e18 TKNbits，这正好等于10^18 TKNbits，并显示为1 TKN。

如果您想为每1 ETH发行“1 TKN”，并且您的小数位数为18，则您的汇率为1。这是因为实际上在数学运算中发生的是合约看到用户发送了10^18 wei，而不是1 ETH。然后它使用您的汇率1来计算TKNbits = rate * wei，或者1 * 10^18，仍然是10^18。由于您的小数位数为18，这显示为1 TKN。

再举一个例子：如果我想为每美元（USD）以以太币发行“1 TKN”，我们将按以下方式计算：

* 假设1 ETH == 400美元

* 因此，10^18 wei = 400美元

* 因此，1美元等于10^18 / 400，即2.5 * 10^15 wei

* 我们的小数位数为18，因此我们将使用10 ^ 18 TKNbits而不是1 TKN

* 因此，如果参与者向众售发送2.5 * 10^15 wei，我们应该给他们10 ^ 18 TKNbits

* 因此，汇率为2.5 * 10^15 wei === 10^18 TKNbits，或者1 wei = 400 TKNbits

* 因此，我们的汇率为400

（当您保留18个小数位数时，这个过程非常简单，与以太币/wei相同）

## 代币发行
您必须做出的首要决定之一是“如何将这些代币分发给用户？”通常有以下三种方式：

* （默认） - 众售合约拥有代币，并将代币从自己的所有权转移到购买它们的用户。

* *MintedCrowdsale* - 众售在进行购买时铸造代币。

* *AllowanceCrowdsale* - 众售获得给予另一个已经拥有要在众售中出售的代币的钱包（如Multisig）的津贴。

### 默认发行
在默认情况下，您的众售必须拥有要出售的代币。您可以通过各种方法将代币发送给众售，但在Solidity中的代码如下：
```
IERC20(tokenAddress).transfer(CROWDSALE_ADDRESS, SOME_TOKEN_AMOUNT);
```

然后当您部署您的众售时，只需告诉它有关代币的信息。
```
new Crowdsale(
    1,             // rate in TKNbits
    MY_WALLET,     // address where Ether is sent
    TOKEN_ADDRESS  // the token contract address
);
```

### Minted Crowdsale
要使用*MintedCrowdsale*，您的代币还必须是一个*ERC20Mintable*代币，众筹活动有权从中进行铸币。这可以如下所示：
```
contract MyToken is ERC20, ERC20Mintable {
    // ... see "Tokens" for more info
}

contract MyCrowdsale is Crowdsale, MintedCrowdsale {
    constructor(
        uint256 rate,    // rate in TKNbits
        address payable wallet,
        IERC20 token
    )
        MintedCrowdsale()
        Crowdsale(rate, wallet, token)
        public
    {

    }
}

contract MyCrowdsaleDeployer {
    constructor()
        public
    {
        // create a mintable token
        ERC20Mintable token = new MyToken();

        // create the crowdsale and tell it about the token
        Crowdsale crowdsale = new MyCrowdsale(
            1,               // rate, still in TKNbits
            msg.sender,      // send Ether to the deployer
            token            // the token
        );
        // transfer the minter role from this contract (the default)
        // to the crowdsale, so it can mint tokens
        token.addMinter(address(crowdsale));
        token.renounceMinter();
    }
}
```
### AllowanceCrowdsale
使用*AllowanceCrowdsale*将代币从另一个钱包发送给众筹参与者。为了使其工作，源钱包必须通过ERC20的*approve*方法授予众筹合约一个授权额度。
```
contract MyCrowdsale is Crowdsale, AllowanceCrowdsale {
    constructor(
        uint256 rate,
        address payable wallet,
        IERC20 token,
        address tokenWallet  // <- new argument
    )
        AllowanceCrowdsale(tokenWallet)  // <- used here
        Crowdsale(rate, wallet, token)
        public
    {

    }
}
```
在创建众筹销售后，不要忘记批准使用你的代币！
```
IERC20(tokenAddress).approve(CROWDSALE_ADDRESS, SOME_TOKEN_AMOUNT);
```

## 验证
您的众筹可能是以下几种验证要求的一部分：

* *CappedCrowdsale* - 为您的众筹添加了一个上限，使超过该上限的购买无效。

* *IndividuallyCappedCrowdsale* - 限制个人的贡献额。

* *WhitelistCrowdsale* - 只允许白名单参与者购买代币。这对于将您的KYC / AML白名单上链非常有用！

* *TimedCrowdsale* - 为您的众筹添加了一个*开放时间*和*结束时间*。

只需根据您的需要随意混合和匹配这些众筹方式。
```
contract MyCrowdsale is Crowdsale, CappedCrowdsale, TimedCrowdsale {

    constructor(
        uint256 rate,            // rate, in TKNbits
        address payable wallet,  // wallet to send Ether
        IERC20 token,            // the token
        uint256 cap,             // total cap, in wei
        uint256 openingTime,     // opening time in unix epoch seconds
        uint256 closingTime      // closing time in unix epoch seconds
    )
        CappedCrowdsale(cap)
        TimedCrowdsale(openingTime, closingTime)
        Crowdsale(rate, wallet, token)
        public
    {
        // nice, we just created a crowdsale that's only open
        // for a certain amount of time
        // and stops accepting contributions once it reaches `cap`
    }
}
```

## 分发
每个众售的生命周期中都会有一个时刻，它必须放弃它所托管的代币。这是您决定何时发生的！

默认行为是在参与者购买代币时释放它们，但有时可能并不理想。例如，如果我们希望在销售中未达到最低筹集目标时为用户提供退款怎么办？或者，出于合规原因，也许我们想在销售结束后等待用户认领他们的代币并开始交易？

OpenZeppelin Contracts在这方面提供了便利！

### PostDeliveryCrowdsale
正如其名称所示，*PostDeliveryCrowdsale*在众售结束后分发代币，让用户调用*withdrawTokens*来认领他们购买的代币。

```
contract MyCrowdsale is Crowdsale, TimedCrowdsale, PostDeliveryCrowdsale {

    constructor(
        uint256 rate,            // rate, in TKNbits
        address payable wallet,  // wallet to send Ether
        IERC20 token,            // the token
        uint256 openingTime,     // opening time in unix epoch seconds
        uint256 closingTime      // closing time in unix epoch seconds
    )
        PostDeliveryCrowdsale()
        TimedCrowdsale(openingTime, closingTime)
        Crowdsale(rate, wallet, token)
        public
    {
        // nice! this Crowdsale will keep all of the tokens until the end of the crowdsale
        // and then users can `withdrawTokens()` to get the tokens they're owed
    }
}
```

### RefundableCrowdsale
*RefundableCrowdsale*提供了如果达不到最低目标的用户可以退款的服务。如果目标未达到，用户可以*申请Refund*来取回他们的以太币。