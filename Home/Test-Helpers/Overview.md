# Test Helpers
用于以太坊智能合约测试的断言库。确保您的合约的作用符合预期！

* 检查[事务回滚](./API-Reference.md#expectrevert)的原因以及是否以正确

* 验证[事件](./API-Reference.md#expectevent)是否以正确的值进行了发送

* 优雅地跟踪[余额变化](./API-Reference.md#balance)

* 处理[非常大的数字](./API-Reference.md#bn)

* 模拟[时间的流逝](./API-Reference.md#time)

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
    // 捆绑的BN库是web3在内部使用的相同库。the hood
    this.value = new BN(1);

    this.erc20 = await ERC20.new();
  });

  it('reverts when transferring tokens to the zero address', async function () {
    // 可以精确测试触发 require 语句的条件。
    await expectRevert(
      this.erc20.transfer(constants.ZERO_ADDRESS, this.value, { from: sender }),
      'ERC20: transfer to the zero address',
    );
  });

  it('emits a Transfer event on successful transfers', async function () {
    const receipt = await this.erc20.transfer(
      receiver, this.value, { from: sender }
    );

    // 事件断言可以验证参数是否是预期的参数。
    expectEvent(receipt, 'Transfer', {
      from: sender,
      to: receiver,
      value: this.value,
    });
  });

  it('updates balances on successful transfers', async function () {
    this.erc20.transfer(receiver, this.value, { from: sender });

    // 如果使用Chai，BN断言可以通过chai-bn自动使用。
    expect(await this.erc20.balanceOf(receiver))
      .to.be.bignumber.equal(this.value);
  });
});
```

了解更多
* 前往[配置](./Configuration.md)以获取高级设置。

* 要获取详细的使用信息，请查看[API参考](./API-Reference.md)。
