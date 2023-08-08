# Configuration
测试助手需要进行一些轻量级配置，以正确连接到您的测试环境。

在可能的情况下，这将自动完成，但某些情况需要手动设置。这可以通过调用configure函数来实现。：
```
// Apply configuration
require('@openzeppelin/test-helpers/configure')({ ... });

// Import the helpers
const { expectRevert } = require('@openzeppelin/test-helpers');
```

配置必须在所有测试文件中应用，在导入助手之前。

### provider
用于通过web3提供者（如[ balance](./API%20Reference.md)和[time](./API%20Reference.md)）连接到测试环境的助手。

在Truffle环境中，web3提供者将从Truffle的全局web3实例中获取。否则，默认值为http://localhost:8545/。

您可以通过provider键重写此行为并配置自己的提供者：
```
require('@openzeppelin/test-helpers/configure')({
  provider: 'http://localhost:8080',
});
```

### singletons
singletons助手返回合约对象，这些对象有多个可以配置的设置：

* abstraction：底层合约抽象类型，'web3'表示web3-eth-contract，'truffle'表示@truffle/contract实例。默认为'web3'，除非检测到Truffle环境。

* defaultGas：当事务的gas字段未指定时分配多少gas。默认为200k。

* defaultSender：当事务的from字段未指定时要使用的发送者地址。没有默认值。

虽然自动检测和默认值应该涵盖大多数用例，但所有值都可以手动提供：
```
require('@openzeppelin/test-helpers/configure')({
  singletons: {
    abstraction: 'web3',
    defaultGas: 6e6,
    defaultSender: '0x5a0b5...',
  },
});
```

## 关于Truffle迁移
在Truffle迁移中，自动检测Truffle环境无法正常工作，因此必须在那里手动配置助手。
```
require('@openzeppelin/test-helpers/configure')({
  provider: web3.currentProvider,
  singletons: {
    abstraction: 'truffle',
  },
});
```