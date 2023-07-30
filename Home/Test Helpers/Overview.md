# Test Helpers
用于以太坊智能合约测试的断言库。确保您的合约的行为符合预期！

* *检查事务*是否以正确的原因回滚

* *验证事件*是否以正确的值进行了发射

* 优雅地*跟踪余额*变化

* 处理*非常大的数字*

* 模拟*时间的流逝*

## 概述

### 安装
```
npm install --save-dev @openzeppelin/test-helpers
```

#### Hardhat（以前的Buidler）
安装web3和hardhat-web3插件。
```
npm install --save-dev @nomiclabs/hardhat-web3 web3
```

请记得按照[安装说明](https://hardhat.org/plugins/nomiclabs-hardhat-web3.html#installation)将插件包含在您的配置中。

### 用法
在您的测试文件中导入@openzeppelin/test-helpers以访问不同的断言和实用程序。
```
const {
  BN,           // Big Number support
  constants,    // Common constants, like the zero address and largest integers
  expectEvent,  // Assertions for emitted events
  expectRevert, // Assertions for transactions that should fail
} = require('@openzeppelin/test-helpers');

const ERC20 = artifacts.require('ERC20');

describe('ERC20', function ([sender, receiver]) {
  beforeEach(async function () {
    // The bundled BN library is the same one web3 uses under the hood
    this.value = new BN(1);

    this.erc20 = await ERC20.new();
  });

  it('reverts when transferring tokens to the zero address', async function () {
    // Conditions that trigger a require statement can be precisely tested
    await expectRevert(
      this.erc20.transfer(constants.ZERO_ADDRESS, this.value, { from: sender }),
      'ERC20: transfer to the zero address',
    );
  });

  it('emits a Transfer event on successful transfers', async function () {
    const receipt = await this.erc20.transfer(
      receiver, this.value, { from: sender }
    );

    // Event assertions can verify that the arguments are the expected ones
    expectEvent(receipt, 'Transfer', {
      from: sender,
      to: receiver,
      value: this.value,
    });
  });

  it('updates balances on successful transfers', async function () {
    this.erc20.transfer(receiver, this.value, { from: sender });

    // BN assertions are automatically available via chai-bn (if using Chai)
    expect(await this.erc20.balanceOf(receiver))
      .to.be.bignumber.equal(this.value);
  });
});
```

了解更多
* 前往*配置*以获取高级设置。

* 要获取详细的使用信息，请查看*API参考*。
* 