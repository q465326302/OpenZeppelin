# 部署和交互智能合约
与大多数软件不同，智能合约不在你的计算机或某个服务器上运行：它们运行在以太坊网络本身上。这意味着与它们交互与更传统的应用程序有所不同。

本指南将涵盖你需要了解的所有内容，包括：

- [部署和交互智能合约](#部署和交互智能合约)
  - [建立本地区块链](#建立本地区块链)
  - [部署智能合约](#部署智能合约)
  - [从控制台交互](#从控制台交互)
    - [发送交易](#发送交易)
    - [查询状态](#查询状态)
  - [以编程方式交互](#以编程方式交互)
    - [设置](#设置)
    - [获取合约实例](#获取合约实例)
    - [调用合约](#调用合约)
    - [发送交易](#发送交易-1)
  - [下一步](#下一步)

> Note
*Truffle*和*Hardhat*都提供了说明。你可以使用此切换选择你偏好的工具！

## 建立本地区块链
在开始之前，我们首先需要一个可以部署我们合约的环境。以太坊区块链（通常称为“主网”）需要使用以太（其本地货币）来支付真正的费用。这对测试项目非常不友好。

为了解决这个问题，存在一些“测试网”（用于“测试网络”），包括Sepolia和Goerli区块链。它们的工作方式与主网非常相似，但有一个区别：你可以免费获得这些网络的以太，因此使用它们不会花费你一分钱。但是，你仍然需要处理私钥管理、5到20秒的区块时间以及实际获得这些免费以太的问题。

在开发过程中，使用本地区块链是一个更好的选择。它在你的计算机上运行，不需要互联网访问，为你提供所需的所有以太，并立即挖掘区块。这些原因也使本地区块链非常适合[自动化测试](../Writing-automated-tests/Writing-automated-tests-hardhat.md#编写智能合约的自动化测试)。

> TIP
如果你想学习如何在公共区块链上部署和使用合约，比如以太坊测试网，可以前往我们的[连接公共测试网络指南](../Connecting-to-public-test-networks/Connecting-to-public-test-networks-hardhat.md)。

Hardhat内置了一个本地区块链，即[Hardhat Network](https://hardhat.org/hardhat-network/)。

启动时，Hardhat Network将创建一组解锁的账户并给它们提供以太。
```
npx hardhat node
```

启动Ganache时，它会创建一组随机的未锁定账户并给它们以太。为了得到本指南中将使用的相同地址，你可以在确定性模式下启动Ganache：
```
npx ganache-cli --deterministic
```

Hardhat Network将打印出其地址，http://127.0.0.1:8545，以及可用账户和它们的私钥列表。

请记住，每次运行Hardhat Network时，它都会创建一个全新的本地区块链 - 之前运行的状态不会被保留。这对于短期实验来说是可以接受的，但这意味着你需要打开一个窗口来运行Hardhat Network，以便在这些指南的持续时间内保持运行状态。

> TIP
当没有指定网络且没有配置默认网络或默认网络设置为hardhat时，Hardhat将始终启动一个Hardhat Network实例。

> NOTE
你也可以在开发模式下运行一个实际的[以太坊节点](https://geth.ethereum.org/getting-started/dev-mode)。这些设置起来稍微复杂一些，对于测试和开发来说不太灵活，但更接近真实网络。

## 部署智能合约
在[“开发智能合约”指南](../Developing-smart-contracts/Developing-smart-contracts-hardh.md)中，我们设置了开发环境。

如果你还没有设置，请[创建](../Setting-up-a-Node-project/Setting-up-a-Node-project.md#创建项目)并[设置](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#设置项目)项目，然后[创建](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#第一份合约)并[编译](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#solidity的编译)我们的[Box](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#第一份合约)智能合约。

完成项目设置后，我们现在准备部署合约。我们将部署来自[“开发智能合约”指南](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#第一份合约)的Box。请确保你在contracts/Box.sol中有[Box](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#第一份合约)的副本。

Hardhat目前没有原生的部署系统，而是使用[脚本](https://hardhat.org/guides/deploying.html)来部署合约。

我们将创建一个脚本来部署我们的Box合约。我们将把这个文件保存为scripts/deploy.js。
```
// scripts/deploy.js
async function main () {
  // We get the contract to deploy
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await Box.deploy();
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

我们在脚本中使用了[ethers](https://github.com/ethers-io/ethers.js)，所以我们需要安装它和[@nomiclabs/hardhat-ethers](https://hardhat.org/plugins/nomiclabs-hardhat-ethers.html)插件。

```
npm install --save-dev @nomiclabs/hardhat-ethers ethers
```

我们需要在[配置](https://hardhat.org/config/)中添加我们正在使用@nomiclabs/hardhat-ethers插件。
```
// hardhat.config.js
require('@nomiclabs/hardhat-ethers');

...
module.exports = {
...
};
```

使用run命令，我们可以将Box合约部署到本地网络（[Hardhat Network](../Deploying-and-interacting/Deploying-and-interacting-hardat.md#建立本地区块链)）：
```
npx hardhat run --network localhost scripts/deploy.js
Deploying Box...
Box deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```
> NOTE
Hardhat不会跟踪你部署的合约。我们在脚本中显示了部署的地址（在我们的示例中为0x5FbDB2315678afecb367f032d93F642f64180aa3）。这在以编程方式与它们交互时会很有用。

全部完成！在真实网络上，这个过程可能需要几秒钟，但在本地区块链上几乎是瞬间完成的。

> TIP
如果你遇到连接错误，请确保在另一个终端中运行[本地区块链](../Deploying-and-interacting/Deploying-and-interacting-hardat.md#建立本地区块链)。

> NOTE
请记住，本地区块链不会在多次运行中保持其状态！如果关闭本地区块链进程，你将需要重新部署合约。

## 从控制台交互
通过[部署](../Deploying-and-interacting/Deploying-and-interacting-hardat.md)了的Box合约，我们可以立即开始使用它。

我们将使用[Hardhat控制台](https://hardhat.org/guides/hardhat-console.html)与部署在本地主机网络上的Box合约进行交互。

> NOTE
我们需要指定我们在部署脚本中显示的Box合约的地址。

> NOTE
重要的是要明确设置Hardhat连接到的网络，以便将我们的控制台会话连接到该网络。如果不这样做，Hardhat将默认使用一个新的临时网络，而我们的Box合约将不会被部署到该网络上。
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x5FbDB2315678afecb367f032d93F642f64180aa3')
undefined
```

### 发送交易
Box的第一个函数store接收一个整数值并将其存储在合约存储中。因为这个函数修改了区块链状态，所以我们需要发送一个交易给合约来执行它。

我们将发送一个交易来调用store函数并传入一个数值：

```
> await box.store(42)
{
  hash: '0x3d86c5c2c8a9f31bedb5859efa22d2d39a5ea049255628727207bc2856cce0d3',
...
```

### 查询状态
Box的另一个函数叫做retrieve，它返回合约中存储的整数值。这是一个查询区块链状态的操作，所以我们不需要发送交易：
```
> await box.retrieve()
BigNumber { _hex: '0x2a', _isBigNumber: true }
```

因为查询只读取状态而不发送交易，所以没有交易哈希可供报告。这也意味着使用查询不需要花费任何以太，并且可以在任何网络上免费使用。

> NOTE
我们的Box合约返回的是uint256，这对于JavaScript来说是一个太大的数字，所以我们得到的是一个大数对象。我们可以使用(await box.retrieve()).toString()将大数显示为字符串。

```
> (await box.retrieve()).toString()
'42'
```

> TIP
要了解有关使用控制台的更多信息，请查看[Hardhat文档](https://hardhat.org/guides/hardhat-console.html)。

## 以编程方式交互
控制台对于原型设计和运行一次性查询或事务非常有用。然而，最终你会想要从自己的代码中与你的合约进行交互。

在本节中，我们将看到如何从JavaScript中与合约进行交互，并使用[Hardhat脚本](https://hardhat.org/guides/scripts.html)在我们的Hardhat配置中运行我们的合约。

> TIP
请记住，还有许多其他JavaScript库可用，你可以使用你最喜欢的任何一个。一旦合约部署，你可以通过任何库与其交互！

### 设置
让我们从一个新的scripts/index.js文件中开始编码，我们将在其中编写我们的JavaScript代码，首先包括一些样板代码，包括用于编写[异步代码的样板代码](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)。

```
// scripts/index.js
async function main () {
  // Our code will go here
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

我们可以通过询问本地节点一些问题来测试我们的设置，比如启用的账户列表：
```
// Retrieve accounts from the local node
const accounts = await ethers.provider.listAccounts();
console.log(accounts);
```

> NOTE
我们不会在每个片段中重复样板代码，但请确保始终在我们上面定义的main函数内部编写代码！

使用hardhat run运行上面的代码，并检查是否得到了可用账户列表的响应。
```
npx hardhat run --network localhost ./scripts/index.js
[
  '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  '0x70997970C51812dc3A010C7d01b50e0d17dc79C8',
...
]
```

这些账户应该与你在之前启动[本地区块链](../Deploying-and-interacting/Deploying-and-interacting-hardat.md)时显示的账户相匹配。现在我们已经有了第一个从区块链中获取数据的代码片段，让我们开始使用我们的合约。记住，我们将把代码添加到上面定义的main函数中。

### 获取合约实例
为了与我们部署的Box合约进行交互，我们将使用一个[ethers合约实例](https://docs.ethers.io/v5/api/contract/contract/)。

ethers合约实例是一个表示我们在区块链上的合约的JavaScript对象，我们可以使用它来与我们的合约进行交互。要将其附加到我们部署的合约，我们需要提供合约地址。

```
// Set up an ethers contract, representing our deployed Box instance
const address = '0x5FbDB2315678afecb367f032d93F642f64180aa3';
const Box = await ethers.getContractFactory('Box');
const box = await Box.attach(address);
```

> NOTE
请确保用你在部署合约时获得的地址替换上面显示的地址，这可能与上面显示的地址不同。

现在我们可以使用这个JavaScript对象与我们的合约进行交互。

### 调用合约

让我们首先显示Box合约的当前值。

我们需要调用合约的只读retrieve()公共方法，并[等待](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)响应：

```
// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

这个代码片段等同于我们之前从控制台运行的[查询](../Deploying-and-interacting/Deploying-and-interacting-hardat.md#查询状态)。现在，通过再次运行脚本并检查打印的值，确保一切正常：

```
npx hardhat run --network localhost ./scripts/index.js
Box value is 42
```

> WARNING
如果你在任何时候重新启动了本地区块链，此脚本可能会失败。重新启动会清除所有本地区块链状态，因此Box合约实例将不会在预期的地址上。
如果发生这种情况，只需启动[本地区块链](../Deploying-and-interacting/Deploying-and-interacting-hardat.md#建立本地区块链)并[重新部署](../Deploying-and-interacting/Deploying-and-interacting-hardat.md#部署智能合约)Box合约即可。

### 发送交易
现在，我们将发送一个交易来在我们的Box中存储一个新值。

让我们将一个值23存储在我们的Box中，然后使用我们之前编写的代码来显示更新后的值：

```
// Send a transaction to store() a new value in the Box
await box.store(23);

// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

> NOTE
在实际应用中，你可能希望[估算你的交易的gas](https://docs.ethers.io/v5/api/contract/contract/#contract-estimateGas)，并检查[gas价格预言机](https://ethgasstation.info/)以了解每个交易的最佳值。

现在，我们可以运行这个代码片段，并检查box的值是否更新！

```
npx hardhat run --network localhost ./scripts/index.js
Box value is 23
```

## 下一步
现在你已经知道如何设置本地区块链、部署合约并手动和编程地与它们交互，你需要学习有关测试环境、公共测试网络和进入生产的知识：
* [编写自动化测试](../Writing-automated-tests/Writing-automated-tests-hardhat.md)

* [连接到公共测试网络](../Connecting-to-public-test-networks/Connecting-to-public-test-networks-hardhat.md)

* [为主网做准备](../Preparing-for-mainnet/Preparing-for-mainnet.md)