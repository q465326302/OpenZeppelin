# Contracts
一个用于安全智能合约开发的库。在经过社区验证的代码基础上构建。

实现了诸如*ERC20*和*ERC721*之类的标准。

灵活*的基于角色的权限*控制方案。

可重用的*Solidity组件*，用于构建自定义合约和复杂的去中心化系统。

## 概述

### 安装
```
npm install @openzeppelin/contracts
```

OpenZeppelin合约具有*稳定的API*功能，这意味着在升级到新的次要版本时，您的合约不会意外中断。

使用方法
安装后，您可以通过导入它们来使用库中的合约：
```
// contracts/MyNFT.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    constructor() ERC721("MyNFT", "MNFT") {
    }
}
```

>TIP
如果您是智能合约开发的新手，请前往*“开发智能合约”*了解如何创建新项目并编译您的合约。

为了保证系统安全，您应**始终**使用安装的代码，不应从在线源复制粘贴或自行修改。该库的设计使得只有您使用的合约和函数被部署，因此您不必担心它会不必要地增加燃气成本。

## 安全性
如果您发现任何安全问题，请通过我们在[Immunefi上的漏洞赏金计划](https://www.immunefi.com/bounty/openzeppelin)或直接发送至security@openzeppelin.org进行报告。

[安全中心](https://contracts.openzeppelin.com/security)包含有关安全开发过程的更多详细信息。

## 了解更多
侧边栏中的指南将教授不同的概念以及如何使用OpenZeppelin Contracts提供的相关合约：

* *访问控制*：决定谁可以在您的系统上执行每个操作。

* *代币*：创建可交易的资产或收藏品，例如著名的*ERC20*和*ERC721*标准。

* *实用工具*：通用有用的工具，包括非溢出的数学、签名验证和无信任支付系统。

*完整的API*也有详细的文档，并且在开发智能合约应用程序时是一个很好的参考。您还可以在社区论坛中寻求帮助或关注合约的开发。

最后，您可能需要查看我们[博客](https://blog.openzeppelin.com/guides/)上的指南，其中涵盖了几个常见用例和良好实践。以下文章提供了很好的背景阅读，但请注意，一些引用的工具随着生态系统中的工具不断快速发展而发生了变化。

* [《以太坊智能合约搭便车指南》](https://blog.openzeppelin.com/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05)将帮助您了解智能合约开发的各种可用工具，并帮助您设置环境。

* [《以太坊编程的温柔介绍，第1部分》](https://blog.openzeppelin.com/a-gentle-introduction-to-ethereum-programming-part-1-783cc7796094)提供了非常有用的信息，包括来自以太坊平台的许多基本概念。

* 对于更深入的研究，您可以阅读指南[“为您的以太坊应用程序设计架构”](https://blog.openzeppelin.com/designing-the-architecture-for-your-ethereum-application-9cec086f8317)，其中讨论如何更好地构建应用程序及其与现实世界的关系。