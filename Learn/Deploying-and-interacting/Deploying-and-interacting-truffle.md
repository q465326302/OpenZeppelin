# 部署和交互智能合约
与大多数软件不同，智能合约不在您的计算机或某个服务器上运行：它们运行在以太坊网络本身上。这意味着与它们交互与更传统的应用程序有所不同。

本指南将涵盖您需要了解的所有内容，包括：

- [部署和交互智能合约](#部署和交互智能合约)
  - [建立本地区块链](#建立本地区块链)
  - [部署智能合约](#部署智能合约)
  - [通过控制台进行交互](#通过控制台进行交互)
    - [发送交易](#发送交易)
    - [查询状态](#查询状态)
  - [以编程方式交互](#以编程方式交互)
    - [设置](#设置)
    - [获取合约实例](#获取合约实例)
    - [调用合约](#调用合约)
    - [发送交易](#发送交易-1)
  - [下一步](#下一步)


> Note
Truffle和Hardhat都提供了说明。您可以使用此切换选择您偏好的工具！

## 建立本地区块链
在开始之前，我们首先需要一个可以部署我们合约的环境。以太坊区块链（通常称为“主网”）需要使用以太币（其本地货币）来支付真正的费用。这对测试项目非常不友好。

为了解决这个问题，存在一些“测试网”（用于“测试网络”），包括Sepolia和Goerli区块链。它们的工作方式与主网非常相似，但有一个区别：你可以免费获得这些网络的以太币，因此使用它们不会花费你一分钱。但是，你仍然需要处理私钥管理、5到20秒的区块时间以及实际获得这些免费以太币的问题。

在开发过程中，使用本地区块链是一个更好的选择。它在你的计算机上运行，不需要互联网访问，为你提供所需的所有以太币，并立即挖掘区块。这些原因也使本地区块链非常适合[自动化测试](../Writing%20automated%20tests/Writing%20automated%20smart%20contract%20tests-truffle.md)。

> TIP
如果您想学习如何在公共区块链上部署和使用合约，比如以太坊测试网，可以前往我们的[连接公共测试网络指南](../Connecting-to-public-test-networks/Connecting-to-public-test-networks-truffle.md)。

最受欢迎的本地区块链是[Ganache](https://github.com/trufflesuite/ganache-cli)。要在项目中安装它，请运行以下命令：
```
$ npm install --save-dev ganache-cli
```

启动时，Ganache将创建一组随机的解锁账户并给予它们以太币。为了获得本指南中将使用的相同地址，您可以以确定性模式启动Ganache：
```
$ npx ganache-cli --deterministic
```

Ganache将打印出可用账户及其私钥列表，以及一些区块链配置值。最重要的是，它将显示其地址，我们将用它来连接。默认情况下，这将是127.0.0.1:8545。

请记住，每次运行Ganache时，它都会创建一个全新的本地区块链 - 之前运行的状态不会被保留。这对于短期实验来说是可以接受的，但这意味着您需要在运行这些指南的整个过程中保持打开Ganache的窗口。或者，您可以使用--db选项运行Ganache，提供一个目录来在运行之间存储其数据。

> TIP
Truffle还有一个名为[Ganache](https://www.trufflesuite.com/ganache)的图形化版本，也称为ganache-cli。

> NOTE
您还可以在[开发模式](https://geth.ethereum.org/getting-started/dev-mode)下运行一个实际的以太坊节点。这些设置起来稍微复杂一些，对于测试和开发来说不太灵活，但更接近真实网络。

## 部署智能合约
在[开发智能合约指南](../Developing-smart-contracts/Developing-smart-contracts-truffle.md)中，我们设置了开发环境。

如果您还没有设置，请[创建](../Setting-up-a-Node-project/Setting-up-a-Node-project.md#创建项目)并[设置](../Developing-smart-contracts/Developing-smart-contracts-truffle.md#设置项目)项目，然后[创建](../Developing-smart-contracts/Developing-smart-contracts-truffle.md#第一份合约)并[编译](../Developing-smart-contracts/Developing-smart-contracts-truffle.md#solidity的编译)我们的[Box](../Developing-smart-contracts/Developing-smart-contracts-truffle.md#第一份合约)智能合约。

完成项目设置后，我们现在可以部署合约了。我们将部署Box合约，该合约在[开发智能合约指南](../Developing-smart-contracts/Developing-smart-contracts-truffle.md#第一份合约)中有介绍。请确保在contracts/Box.sol中有[Box](../Developing-smart-contracts/Developing-smart-contracts-truffle.md#第一份合约)的副本。

Truffle使用迁移来部署合约。迁移由JavaScript文件和一个特殊的[Migrations](https://www.trufflesuite.com/docs/truffle/getting-started/running-migrations)合约组成，用于跟踪链上的迁移。

我们将创建一个JavaScript迁移来部署我们的Box合约。我们将把这个文件保存为migrations/2_deploy.js。
```
// migrations/2_deploy.js
const Box = artifacts.require('Box');

module.exports = async function (deployer) {
  await deployer.deploy(Box);
};
```

在部署之前，我们需要配置与ganache的连接。我们需要为本地主机和端口8545添加一个开发网络，这是我们本地区块链使用的端口。
```
// truffle-config.js
module.exports = {
...
  networks: {
...
    development: {
     host: "127.0.0.1",     // Localhost (default: none)
     port: 8545,            // Standard Ethereum port (default: none)
     network_id: "*",       // Any network (default: none)
    },
    ...
```

使用[migrate](https://www.trufflesuite.com/docs/truffle/reference/truffle-commands#migrate)命令，我们可以将Box合约部署到开发网络（_Ganache_）:
```
npx truffle migrate --network development

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      1619762548805
> Block gas limit: 6721975 (0x6691b7)


1_initial_migration.js
======================

   Deploying 'Migrations'
   ----------------------
...

2_deploy.js
===========

   Deploying 'Box'
   ---------------
   > transaction hash:    0x25b0a326bfc9aa64be13efb5a4fb3f784ffa845c36d049547eeb0f78e0a3108d
   > Blocks: 0            Seconds: 0
   > contract address:    0xCfEB869F69431e42cdB54A4F4f105C19C080A601
...
```

> NOTE
Truffle会跟踪您部署的合约，并在部署时显示它们的地址（在我们的示例中为0xCfEB869F69431e42cdB54A4F4f105C19C080A601）。这些值在以编程方式与它们交互时非常有用。

一切都完成了！在真实的网络上，这个过程可能需要几秒钟，但在本地区块链上几乎是瞬间完成的。

> TIP
如果出现连接错误，请确保在另一个终端中运行[本地区块链](#建立本地区块链)。

> NOTE
请记住，本地区块链不会在多次运行中保留其状态！如果关闭本地区块链进程，您将需要重新部署您的合约。



## 通过控制台进行交互
通过[部署](#部署智能合约)了的Box合约，我们可以立即开始使用它。

我们将使用[Truffle控制台](https://www.trufflesuite.com/docs/truffle/getting-started/using-truffle-develop-and-the-console)与我们在本地开发网络上部署的Box合约进行交互。
```
npx truffle console --network development
truffle(development)> const box = await Box.deployed();
undefined
```

### 发送交易
Box的第一个函数store接收一个整数值并将其存储在合约存储中。由于此函数修改了区块链状态，因此我们需要发送一个交易到合约来执行它。

我们将发送一个交易来调用store函数，并传递一个数字值：
```
truffle(development)> await box.store(42)
{ tx:
   '0x5d4cc78f5d5eac3650214740728192ac760978e261962736289b10da0ec0ea43',
...
       event: 'ValueChanged',
       args: [Result] } ] }
```

注意交易收据还显示了Box发出了一个ValueChanged事件。

### 查询状态
Box的另一个功能被称为retrieve，它返回存储在合约中的整数值。这是对区块链状态的查询，因此我们不需要发送交易：
```
truffle(development)> await box.retrieve()
<BN: 2a>
```

由于查询仅读取状态而不发送交易，因此没有交易哈希可报告。这也意味着使用查询不需要花费任何以太币，并且可以免费在任何网络上使用。

> NOTE
我们的Box合约返回的是uint256，这个数字对于JavaScript来说太大了，所以我们会得到一个大数对象。我们可以使用(await box.retrieve()).toString()将大数显示为字符串。

```
truffle(development)> (await box.retrieve()).toString()
'42'
```

> TIP
要了解更多关于使用控制台的信息，请查阅[Truffle文档](https://www.trufflesuite.com/docs/truffle/getting-started/using-truffle-develop-and-the-console)。

## 以编程方式交互
控制台对于原型设计和运行一次性查询或事务非常有用。然而，最终您会想要从自己的代码中与您的合约进行交互。

在本节中，我们将看到如何从JavaScript中与我们的合约进行交互，并使用[Truffle根据我们的Truffle配置](https://www.trufflesuite.com/docs/truffle/getting-started/writing-external-scripts)执行我们的脚本。

> TIP
请记住，还有许多其他JavaScript库可用，您可以使用您最喜欢的任何一个。一旦合约部署，您可以通过任何库与其交互！

### 设置
让我们在一个新的scripts/index.js文件中开始编写我们的JavaScript代码，从一些样板开始，包括[编写异步代码](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)的样板。
```
// scripts/index.js
module.exports = async function main (callback) {
  try {
    // Our code will go here

    callback(0);
  } catch (error) {
    console.error(error);
    callback(1);
  }
};
```

我们可以通过向本地节点询问一些问题来测试我们的设置，比如获取启用的账户列表：
```
// Retrieve accounts from the local node
const accounts = await web3.eth.getAccounts();
console.log(accounts)
```

> NOTE
我们不会在每个代码片段中重复样板代码，但请确保始终在上面定义的main函数中编写代码！

使用truffle exec运行上面的代码，并检查是否得到了可用账户列表的响应。
```
npx truffle exec --network development ./scripts/index.js
Using network 'development'.

[ '0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1',
  '0xFFcf8FDEE72ac11b5c542428B35EEF5769C409f0',
...
]
```

这些账户应该与您在之前启动[本地区块链](#建立本地区块链)时显示的账户匹配。现在我们已经有了第一个从区块链中获取数据的代码片段，让我们开始使用我们的合约。请记住，我们将代码添加到上面定义的main函数中。

### 获取合约实例
为了与我们部署Box合约进行交互，我们将使用Truffle合约抽象，这是一个表示我们在区块链上的合约的JavaScript对象。
```
// 设置一个Truffle合约，表示我们部署的Box实例
const Box = artifacts.require('Box');
const box = await Box.deployed();
```

现在我们可以使用这个JavaScript对象与我们的合约进行交互。

### 调用合约
让我们首先显示Box合约的当前值。

我们需要调用合约的retrieve()公共方法，并[等待](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)响应：
```
// 调用已部署的Box合约的retrieve()函数
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

这段代码等同于我们之前在控制台中运行的[查询](#查询状态)。现在，通过再次运行脚本并检查打印出的值，确保一切正常运行：
```
npx truffle exec --network development ./scripts/index.js
Using network 'development'.

Box value is 42
```

> WARNING
如果您在任何时候重新启动了本地区块链，此脚本可能会失败。重新启动会清除所有本地区块链状态，因此Box合约实例将不会在预期的地址上。
如果发生这种情况，只需启动[本地区块链](#建立本地区块链)并重新[部署Box合约](#部署智能合约)即可。

### 发送交易
现在，我们将发送一笔交易来在我们的Box中存储一个新值。

让我们将一个值23存储在我们的Box中，然后使用我们之前编写的代码来显示更新后的值：
```
// 发送一个交易来在Box中存储一个新值
await box.store(23);

// 调用部署的Box合约的retrieve()函数
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

> NOTE
在实际应用中，您可能希望估计您的[交易的gas](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#methods-mymethod-estimategas)，并检查[gas价格预言机](https://ethgasstation.info/)以了解每个交易使用的最佳值。

现在我们可以运行这个代码段，并检查框的值是否已更新！
```
npx truffle exec --network development ./scripts/index.js
Using network 'development'.

Box value is 23
```

## 下一步
现在您已经知道如何设置本地区块链、部署合约并手动和编程地与它们交互，您需要学习有关测试环境、公共测试网络和进入生产的知识：
* [编写自动化测试](../Writing-automated-tests/Writing-automated-tests-truffle.md)

* [连接到公共测试网络](../Connecting-to-public-test-networks/Connecting-to-public-test-networks-truffle.md)

* [为主网做准备](../Preparing-for-mainnet/Preparing-for-mainnet.md)