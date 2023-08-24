# Contracts
**一个专注于安全智能合约开发的库。**基于社区审核的代码打造。

* 实现了[ERC20](./Tokens/ERC20/ERC20.md)和[ERC721](./Tokens/ERC721.md)等标准。

* 灵活的[基于角色的权限控制](./Access-Control.md)控制方案。

* 可重用的[Solidity组件](./Utilities.md)，用于构建自定义合约和复杂的去中心化系统。

* 与*Gas Station Network*实现了一流的集成，可以实现无燃气费用的系统！

* 通过领先的安全公司进行了[审计](https://github.com/OpenZeppelin/openzeppelin-contracts/tree/master/audit)（上一次完整审计是在v2.0.0版本）。

## 概述

### 安装
```
npm install @openzeppelin/contracts
```
OpenZeppelin Contracts提供了[稳定的API](./Releases&Stability.md#api稳定性)，这意味着当升级到较新的次要版本时，您的合约不会意外中断。

### 用法
一旦安装完成，您可以通过导入它们来使用库中的合约：
```
// contracts/MyNFT.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    constructor() ERC721("MyNFT", "MNFT") public {
    }
}
```

> TIP
如果您是智能合约开发的新手，请前往[“开发智能合约”](../../Learn/Developing-smart-contracts/Developing-smart-contracts-hardh.md)以了解如何创建新项目和编译您的合约。

为了保护您的系统安全，您应**始终**使用已安装的代码，不要从在线源代码中复制粘贴，也不要自行修改。该库的设计使得只有您使用的合约和函数被部署，因此您不需要担心它会不必要地增加燃气成本。

## 了解更多

侧边栏中的指南将教您不同的概念，以及如何使用OpenZeppelin Contracts提供的相关合约：

* [访问控制](./Access-Control.md)：决定谁可以执行系统上的每个操作。

* [代币](./Tokens/Tokens.md)：创建可交易的资产或收藏品，如众所周知的[ERC20](./Tokens/ERC20/ERC20.md)和[ERC721](./Tokens/ERC721.md)标准。

* [Gas Station Network](./API/GSN.md)：让您的用户与您的合约交互，而无需为燃气支付费用。

* [实用工具](../Contracts.4.x/Utilities.md)：通用的有用工具，包括非溢出数学、签名验证和无信任支付系统。

*完整的API*也有详细的文档，并且在开发智能合约应用程序时可以作为很好的参考。您还可以在社区论坛中寻求帮助或关注[Contracts的开发](https://forum.openzeppelin.com/)。

最后，您可能还想看一下我们博客上的[指南](https://blog.openzeppelin.com/guides/)，其中涵盖了几种常见的用例和良好的实践方法。以下文章提供了很好的背景阅读，但请注意，由于生态系统中的工具不断快速演变，其中一些引用的工具可能已经发生了变化。

* [以太坊智能合约的自助指南](https://blog.openzeppelin.com/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05)将帮助您了解智能合约开发的各种可用工具，并帮助您设置环境。

* [以太坊编程的温和介绍](https://blog.openzeppelin.com/a-gentle-introduction-to-ethereum-programming-part-1-783cc7796094)，第1部分提供了非常有用的入门级信息，包括以太坊平台的许多基本概念。

* 如果您想更深入地了解，可以阅读指南[“设计您的以太坊应用程序的架构”](https://blog.openzeppelin.com/designing-the-architecture-for-your-ethereum-application-9cec086f8317)，该指南讨论了如何更好地构建您的应用程序及其与现实世界的关系。