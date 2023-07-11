# 部署和交互智能合约
与大多数软件不同，智能合约不在您的计算机或某个服务器上运行：它们运行在以太坊网络本身上。这意味着与它们交互与更传统的应用程序有所不同。

本指南将涵盖您需要了解的所有内容，包括：

* 设置本地区块链

* 部署智能合约

* 从控制台交互

* 编程交互

>Note
Truffle和Hardhat都提供了说明。您可以使用此切换选择您偏好的工具！

## 建立本地区块链
在开始之前，我们首先需要一个可以部署我们合约的环境。以太坊区块链（通常称为“主网”）需要使用以太币（其本地货币）来支付真正的费用。这对测试项目非常不友好。

为了解决这个问题，存在一些“测试网”（用于“测试网络”），包括Sepolia和Goerli区块链。它们的工作方式与主网非常相似，但有一个区别：你可以免费获得这些网络的以太币，因此使用它们不会花费你一分钱。但是，你仍然需要处理私钥管理、5到20秒的区块时间以及实际获得这些免费以太币的问题。

在开发过程中，使用本地区块链是一个更好的选择。它在你的计算机上运行，不需要互联网访问，为你提供所需的所有以太币，并立即挖掘区块。这些原因也使本地区块链非常适合*自动化测试*。

>TIP
如果您想学习如何在公共区块链上部署和使用合约，比如以太坊测试网，可以前往我们的*连接公共测试网络指南*。

最受欢迎的本地区块链是 Ganache。要在您的项目中安装它，请运行：
```
npm install --save-dev ganache-cli
```
启动Ganache时，它会创建一组随机的未锁定账户并给它们以太币。为了得到本指南中将使用的相同地址，您可以在确定性模式下启动Ganache：
```
npx ganache-cli --deterministic
```
Ganache会打印出可用账户及其私钥列表，以及一些区块链配置值。最重要的是，它会显示其地址，我们将使用该地址连接到它。默认情况下，这将是127.0.0.1:8545。

请记住，每次运行Ganache，它都会创建一个全新的本地区块链 - 以前运行的状态不会被保留。这对于短期实验来说是可以的，但这意味着您需要在这些指南的持续时间内打开运行Ganache的窗口。或者，您可以使用--db选项运行Ganache，提供一个目录以在运行之间存储其数据。

>TIP
Truffle有一个图形化版本的ganache-cli，也称为[Ganache](https://www.trufflesuite.com/ganache)。

>NOTE
你也可以在开发模式下运行一个真正的[以太坊节点](https://geth.ethereum.org/getting-started/dev-mode)。这些设置比较复杂，并且不如测试和开发灵活，但更能代表真实网络。

## 部署智能合约
在*“开发智能合约”指南*中，我们设置了开发环境。

如果您还没有设置，请*创建*并*设置*项目，然后*创建*并*编译*我们的Box智能合约。

完成项目设置后，我们现在准备部署合约。我们将部署来自*“开发智能合约”指南*的Box。请确保您在contracts/Box.sol中有*Box*的副本。

Truffle使用[迁移](https://www.trufflesuite.com/docs/truffle/getting-started/running-migrations)来部署合约。迁移由JavaScript文件和一个特殊的迁移合约组成，用于跟踪链上的迁移。

我们将创建一个JavaScript迁移来部署我们的Box合约。我们将把这个文件保存为migrations/2_deploy.js。
```
// migrations/2_deploy.js
const Box = artifacts.require('Box');

module.exports = async function (deployer) {
  await deployer.deploy(Box);
};
```
在部署之前，我们需要配置与 ganache 的连接。我们需要添加一个开发网络，用于本地主机和端口 8545，这是我们本地区块链正在使用的端口。
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

使用[迁移](https://www.trufflesuite.com/docs/truffle/reference/truffle-commands#migrate)命令，我们可以将Box合约部署到开发网络（*Ganache*）。
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

>NOTE
Truffle将跟踪您部署的合约，但在部署时它还会显示它们的地址（在我们的示例中为0xCfEB869F69431e42cdB54A4F4f105C19C080A601）。这些值在以编程方式与它们交互时将非常有用。

完成了！在真实的网络上，这个过程可能需要几秒钟，但在本地区块链上几乎是瞬间完成的。

>TIP
如果您遇到连接错误，请确保在另一个终端中运行了*本地区块链*。

>NOTE
请记住，本地区块链在多次运行中**不会保持**其状态！如果您关闭本地区块链进程，则必须重新部署合约。

## 从控制台交互
有了我们*部署*的Box合约，我们可以立即开始使用它。

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
请注意，交易收据还显示了Box发出了ValueChanged事件。

### 查询状态
Box的另一个功能被称为retrieve，它返回存储在合约中的整数值。这是对区块链状态的查询，因此我们不需要发送交易：
```
truffle(development)> await box.retrieve()
<BN: 2a>
```
由于查询仅读取状态而不发送交易，因此没有交易哈希可报告。这也意味着使用查询不需要花费任何以太币，并且可以免费在任何网络上使用。

>NOTE
我们的Box合约返回的是uint256，这个数字对于JavaScript来说太大了，所以我们会得到一个大数对象。我们可以使用(await box.retrieve()).toString()将大数显示为字符串。

```
truffle(development)> (await box.retrieve()).toString()
'42'
```

>TIP
要了解更多关于如何使用控制台的信息，请查阅[Truffle文档](https://www.trufflesuite.com/docs/truffle/getting-started/using-truffle-develop-and-the-console)。

## 以编程方式交互
控制台对于原型设计和运行一次性查询或事务非常有用。然而，最终您会想要从自己的代码中与您的合约进行交互。

在本节中，我们将看到如何从 JavaScript 与我们的合约进行交互，并使用 Truffle 在我们的 [Truffle 配置中执行我们的脚本](https://www.trufflesuite.com/docs/truffle/getting-started/writing-external-scripts)。

>TIP
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
我们可以通过询问本地节点一些内容，例如启用的账户列表来测试我们的设置：
```
// Retrieve accounts from the local node
const accounts = await web3.eth.getAccounts();
console.log(accounts)
```

>NOTE
我们不会在每个代码片段中重复使用样板代码，但请确保始终在我们上面定义的主函数中编写代码！

使用truffle exec运行上述代码，并检查是否在响应中获得了可用账户列表。
```
npx truffle exec --network development ./scripts/index.js
Using network 'development'.

[ '0x90F8bf6A479f320ead074411a4B0e7944Ea8c9C1',
  '0xFFcf8FDEE72ac11b5c542428B35EEF5769C409f0',
...
]
```
这些账户应该与您之前启动*本地区块链*时显示的账户匹配。现在我们有了第一个代码片段来从区块链中获取数据，让我们开始处理我们的合约。请记住，我们将在上面定义的主函数中添加我们的代码。

### 获取合约实例
为了与我们部署的*Box*合约进行交互，我们将使用Truffle合约抽象，这是一个表示我们在区块链上的合约的JavaScript对象。
```
// Set up a Truffle contract, representing our deployed Box instance
const Box = artifacts.require('Box');
const box = await Box.deployed();
```
我们现在可以使用这个 JavaScript 对象与我们的合约进行交互。

### 呼叫合约

让我们从显示Box合约的当前值开始。

我们需要调用合约的retrieve()公共方法，并[等待](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)响应：

```
// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```
这个片段等同于我们之前在控制台中运行的查询。现在，通过再次运行脚本并检查打印出的值，确保一切顺利运行：
```
npx truffle exec --network development ./scripts/index.js
Using network 'development'.

Box value is 42
```

>WARNING
如果您在任何时候重新启动了本地区块链，这个脚本可能会失败。重新启动会清除所有本地区块链状态，因此Box合约实例将不会在预期地址上。
如果发生这种情况，只需*启动本地区块链*并*重新部署*Box合约即可。

### 发送交易
现在，我们将发送一笔交易来在我们的Box中存储一个新值。

让我们将一个值23存储在我们的Box中，然后使用我们之前编写的代码来显示更新后的值：
```
// Send a transaction to store() a new value in the Box
await box.store(23);

// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```
>NOTE
在实际应用中，您可能希望估计您的[交易的gas](https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#methods-mymethod-estimategas)，并检查[gas价格预言机](https://ethgasstation.info/)以了解每个交易使用的最佳值。

现在我们可以运行这个代码段，并检查框的值是否已更新！
```
npx truffle exec --network development ./scripts/index.js
Using network 'development'.

Box value is 23
```

## 下一步
现在您已经知道如何设置本地区块链、部署合约并手动和编程地与它们交互，您需要学习有关测试环境、公共测试网络和进入生产的知识：
* *编写自动化测试*

* *连接到公共测试网络*

* *为主网做准备*