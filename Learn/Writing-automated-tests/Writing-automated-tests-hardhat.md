# Writing automated smart contract tests
在区块链环境中，一个错误可能会让你失去所有资金，或者更糟糕的是，让你的用户失去资金！本指南将帮助你编写自动化测试，验证你的应用程序的行为与你预期的完全一致，从而开发出健壮的应用程序。

我们将涵盖以下主题：
- [Writing automated smart contract tests](#writing-automated-smart-contract-tests)
  - [关于测试](#关于测试)
  - [设置测试环境](#设置测试环境)
  - [编写单元测试](#编写单元测试)
  - [执行复杂的断言](#执行复杂的断言)
  - [下一步](#下一步)

> NOTE
*Truffle*和*Hardhat*都提供了说明。 使用此切换选择你的首选项！

## 关于测试
在测试技术方面，有各种各样的技术，从[简单的手动验证](../Deploying-and-interacting/Deploying-and-interacting-hardat.md#从控制台交互)到复杂的端到端设置，它们各自都有用处。

然而，实践证明，在智能合约开发中，[合约单元测试](https://en.wikipedia.org/wiki/Unit_testing)非常值得。这些测试易于编写和快速运行，让你有信心添加功能并修复代码中的错误。

智能合约单元测试由多个小型、专注的测试组成，每个测试都检查合约的一个小部分是否正确。它们通常可以用单个句子来表达规范，例如“管理员能够暂停合约”、“转移代币会发出事件”或“非管理员不能铸造新代币”。

## 设置测试环境
你可能会想知道我们将如何运行这些测试，因为智能合约是在区块链内执行的。使用实际的以太坊网络将非常昂贵，而测试网是免费的，但它们也很慢（区块时间在5到20秒之间）。如果我们打算在每次更改代码时运行数百个测试，我们需要更好的解决方案。

我们将使用称为本地区块链的东西：这是一个精简版的真实区块链，与互联网断开连接，在你的计算机上运行。这将大大简化事情：你不需要获取Ether，新的区块将立即被挖掘出来。

## 编写单元测试
我们将使用[Chai](https://www.chaijs.com/)断言进行单元测试。
```
npm install --save-dev chai
```

我们将把测试文件保存在一个名为test的目录中。测试最好按照contracts[目录](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#第一份合约)的结构进行组织：对于其中的每个.sol文件，创建一个相应的测试文件。

现在是编写我们的第一组测试的时候了！这些测试将测试之前介绍过的Box[合约的属性](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#第一份合约)：一个简单的合约，允许你检索先前所有者存储的值。

在你的项目根目录中创建一个测试目录。我们将把测试保存为test/Box.test.js。每个测试.js文件通常包含单个合约的测试，并以其命名。
```
// test/Box.test.js
// 加载依赖项
const { expect } = require('chai');

// 开始测试块
describe('Box', function () {
  before(async function () {
    this.Box = await ethers.getContractFactory('Box');
  });

  beforeEach(async function () {
    this.box = await this.Box.deploy();
    await this.box.deployed();
  });

  // 测试用例
  it('retrieve returns a value previously stored', async function () {
    // 存储一个值
    await this.box.store(42);

    // 测试返回的值是否与之前存储的值相同
    // 注意，我们需要使用字符串来比较256位整数
    expect((await this.box.retrieve()).toString()).to.equal('42');
  });
});
```

> TIP
许多书籍都介绍了如何构建单元测试。请查看[Moloch测试指南](https://github.com/MolochVentures/moloch/tree/4e786db8a4aa3158287e0935dcbc7b1e43416e38/test#moloch-testing-guide)，其中包含一系列旨在测试Solidity智能合约的原则。

我们现在可以运行我们的测试了！
运行 npx truffle test 将执行测试目录中的所有测试，检查你的合约是否按照你的意图工作。

```
npx hardhat test


  Box
    ✓ retrieve returns a value previously stored


  1 passing (578ms)
```

此时，建议设置一个持续集成服务，如[CircleCI](https://circleci.com/)，以便在每次将代码提交到GitHub时自动运行测试。

## 执行复杂的断言
许多有趣的合约属性可能很难捕捉，例如：

* 验证合约在错误时是否回滚

* 测量账户的以太余额变化量

* 检查是否发出了正确的事件

[OpenZeppelin Test Helpers](../../Home/Test%20Helpers/Overview.md)是一个旨在帮助你测试所有这些属性的库。它还将简化模拟区块链上时间流逝和处理非常大的数字的任务。

> NOTE
OpenZeppelin Test Helpers 是基于 web3.js 的，因此 Hardhat 用户应该使用 [Truffle 插件](https://hardhat.org/guides/truffle-testing.html)以确保兼容性。然而，我们建议使用 [Hardhat Chai Matchers](https://hardhat.org/hardhat-chai-matchers/docs/overview) 作为 Ethers.js 的更好支持的替代方案。

要安装OpenZeppelin测试助手，请运行以下命令：
```
npm install --save-dev @openzeppelin/test-helpers
```

我们可以使用OpenZeppelin Test Helpers来更新我们的测试，以支持非常大的数字，检查事件是否被触发以及检查事务是否回滚。
```
// test/Box.test.js
// 加载依赖项
const { expect } = require('chai');

// 从Test Helpers导入实用程序
const { BN, expectEvent, expectRevert } = require('@openzeppelin/test-helpers');

// 加载编译后的合约
const Box = artifacts.require('Box');

// 开始测试块
contract('Box', function ([ owner, other ]) {
  // 使用大整数（'big numbers'）
  const value = new BN('42');

  beforeEach(async function () {
    this.box = await Box.new({ from: owner });
  });

  it('retrieve returns a value previously stored', async function () {
    await this.box.store(value, { from: owner });

    // 使用大整数比较
    expect(await this.box.retrieve()).to.be.bignumber.equal(value);
  });

  it('store emits an event', async function () {
    const receipt = await this.box.store(value, { from: owner });

    // 测试是否触发了一个带有新值的ValueChanged事件
    expectEvent(receipt, 'ValueChanged', { value: value });
  });

  it('non owner cannot store a value', async function () {
    // 测试事务是否回滚
    await expectRevert(
      this.box.store(value, { from: other }),
      'Ownable: caller is not the owner',
    );
  });
});
```

这些测试将测试之前指南中的Ownable Box[合约的属性](../Developing-smart-contracts/Developing-smart-contracts-hardh.md#导入openzeppelin-contracts)：一个简单的合约，让你检索所有者先前存储的值。

再次运行测试以查看Test Helpers的作用：
```
npx truffle test
...
  Contract: Box
    ✓ retrieve returns a value previously stored
    ✓ store emits an event
    ✓ non owner cannot store a value (588ms)


  3 passing (753ms)
```

测试助手将让你编写强大的断言，而无需担心底层的以太坊库的低级细节。要了解更多关于它们的用途，可以前往它们的[API参考文档](https://docs.openzeppelin.com/test-helpers/0.5/api)。

## 下一步
一旦你已经彻底测试了你的合约并且相信它们的正确性，你将想要部署它们到一个真实的网络并开始与它们交互。以下指南将帮助你了解这些主题：
* [连接公共测试网络](../Connecting-to-public-test-networks/Connecting-to-public-test-networks-hardhat.md)

* [部署和交互](../Deploying-and-interacting/Deploying-and-interacting-hardat.md)

* [准备主网](../Preparing-for-mainnet/Preparing-for-mainnet.md)