# 编写智能合约的自动化测试
在区块链环境中，一个错误可能会让你失去所有资金，或者更糟糕的是，让你的用户失去资金！本指南将帮助你编写自动化测试，验证你的应用程序的行为与你预期的完全一致，从而开发出健壮的应用程序。

我们将涵盖以下主题：
* 建立测试环境

* 编写单元测试

* 执行复杂断言
>NOTE
Truffle和Hardhat都提供了说明。 使用此切换选择您的首选项！

## 关于测试
有许多测试技术，从*简单的手动验证*到复杂的端到端设置，它们各自都有用。

然而，在智能合约开发中，实践证明，[合约单元测试](https://en.wikipedia.org/wiki/Unit_testing)非常值得。这些测试易于编写和快速运行，让您有信心添加功能并修复代码中的错误。

智能合约单元测试由多个小型、专注的测试组成，每个测试都检查合约的一个小部分是否正确。它们通常可以用单个句子来表达规范，例如“管理员能够暂停合约”、“转移代币会发出事件”或“非管理员不能铸造新代币”。

## 设置测试环境
您可能会想知道我们将如何运行这些测试，因为智能合约是在区块链内执行的。使用实际的以太坊网络将非常昂贵，而测试网是免费的，但它们也很慢（区块时间在5到20秒之间）。如果我们打算在每次更改代码时运行数百个测试，我们需要更好的解决方案。

我们将使用称为本地区块链的东西：真实事物的简化版本，与互联网断开连接，在您的计算机上运行。这将大大简化事情：您不需要获取Ether，新块将立即被挖掘。

## 编写单元测试
我们将使用[Chai](https://www.chaijs.com/)断言进行单元测试。
```
npm install --save-dev chai
```
我们将把测试文件保存在一个名为test的目录中。测试最好按照*contracts目录*的结构进行组织：对于其中的每个.sol文件，创建一个相应的测试文件。

现在是编写我们的第一组测试的时候了！这些测试将测试之前介绍过的Box*合约的属性*：一个简单的合约，允许您检索先前所有者存储的值。

在您的项目根目录中创建一个测试目录。我们将把测试保存为test/Box.test.js。每个测试.js文件通常包含单个合同的测试，并以其命名。
```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

// Start test block
describe('Box', function () {
  before(async function () {
    this.Box = await ethers.getContractFactory('Box');
  });

  beforeEach(async function () {
    this.box = await this.Box.deploy();
    await this.box.deployed();
  });

  // Test case
  it('retrieve returns a value previously stored', async function () {
    // Store a value
    await this.box.store(42);

    // Test if the returned value is the same one
    // Note that we need to use strings to compare the 256 bit integers
    expect((await this.box.retrieve()).toString()).to.equal('42');
  });
});
```

>TIP
许多书籍都介绍了如何构建单元测试。请查看[Moloch测试指南](https://github.com/MolochVentures/moloch/tree/4e786db8a4aa3158287e0935dcbc7b1e43416e38/test#moloch-testing-guide)，其中包含一系列旨在测试Solidity智能合约的原则。

我们现在可以运行我们的测试了！
运行 npx truffle test 将执行测试目录中的所有测试，检查您的合约是否按照您的意图工作。

```
npx hardhat test


  Box
    ✓ retrieve returns a value previously stored


  1 passing (578ms)
```
此时，设立一个持续集成服务（如[CircleCI](https://circleci.com/)）非常明智，这样每次将代码提交到GitHub时，测试都会自动运行。

## 执行复杂的断言
许多有趣的合约属性可能很难捕捉，例如：

* 验证合约在出错时恢复

* 测量账户的以太币余额变化量

* 检查是否发出了正确的事件

*OpenZeppelin Test Helpers*是一个旨在帮助您测试所有这些属性的库。它还将简化模拟区块链上时间流逝和处理非常大的数字的任务。

>NOTE
OpenZeppelin Test Helpers 是基于 web3.js 的，因此 Hardhat 用户应该使用 Truffle 插件以确保兼容性。然而，我们建议使用 Hardhat Chai Matchers 作为 Ethers.js 的更好支持的替代方案。

要安装OpenZeppelin测试助手，请运行以下命令：
```
npm install --save-dev @openzeppelin/test-helpers
```
我们可以使用OpenZeppelin Test Helpers来更新我们的测试，以支持非常大的数字，检查事件是否被触发，并检查交易是否回滚。
```
// test/Box.test.js
// Load dependencies
const { expect } = require('chai');

// Import utilities from Test Helpers
const { BN, expectEvent, expectRevert } = require('@openzeppelin/test-helpers');

// Load compiled artifacts
const Box = artifacts.require('Box');

// Start test block
contract('Box', function ([ owner, other ]) {
  // Use large integers ('big numbers')
  const value = new BN('42');

  beforeEach(async function () {
    this.box = await Box.new({ from: owner });
  });

  it('retrieve returns a value previously stored', async function () {
    await this.box.store(value, { from: owner });

    // Use large integer comparisons
    expect(await this.box.retrieve()).to.be.bignumber.equal(value);
  });

  it('store emits an event', async function () {
    const receipt = await this.box.store(value, { from: owner });

    // Test that a ValueChanged event was emitted with the new value
    expectEvent(receipt, 'ValueChanged', { value: value });
  });

  it('non owner cannot store a value', async function () {
    // Test a transaction reverts
    await expectRevert(
      this.box.store(value, { from: other }),
      'Ownable: caller is not the owner',
    );
  });
});
```
这些测试将测试之前指南中的Ownable Box*合约的属性*：一个简单的合约，让您检索所有者先前存储的值。

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
一旦您已经彻底测试了您的合约并且相信它们的正确性，您将想要部署它们到一个真实的网络并开始与它们交互。以下指南将帮助您了解这些主题：
* 连接公共测试网络

* 部署和交互

* 准备主网