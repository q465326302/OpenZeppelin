# Using with Upgrades
如果你的合约将使用可升级性进行部署，例如使用*OpenZeppelin Upgrades插件*，你将需要使用OpenZeppelin Contracts的可升级版本。

这个版本作为一个单独的包，名为@openzeppelin/contracts-upgradeable，托管在[OpenZeppelin/openzeppelin-contracts-upgradeable的库](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable)中。它将@openzeppelin/contracts作为对等依赖项。

它遵循所有编写*可升级合约*的规则：构造函数被初始化函数替代，状态变量在初始化函数中初始化，我们还会检查跨小版本的存储兼容性。

> TIP
OpenZeppelin提供了一整套工具，用于部署和保护可升级的智能合约。查看*完整的资源列表*。

## 概述

### Installation
```
npm install @openzeppelin/contracts-upgradeable @openzeppelin/contracts
```

### 使用
Upgradeable包复制了主要的OpenZeppelin Contracts包的结构，但是每个文件和合约都有Upgradeable后缀。
```
-import {ERC721} from "@openzeppelin/contracts/token/ERC721/ERC721.sol";
+import {ERC721Upgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol";

-contract MyCollectible is ERC721 {
+contract MyCollectible is ERC721Upgradeable {
```

> NOTE
接口和库不包含在可升级包中，而是从主OpenZeppelin合约包中导入的。

构造函数被内部初始化函数替换，遵循__{ContractName}_init的命名约定。由于这些是内部的，您必须始终定义自己的公共初始化函数，并调用您扩展的合约的父初始化器。
```
-    constructor() ERC721("MyCollectible", "MCO") public {
+    function initialize() initializer public {
+        __ERC721_init("MyCollectible", "MCO");
     }
```

> CAUTION
需要特别注意多重继承的使用。请参阅下面标题为*“多重继承”*的部分。

一旦这个合同设置并编译完成，你可以使用*升级插件*进行部署。以下片段显示了使用Hardhat的示例部署脚本。
```
// scripts/deploy-my-collectible.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const MyCollectible = await ethers.getContractFactory("MyCollectible");

  const mc = await upgrades.deployProxy(MyCollectible);

  await mc.waitForDeployment();
  console.log("MyCollectible deployed to:", await mc.getAddress());
}

main();
```

## 多重继承
初始化函数不像构造函数那样被编译器线性化。因此，每个__{ContractName}_init函数都嵌入了对所有父初始化器的线性化调用。因此，调用两个这样的init函数可能会两次初始化同一个合约。

每个合约中都有一个__{ContractName}_init_unchained函数，这是初始化函数减去对父初始化器的调用，可以用来避免双重初始化问题，但不建议手动执行此操作。我们希望能够在未来版本的升级插件中实现此类安全检查。

### Namespaced Storage
你可能会注意到，合约使用带有@custom:storage-location erc7201:<NAMESPACE_ID>注解的结构体来存储合约的状态变量。这遵循[ERC-7201: Namespaced Storage Layout](https://eips.ethereum.org/EIPS/eip-7201)模式，其中每个合约在继承链中的其他合约之外都有自己的存储布局。

如果没有命名空间存储，简单地添加一个状态变量是不安全的，因为它会“向下移动”继承链中的所有状态变量。这使得存储布局不兼容，如在编写*可升级合约*中所解释的。

Upgradeable包中使用的命名空间存储模式使我们能够在未来自由地添加新的状态变量，而不会影响与现有部署的存储兼容性。它还允许改变继承顺序，只要所有继承的合约都使用命名空间存储，就不会影响最终的存储布局。