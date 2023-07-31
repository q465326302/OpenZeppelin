# Upgrades

OpenZeppelin提供工具来部署和保护可升级的智能合约。

* [Upgrades Plugins](../Upgrades%20Plugins/Overview.md) 用于部署可升级合约，并进行自动化安全检查。
* [Upgradeable Contracts](../Contracts/Contracts.4.x/Using%20with%20Upgrades.md) 使用我们的 Solidity 组件构建您的合约。
* [Defender Admin](../Defender/Components/Admin/Admin.md/#升级) 用于管理生产环境中的升级，并自动化运营。

在下面找到我们所有与可升级性相关的资源。

> TIP
如果您不知道从哪里开始，我们建议从学习：[升级智能合约开始](../Learn/Upgrading%20smart%20contracts/Upgrading%20smart%20contracts-truffle.md)。

## Resources

### [升级插件](../Upgrades%20Plugins/Overview.md)
这些插件适用于Hardhat和Truffle，可以抽象出升级的复杂性，并在运行自动化安全检查时确保成功的升级。

### [编写可升级的合同](../Upgrades%20Plugins/Writing%20Upgradeable%20Contracts.md)
在使用OpenZeppelin Upgrades处理可升级合约时，编写Solidity代码时需要注意一些小细节。

### [OpenZeppelin可升级合同](../Upgrades%20Plugins/Using%20with%20Truffle.md)
这是一个流行的OpenZeppelin Contracts库的变体，其中包含了所有升级合约所需的必要修改。

### [代理合同](../Contracts/Contracts.4.x/Using%20with%20Upgrades.md)
所有可用的代理合约和相关工具的完整列表，以及与低级使用相关的文档，而不需要升级插件。

### [智能合约升级的现状](../Contracts/Contracts.4.x/API/Proxy.md)
对升级模式进行调查，并提出升级管理和治理的良好实践和建议。

### [透明代理和UUPS代理的对比](https://blog.openzeppelin.com/the-state-of-smart-contract-upgrades/)
透明代理模式和新推出的 UUPS 代理之间的区别在于它们的实现方式和功能。

### [学习：升级智能合约](../Contracts/Contracts.4.x/API/Proxy.md/#透明代理和uups代理之间的区别)
我们的Learn系列中的一章，讲述了智能合约开发的升级。适用于Hardhat和Truffle。

### [通过多签钱包升级](../Defender/Guides/Upgrading%20via%20a%20multisig/Upgrading%20via%20a%20multisig.md)
一份Defender指南，介绍如何使用多签钱包来升级一个由Defender Admin和Hardhat Upgrades插件保护的生产智能合约。

### [UUPS代理教程](https://forum.openzeppelin.com/t/uups-proxies-tutorial-solidity-javascript/7786)
关于如何使用UUPS代理模式的教程：Solidity代码应该是什么样的，以及如何使用升级插件与这个新的代理模式。