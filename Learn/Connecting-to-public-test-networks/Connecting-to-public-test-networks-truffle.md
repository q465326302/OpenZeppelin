# 连接到公共测试网络

在[编写合约](../Developing-smart-contracts/Developing-smart-contracts-truffle.md)、[本地测试](../Deploying-and-interacting/Deploying-and-interacting-truffle.md)并进行[充分测试](../Writing-automated-tests/Writing-automated-tests-truffle.md)后，就可以进入持久的公共测试环境，让你和测试用户开始与你的应用程序交互。

我们将使用**公共测试网络**（也称为测试网络）进行测试，这些网络类似于主要的以太坊网络，但以太没有价值且可以免费获取，因此非常适合免费测试你的合约。

在本指南中，我们将使用我们熟悉的[Box合约](../Developing-smart-contracts/Developing-smart-contracts-truffle.md#使用openzeppelin-contracts)，并将其部署到测试网络上，同时学习以下内容：

- [连接到公共测试网络](#连接到公共测试网络)
  - [可用的测试网络](#可用的测试网络)
  - [将项目连接到公共网络](#将项目连接到公共网络)
    - [访问测试网络节点](#访问测试网络节点)
    - [创建新账户](#创建新账户)
    - [配置网络](#配置网络)
    - [为测试网账户提供资金支持](#为测试网账户提供资金支持)
  - [在测试网络上工作](#在测试网络上工作)
  - [下一步](#下一步)


请记住，将项目部署到公共测试网络是开发以太坊项目的必要步骤。它们提供了一个安全的测试环境，可以紧密模拟主网络 - 你不希望在一个会让你和用户损失资金的网络中测试你的项目！

> NOTE
有关*Truffle*和*Hardhat*的说明都可用。使用此切换选择你的首选项！

## 可用的测试网络
有两个可供选择的测试网络，每个都具有自己的特点：

**Sepolia**
一种权益证明网络。这意味着验证者明确地将资本以ETH的形式抵押到智能合约中，该合约作为抵押品在不当行为时被销毁。(id=11155111)

**Goerli**
也是一种权益证明网络，兼容Geth和OpenEthereum客户端，具有15秒的块时间。(id=5)

> NOTE
每个网络都有一个数字ID进行标识。本地网络通常具有大的随机值，而id = 1则保留用于主要的以太坊网络。

## 将项目连接到公共网络
要将我们的项目连接到公共测试网络，我们需要：
1. [获取测试网络节点](#访问测试网络节点)

2. [创建一个新账户](#创建新账户)

3. [更新我们的配置文件](#配置网络)

4. [我们的测试账户提供资金](#为测试网账户提供资金支持)

### 访问测试网络节点

虽然你可以自己创建连接到测试网络的[Geth](https://github.com/ethereum/go-ethereum/wiki/Command-Line-Options)或[OpenEthereum](https://openethereum.github.io/wiki/Chain-specification)节点，但最简单的访问测试网络的方法是通过公共节点服务，例如[Alchemy](https://alchemy.com/)或[Infura](https://infura.io/)。Alchemy和Infura通过免费和付费计划提供所有测试网络和主网络的公共节点访问。

> NOTE
当一个节点可以被公众访问，并且不管理任何账户时，我们称其为公共节点。这意味着它可以回复查询和Relayer 已签名的交易，但不能自行签署交易。

在本指南中，我们将使用Alchemy，但你也可以使用[Infura](https://infura.io/)或其他公共节点提供者。

前往[Alchemy](https://dashboard.alchemyapi.io/signup?referral=53fcee38-b894-4d5f-bd65-885d241f8d29)（包括推荐代码），注册并记下你分配的API密钥-我们稍后将使用它连接到网络。

### 创建新账户
要在测试网发送交易，你需要一个新的以太坊账户。有许多方法可以做到这一点：在这里，我们将使用助记词包，它将输出一个新的助记词（一组12个单词），我们将使用它来派生我们的账户：
```
npx mnemonics
drama film snack motion ...
```

> WARNING
确认保护好你的助记词，不要将 secrets 信息上传到网络中。即使只是为了测试目的，仍有恶意用户会为了娱乐而破坏你的测试网络部署！

### 配置网络
由于我们使用公共节点，因此我们需要在本地签署所有交易。我们将使用我们的助记词和Alchemy端点来配置网络。

> NOTE
这部分假定你已经设置好了一个项目。如果你还没有这样做，请转到[设置Solidity项目的指南](../Developing-smart-contracts/Developing-smart-contracts-truffle.md)。

让我们从安装[@truffle/hdwallet-provider](https://github.com/trufflesuite/truffle/tree/master/packages/hdwallet-provider)开始：
```
npm install --save-dev @truffle/hdwallet-provider
```

我们需要更新我们的配置文件，加入一个连接到测试网络的新网络配置。在这里，我们将使用Goerli，但你可以选择任何你想要的网络。
```
// truffle-config.js
+const { alchemyApiKey, mnemonic } = require('./secrets.json');
+const HDWalletProvider = require('@truffle/hdwallet-provider');

 module.exports = {
   ...
   networks: {
     development: {
      ...
     },
+    goerli: {
+      provider: () => new HDWalletProvider(
+        mnemonic, `https://eth-goerli.alchemyapi.io/v2/${alchemyApiKey}`,
+      ),
+      network_id: 5,
+      gasPrice: 10e9,
+      skipDryRun: true,
+    },
   },
   ...
 };
```

> NOTE
请查看[HDWalletProvider文档](https://github.com/trufflesuite/truffle/tree/master/packages/hdwallet-provider)以获取有关配置选项的信息。

请注意代码第一行中，我们正在从一个名为 secrets.json 的文件中加载项目 ID 和助记词，该文件应该看起来像以下内容，但要使用你自己的值。请务必将其加入 .gitignore 中，以确保你不会将机密信息上传网络中！

```
{
  "mnemonic": "drama film snack motion ...",
  "alchemyApiKey": "JPV2..."
}
```

> TIP
你可以使用任何你喜欢的项目 secrets 管理解决方案，而不是一个secrets.json文件。一个流行且简单的选项是使用[dotenv](https://github.com/motdotla/dotenv)将 secrets 作为环境变量注入。

现在我们可以通过列出我们在goerli网络中可用的账户来测试这个配置是否工作。请记住，你的账户将会不同，因为它们取决于你使用的助记词。
```
npx truffle console --network goerli
truffle(goerli)> accounts
[ '0xEce6999C6c5BDA71d673090144b6d3bCD21d13d4',
  '0xC1310ade58A75E6d4fCb8238f9559188Ea3808f9',
... ]
```

我们也可以通过查询账户余额来测试与节点的连接。
```
> await web3.eth.getBalance(accounts[0])
'0'
```

空的！这指向我们下一个任务：获取测试网资金，以便我们可以发送交易。

### 为测试网账户提供资金支持
大多数公共测试网都有一个水龙头：一个可以免费提供少量测试以太的网站。如果你在goerli网络上，请前往Alchemy的免费[Goerli水龙头获取免费的测试ETH](https://goerlifaucet.com/)。或者，你也可以使用MetaMask的水龙头直接向你的[MetaMask账户](https://faucet.metamask.io/)请求资金。如果你需要在Sepolia网络上获得测试以太，可以使用[Infura的免费Sepolia水龙头](https://www.infura.io/faucet)获取免费的测试ETH。

有了资金支持的账户，让我们将合约部署到测试网上！

## 在测试网络上工作
通过配置项目以在公共测试网络上运行，我们现在终于可以[部署我们的Box合约](../Deploying-and-interacting/Deploying-and-interacting-truffle.md#部署智能合约)了。这里的命令与你在[本地开发网络](../Deploying-and-interacting/Deploying-and-interacting-truffle.md#建立本地区块链)上的命令完全相同，但是由于会挖掘新块，所以需要等待几秒钟才能运行。
```
npx truffle migrate --network goerli
...
2_deploy.js
===========

   Deploying 'Box'
   ---------------
   > transaction hash:    0x56237c80fc5b562736d27dc12560241706849cebe01c46b7c080dad596a65f28
   > Blocks: 0            Seconds: 6
   > contract address:    0xA4D767f2Fba05242502ECEcb2Ae97232F7611353
...
```

就是这样！你的Box合约实例将永久存储在测试网络中，并且对任何人都可以公开访问。

你可以在区块浏览器（如[Etherscan](https://etherscan.io/)）上查看你的合约。请记住，在你部署合约的测试网络上访问浏览器，例如在goerli上访问[goerli.etherscan.io](https://goerli.etherscan.io/)。

> TIP
你可以在上面的示例中查看我们部署的合约以及发送到它的所有交易，[链接](https://goerli.etherscan.io/address/0xA4D767f2Fba05242502ECEcb2Ae97232F7611353)在此。

你也可以像平常一样与实例进行交互，可以使用[控制台或编程方式](../Deploying%20and%20interacting/Deploying%20and%20interacting-truffle.md)。
```
npx truffle console --network goerli
truffle(goerli)> box = await Box.deployed()
undefined
truffle(goerli)> await box.store(42)
{ tx:
   '0x7d1a556b34fcb26855c6646669fc926738b05589591de6c7bb1f8773546817e7',
...
truffle(goerli)> (await box.retrieve()).toString()
'42'
```

请记住，每笔交易都会产生一定的gas，因此你最终需要用更多的资金来充值你的账户。

## 下一步
在公共测试网络上彻底测试你的应用程序之后，你已经准备好进行开发旅程的最后一步：[在生产环境中部署你的应用程序](../Preparing-for-mainnet/Preparing-for-mainnet.md)。