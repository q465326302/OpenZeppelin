# Contracts
**一个专注于安全智能合约开发的库。**基于社区审核的代码打造。

实现了如[ERC20](./Tokens/ERC20/ERC20.md)和[ERC721](./Tokens/ERC721.md)等标准。

此外，该库提供了灵活的[基于角色的权限控制](./Access-Control.md)方案。

同时，Contracts还提供了可重用的[Solidity组件](./Utilities.md)，用于构建自定义合约和复杂的去中心化系统。

## 概述

### 安装

```
npm install @openzeppelin/contracts
```

OpenZeppelin合约具有[稳定的API](./Releases&Stability.md#api稳定性)功能，这意味着在升级到新的次要版本时，你的合约不会意外中断。

使用方法
安装后，你可以通过导入来使用库中的合约：
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

> TIP
如果你是智能合约开发的新手，请前往[“开发智能合约”](../../Learn/Developing-smart-contracts/Developing-smart-contracts-hardh.md)了解如何创建新项目并编译你的合约。

为了保证系统安全，你应**始终**使用已安装的代码，不应从在线源复制粘贴,也不要自行修改代码。该库的设计使得只有你使用的合约和函数被部署，因此你不必担心它会不必要地增加gas成本。

## 安全性
如果你发现任何安全问题，请通过我们在[Immunefi上的漏洞赏金计划](https://www.immunefi.com/bounty/openzeppelin)或直接发送至security@openzeppelin.org进行报告。

[安全中心](https://contracts.openzeppelin.com/security)包含有关安全开发过程的更多详细信息。

## 了解更多
接下来的指南将指导不同的概念以及如何使用OpenZeppelin Contracts提供的相关合约：

* [访问控制](./Access-Control.md)：决定谁可以在你的系统上执行每个操作。

* [代币](./Tokens/Tokens.md)：创建可交易的资产或收藏品，例如著名的[ERC20](./Tokens/ERC20/ERC20.md)和[ERC721](./Tokens/ERC721.md)标准。

* [实用工具](../Contracts.4.x/Utilities.md)：通用的有用工具，包括防止溢出的数学、签名验证和无需信任的支付系统。

[完整的API](../Contracts.4.x/API/ERC20.md)也有详细的文档，并且在开发智能合约应用程序时是一个很好的参考。你还可以在[社区论坛](https://forum.openzeppelin.com/)中寻求帮助或关注合约的开发。

最后，我们建议你查看我们[博客](https://blog.openzeppelin.com/guides/)上的指南，其中涵盖了几个常见用例和良好的实践方法。以下文章提供了很好的背景阅读，但请注意，由于生态系统中的工具不断快速演变，一些文章中引用的工具可能已经发生了变化。

* [《以太坊智能合约搭便车指南》](https://blog.openzeppelin.com/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05)将帮助你了解智能合约开发的各种可用工具，并帮助你设置环境。

* [《以太坊编程的温柔介绍，第1部分》](https://blog.openzeppelin.com/a-gentle-introduction-to-ethereum-programming-part-1-783cc7796094)提供了非常有用的信息，包括以太坊平台的许多基本概念。

* 对于更深入的研究，你可以阅读指南[“为你的以太坊应用程序设计架构”](https://blog.openzeppelin.com/designing-the-architecture-for-your-ethereum-application-9cec086f8317)，其中讨论如何更好地构建应用程序及其与现实世界的关系。