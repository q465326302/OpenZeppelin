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

Hardhat自带一个本地区块链，称为Hardhat Network。
启动时，Hardhat Network将创建一组未锁定的账户并赋予它们以太币。
```
npx hardhat node
```
Hardhat Network将打印出其地址http://127.0.0.1:8545，以及可用帐户及其私钥的列表。

请注意，每次运行Hardhat Network时，都会创建一个全新的本地区块链 - 不保留以前运行的状态。这对于短期实验是可以接受的，但这意味着您需要在运行这些指南的过程中保持打开运行Hardhat Network的窗口。

>TIP
当没有指定网络且没有配置默认网络，或默认网络设置为hardhat时，Hardhat将始终启动Hardhat Network的一个实例。

>NOTE
你也可以在开发模式下运行一个真正的[以太坊节点](https://geth.ethereum.org/getting-started/dev-mode)。这些设置比较复杂，并且不如测试和开发灵活，但更能代表真实网络。

## 部署智能合约
在*“开发智能合约”指南*中，我们设置了开发环境。

如果您还没有设置，请*创建*并*设置*项目，然后*创建*并*编译*我们的Box智能合约。

完成项目设置后，我们现在准备部署合约。我们将部署来自*“开发智能合约”指南*的Box。请确保您在contracts/Box.sol中有*Box*的副本。

Hardhat目前没有本地部署系统，我们使用[脚本](https://hardhat.org/guides/deploying.html)来部署合约。

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
我们在脚本中使用ethers，因此需要安装@nomiclabs/hardhat-ethers插件。
```
npm install --save-dev @nomiclabs/hardhat-ethers ethers
```
我们需要在配置中添加我们正在使用@nomiclabs/hardhat-ethers插件。
```
// hardhat.config.js
require('@nomiclabs/hardhat-ethers');

...
module.exports = {
...
};
```
使用运行命令，我们可以将 Box 合约部署到本地网络（Hardhat Network）：
```
npx hardhat run --network localhost scripts/deploy.js
Deploying Box...
Box deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

>NOTE
Hardhat不会跟踪您部署的合约。我们在脚本中显示了已部署的地址（在我们的示例中为0x5FbDB2315678afecb367f032d93F642f64180aa3）。这在以编程方式与它们交互时将很有用。

完成了！在真实的网络上，这个过程可能需要几秒钟，但在本地区块链上几乎是瞬间完成的。

>TIP
如果您遇到连接错误，请确保在另一个终端中运行了*本地区块链*。

>NOTE
请记住，本地区块链在多次运行中**不会保持**其状态！如果您关闭本地区块链进程，则必须重新部署合约。

## 从控制台交互
有了我们*部署*的Box合约，我们可以立即开始使用它。

我们将使用[Hardhat控制台](https://hardhat.org/guides/hardhat-console.html)与我们在localhost网络上部署的Box合约进行交互。
>NOTE
我们需要指定我们在部署脚本中显示的Box合约的地址。

>NOTE
重要的是我们明确设置Hardhat连接到的网络，以便将我们的控制台会话连接到正确的网络。如果我们不这样做，Hardhat将默认使用一个新的临时网络，我们的Box合约将无法部署到该网络上。
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
Box的第一个函数store接收一个整数值并将其存储在合约存储中。由于此函数修改了区块链状态，因此我们需要发送一个交易到合约来执行它。

我们将发送一个交易来调用store函数，并传递一个数字值：
```
> await box.store(42)
{
  hash: '0x3d86c5c2c8a9f31bedb5859efa22d2d39a5ea049255628727207bc2856cce0d3',
...
```

### 查询状态
Box的另一个功能被称为retrieve，它返回存储在合约中的整数值。这是对区块链状态的查询，因此我们不需要发送交易：
```
> await box.retrieve()
BigNumber { _hex: '0x2a', _isBigNumber: true }
```
由于查询仅读取状态而不发送交易，因此没有交易哈希可报告。这也意味着使用查询不需要花费任何以太币，并且可以免费在任何网络上使用。

>NOTE
我们的Box合约返回的是uint256，这个数字对于JavaScript来说太大了，所以我们会得到一个大数对象。我们可以使用(await box.retrieve()).toString()将大数显示为字符串。

```
> (await box.retrieve()).toString()
'42'
```

>TIP
要了解更多关于使用控制台的信息，请查看[Hardhat文档](https://hardhat.org/guides/hardhat-console.html)。

## 以编程方式交互
控制台对于原型设计和运行一次性查询或事务非常有用。然而，最终您会想要从自己的代码中与您的合约进行交互。

在本节中，我们将学习如何从JavaScript与我们的合约进行交互，并使用Hardhat在我们的[Hardhat配置中运行我们的脚本](https://hardhat.org/guides/scripts.html)。

>TIP
请记住，还有许多其他JavaScript库可用，您可以使用您最喜欢的任何一个。一旦合约部署，您可以通过任何库与其交互！

### 设置
让我们在一个新的scripts/index.js文件中开始编写我们的JavaScript代码，从一些样板开始，包括[编写异步代码](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)的样板。

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
我们可以通过询问本地节点一些内容，例如启用的账户列表来测试我们的设置：
```
// Retrieve accounts from the local node
const accounts = await ethers.provider.listAccounts();
console.log(accounts);
```

>NOTE
我们不会在每个代码片段中重复使用样板代码，但请确保始终在我们上面定义的主函数中编写代码！

使用hardhat run运行上面的代码，并检查是否返回可用账户列表。
```
npx hardhat run --network localhost ./scripts/index.js
[
  '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  '0x70997970C51812dc3A010C7d01b50e0d17dc79C8',
...
]
```
这些账户应该与您之前启动*本地区块链*时显示的账户匹配。现在我们有了第一个代码片段来从区块链中获取数据，让我们开始处理我们的合约。请记住，我们将在上面定义的主函数中添加我们的代码。

### 获取合约实例
为了与我们部署的*Box*合约进行交互，我们将使用一个[以太坊合约实例](https://docs.ethers.io/v5/api/contract/contract/)。

以太坊合约实例是一个JavaScript对象，它代表了我们在区块链上的合约，我们可以使用它来与我们的合约进行交互。要将它附加到我们部署的合约上，我们需要提供合约地址。
```
// Set up an ethers contract, representing our deployed Box instance
const address = '0x5FbDB2315678afecb367f032d93F642f64180aa3';
const Box = await ethers.getContractFactory('Box');
const box = await Box.attach(address);
```
>NOTE
请确保将地址替换为部署合约时获得的地址，该地址可能与此处显示的地址不同。

现在，我们可以使用这个JavaScript对象与我们的合约进行交互。

### 呼叫合约

让我们从显示Box合约的当前值开始。

我们需要调用合约的retrieve()公共方法，并[等待](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)响应：

```
// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```
这个片段等同于我们之前在控制台中运行的*查询*。现在，通过再次运行脚本并检查打印出的值，确保一切顺利运行：
```
npx hardhat run --network localhost ./scripts/index.js
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
npx hardhat run --network localhost ./scripts/index.js
Box value is 23
```

## 下一步
现在您已经知道如何设置本地区块链、部署合约并手动和编程地与它们交互，您需要学习有关测试环境、公共测试网络和进入生产的知识：
* *编写自动化测试*

* *连接到公共测试网络*

* *为主网做准备*