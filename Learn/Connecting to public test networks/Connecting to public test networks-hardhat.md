# 连接到公共测试网络
在*编写合同*、*本地测试*并进*行充分测试*后，就可以进入持久的公共测试环境，让您和测试用户开始与您的应用程序交互。

我们将使用**公共测试网络**（也称为测试网络）进行测试，这些网络类似于主要的以太坊网络，但以太币没有价值且可以免费获取，因此非常适合免费测试您的合同。

在本指南中，我们将使用我们喜爱的*Box合约*，并将其部署到测试网络上，同时学习以下内容：

* 可用的测试网络

* 如何为测试网络设置项目

* 如何部署和与测试网络合同实例进行交互

请记住，将项目部署到公共测试网络是开发以太坊项目的必要步骤。它们提供了一个安全的测试环境，可以紧密模拟主网络 - 您不希望在一个会让您和用户损失资金的网络中测试您的项目！

>NOTE
有关Truffle和Hardhat的说明都可用。使用此切换选择您的首选项！

## 可用的测试网络
有两个可供选择的测试网络，每个都具有自己的特点：

Sepolia
一种权益证明网络。这意味着验证者明确地将资本以ETH的形式抵押到智能合约中，该合约作为抵押品在不当行为时被销毁。(id=11155111)

Goerli
也是一种权益证明网络，兼容Geth和OpenEthereum客户端，具有15秒的块时间。(id=5)

>NOTE
每个网络都有一个数字ID进行标识。本地网络通常具有大的随机值，而id = 1则保留用于主要的以太坊网络。

## 将项目连接到公共网络
要将我们的项目连接到公共测试网络，我们需要：
1. *获取测试网络节点*

2. *创建一个新账户*

3. *更新我们的配置文件*

4. *为我们的测试账户提供资金*

### 访问测试网络节点

虽然您可以自己创建连接到测试网络的[Geth](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)或[OpenEthereum](https://openethereum.github.io/wiki/Chain-specification)节点，但最简单的访问测试网络的方法是通过公共节点服务，例如[Alchemy](https://alchemy.com/)或[Infura](https://infura.io/)。Alchemy和Infura通过免费和付费计划提供所有测试网络和主网络的公共节点访问。

>NOTE
当一个节点可以被公众访问，并且不管理任何账户时，我们称其为公共节点。这意味着它可以回复查询和中继已签名的交易，但不能自行签署交易。

在本指南中，我们将使用Alchemy，但您也可以使用[Infura](https://infura.io/)或其他公共节点提供者。

前往[Alchemy](https://dashboard.alchemyapi.io/signup?referral=53fcee38-b894-4d5f-bd65-885d241f8d29)（包括推荐代码），注册并记下您分配的API密钥-我们稍后将使用它连接到网络。

### 创建新账户
要在测试网发送交易，您需要一个新的以太坊账户。有许多方法可以做到这一点：在这里，我们将使用助记词包，它将输出一个新的助记词（一组12个单词），我们将使用它来派生我们的账户：
```
npx mnemonics
drama film snack motion ...
```

>WARNING
确认保护好你的助记词，不要将秘密信息提交到版本控制中。即使只是为了测试目的，仍有恶意用户会为了娱乐而破坏你的测试网络部署！

### 配置网络
由于我们使用公共节点，因此我们需要在本地签署所有交易。我们将使用我们的助记词和Alchemy端点来配置网络。

>NOTE
这部分假定您已经设置好了一个项目。如果您还没有这样做，请转到*设置Solidity项目的指南*。

我们需要更新我们的配置文件，添加一个连接到测试网络的新网络连接。在这里，我们将使用Goerli，但您可以使用任何您想要的网络。
```
// hardhat.config.js
+ const { alchemyApiKey, mnemonic } = require('./secrets.json');
...
  module.exports = {
+    networks: {
+     goerli: {
+       url: `https://eth-goerli.alchemyapi.io/v2/${alchemyApiKey}`,
+       accounts: { mnemonic: mnemonic },
+     },
+   },
...
};
```

>NOTE
请查看[HDWalletProvider文档](https://github.com/trufflesuite/truffle/tree/master/packages/hdwallet-provider)以获取有关配置选项的信息。

请注意第一行中，我们正在从一个名为 secrets.json 的文件中加载项目 ID 和助记词，该文件应该看起来像以下内容，但要使用您自己的值。请务必将其加入 .gitignore 中，以确保您不会将机密信息提交到版本控制中！

```
{
  "mnemonic": "drama film snack motion ...",
  "alchemyApiKey": "JPV2..."
}
```

>TIP
你可以使用任何你喜欢的项目秘密管理解决方案，而不是一个secrets.json文件。一个流行且简单的选项是使用[dotenv](https://github.com/motdotla/dotenv)将秘密作为环境变量注入。

现在我们可以通过列出我们在goerli网络中可用的账户来测试这个配置是否工作。请记住，您的账户将会不同，因为它们取决于您使用的助记词。
```
npx hardhat console --network goerli
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> accounts = await ethers.provider.listAccounts()
[
  '0xEce6999C6c5BDA71d673090144b6d3bCD21d13d4',
  '0xC1310ade58A75E6d4fCb8238f9559188Ea3808f9',
...
]
```
我们还可以通过查询我们的账户余额来测试与节点的连接。
```
> (await ethers.provider.getBalance(accounts[0])).toString()
'0'
```
空的！这指向我们下一个任务：获取测试网资金，以便我们可以发送交易。

### 为测试网账户提供资金支持
大多数公共测试网都有一个水龙头：一个可以免费提供少量测试以太币的网站。如果你在goerli网络上，请前往Alchemy的免费[Goerli水龙头获取免费的测试ETH](https://goerlifaucet.com/)。或者，你也可以使用MetaMask的水龙头直接向你的[MetaMask账户](https://faucet.metamask.io/)请求资金。如果你需要在Sepolia网络上获得测试以太币，可以使用[Infura的免费Sepolia水龙头](https://www.infura.io/faucet)获取免费的测试ETH。

有了资金支持的账户，让我们将合约部署到测试网上！

## 在测试网络上工作
通过配置项目以在公共测试网络上运行，我们现在终于可以*部署我们的Box合约*了。这里的命令与您在*本地开发网络*上的命令完全相同，但是由于会挖掘新块，所以需要几秒钟才能运行。
```
npx hardhat run --network goerli scripts/deploy.js
Deploying Box...
Box deployed to: 0xD7fBC6865542846e5d7236821B5e045288259cf0
```
就是这样！您的Box合约实例将永久存储在测试网络中，并且对任何人都可以公开访问。

您可以在区块浏览器（如[Etherscan](https://etherscan.io/)）上查看您的合约。请记住，在您部署合约的测试网络上访问浏览器，例如在goerli上访问[goerli.etherscan.io](https://goerli.etherscan.io/)。

>TIP
您可以在上面的示例中查看我们部署的合约以及发送到它的所有交易，[链接](https://goerli.etherscan.io/address/0xA4D767f2Fba05242502ECEcb2Ae97232F7611353)在此。

您也可以像平常一样与实例进行交互，可以使用*控制台*或*编程方式*。
```
npx hardhat console --network goerli
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0xD7fBC6865542846e5d7236821B5e045288259cf0');
undefined
> await box.store(42);
{
  hash: '0x330e331d30ee83f96552d82b7fdfa6156f9f97d549a612eeef7283d18b31d107',
...
> (await box.retrieve()).toString()
'42'
```
请记住，每笔交易都会产生一定的gas，因此您最终需要用更多的资金来充值您的账户。

>下一步
在公共测试网络上彻底测试您的应用程序之后，您已经准备好进行开发旅程的最后一步：*在生产环境中部署您的应用程序*。