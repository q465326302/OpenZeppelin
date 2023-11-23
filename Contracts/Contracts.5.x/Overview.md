# Contracts
**一个用于安全智能合约开发的库。**基于社区审核的代码构建坚实的基础。

* 实现了如[ERC20](./Tokens/ERC20/ERC20.md)和[ERC721](./Tokens/ERC721.md)等标准。

* 灵活的[基于角色的控制权限](./Access-Control.md)方案。

* 可重用的[Solidity组件](./Utilities.md)，用于构建自定义合约和复杂的去中心化系统。

> IMPORTANT
OpenZeppelin合约使用语义版本控制来传达其API和存储布局的向后兼容性。对于可升级的合约，应假定不同主要版本的存储布局是不兼容的，例如，从4.9.3升级到5.0.0是不安全的。在[向后兼容性](./Backwards-Compatibility.md)中了解更多。

## Overview

### Installation

#### Hardhat, Truffle (npm)
```
npm install @openzeppelin/contracts
```

#### Foundry (git)
> WARNING
通过git安装时，常见的错误是使用master分支。这是一个开发分支，应避免使用，而应选择标记的发布版本。发布过程涉及到master分支不能保证的安全措施。

> WARNING
Foundry初始安装的是最新版本，但随后的forge更新命令将使用master分支。

```
forge install OpenZeppelin/openzeppelin-contracts
```

在remappings.txt中添加@openzeppelin/contracts/=lib/openzeppelin-contracts/contracts/。

### Usage
安装后，你可以通过导入它们来使用库中的合约:
```
// contracts/MyNFT.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC721} from "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    constructor() ERC721("MyNFT", "MNFT") {
    }
}
```

> TIP
如果你是智能合约开发的新手，可以去[开发智能合约](/Learn/Developing-smart-contracts/Developing-smart-contracts-hardh.md)了解如何创建新项目和编译你的合约。

为了保证你的系统安全，你应该**始终**使用已安装的代码，不要从网上复制粘贴，也不要自己修改。这个库设计得只有你使用的合约和函数才会被部署，所以你不需要担心它会无谓地增加gas费用。

## Security
如果你发现任何安全问题，请通过我们在[Immunefi的漏洞赏金计划](https://www.immunefi.com/bounty/openzeppelin)或直接向security@openzeppelin.org报告。

[安全中心](https://contracts.openzeppelin.com/security)包含了更多关于安全开发过程的详细信息。

## Learn More
侧边栏的指南将教你关于不同概念的知识，以及如何使用OpenZeppelin合约提供的相关合约：

* [访问控制](./Access-Control.md)：决定谁可以在你的系统上执行每个操作。

* [代币](./Tokens/Tokens.md)：创建可交易的资产或收藏品，如众所周知的[ERC20](./Tokens/ERC20/ERC20.md)和[ERC721](./Tokens/ERC721.md)标准。

* [实用工具](./Utilities.md)：包括不溢出的数学、签名验证和无需信任的支付系统等通用有用的工具。

[完整的API](./API/ERC-20.md)也有详细的文档，是开发你的智能合约应用时的极好参考。你也可以在[社区论坛](https://forum.openzeppelin.com/)中寻求帮助或关注合约的开发。

最后，你可能想看看我们[博客](https://blog.openzeppelin.com/guides/)上的指南，它们涵盖了几种常见的用例和良好的实践。以下文章提供了很好的背景阅读，尽管请注意，一些参考的工具已经随着生态系统中的工具快速发展而改变。

* [《以太坊智能合约的浏览指南》](https://blog.openzeppelin.com/the-hitchhikers-guide-to-smart-contracts-in-ethereum-848f08001f05)将帮助你了解可用于智能合约开发的各种工具，并帮助你设置你的环境。

* [《以太坊编程入门指南，第一部分》](https://blog.openzeppelin.com/a-gentle-introduction-to-ethereum-programming-part-1-783cc7796094)提供了非常有用的入门级信息，包括以太坊平台的许多基本概念。

对于更深入的研究，你可以阅读[《设计你的以太坊应用的架构》指南](https://blog.openzeppelin.com/designing-the-architecture-for-your-ethereum-application-9cec086f8317)，该指南讨论了如何更好地构建你的应用及其与现实世界的关系。