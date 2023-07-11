# Contracts
一个安全的智能合约开发库。基于社区审查的可靠代码构建。
* 实现了*ERC20*和*ERC721*等标准。
* 灵活的*基于角色的权限控制*方案。
* 可重用的*Solidity组件*，用于构建自定义合约和复杂的去中心化系统。
* 与Gas Station Network的一流集成，适用于无燃料费用的系统！
* 由领先的安全公司审计。

## Overview

### 安装
```
npm install @openzeppelin/contracts
```
OpenZeppelin Contracts具有稳定的API功能，这意味着当升级到较新的次要版本时，您的合约不会意外中断。

### 使用方法
安装后，您可以通过导入它们来使用库中的合约：
```
pragma solidity ^0.5.0;

import "@openzeppelin/contracts/token/ERC721/ERC721Full.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721Mintable.sol";

contract MyNFT is ERC721Full, ERC721Mintable {
    constructor() ERC721Full("MyNFT", "MNFT") public {
    }
}
```

> TIP
如果您刚开始进行智能合约开发，请前往*开发智能合约*了解如何创建新项目并编译您的合约。

为了保证系统的安全性，您应该**始终**使用已安装的代码，不要从在线来源复制粘贴，也不要自行修改。

### 了解更多

侧边栏中的指南将教您不同的概念以及如何使用OpenZeppelin Contracts提供的相关合约：

* *访问控制*：决定谁可以执行系统上的每个操作。

* *代币*：创建可交易的资产或集体，并通过众筹进行*分发*。

* *加油站网络*：让用户与您的合约进行交互，而无需支付燃料费用。

* *实用工具*：通用的有用工具，包括不溢出的数学、签名验证和无信任支付系统。

完整的*API文档*也有详细的说明，是开发智能合约应用程序时的重要参考资料。您还可以在[社区论坛](https://forum.openzeppelin.com/)上寻求帮助或关注Contracts的开发。

最后，您可能还想查看我们[博客上的指南](https://blog.openzeppelin.com/guides/)，其中涵盖了几种常见的用例和良好的实践方法。以下文章提供了很好的背景阅读材料，但请注意，由于生态系统中的工具不断快速演变，其中一些引用的工具已经发生了变化。

* [以太坊智能合约之旅](https://blog.openzeppelin.com/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05)将帮助您了解智能合约开发的各种可用工具，并帮助您设置环境。

* [以太坊编程简介](https://blog.openzeppelin.com/a-gentle-introduction-to-ethereum-programming-part-1-783cc7796094)，第一部分提供了非常有用的入门级信息，包括以太坊平台的许多基本概念。

如果您希望更深入地了解，可以阅读指南设计您的[以太坊应用程序的架构](https://blog.openzeppelin.com/designing-the-architecture-for-your-ethereum-application-9cec086f8317)，该指南讨论了如何更好地构建应用程序及其与现实世界的关系。