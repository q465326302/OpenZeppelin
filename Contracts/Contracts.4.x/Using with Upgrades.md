# Using with Upgrades
如果您的合约将使用可升级性部署，例如使用*OpenZeppelin Upgrades插件*，您将需要使用OpenZeppelin Contracts的可升级变体。

这个变体作为一个单独的包@openzeppelin/contracts-upgradeable可用，它托管在[OpenZeppelin/openzeppelin-contracts-upgradeable存储库](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable)中。

它遵循编写*可升级合约的所有规则*：构造函数被替换为初始化函数，在初始化函数中初始化状态变量，并且我们还在不同的小版本之间检查存储不兼容性。

> TIP
OpenZeppelin提供了完整的工具套件，用于部署和保护可升级的智能合约。请查看*完整的资源列表*。

## 概述

### 安装
```
npm install @openzeppelin/contracts-upgradeable
```

### 用法
该软件包复制了主要的OpenZeppelin Contracts软件包的结构，但是每个文件和合约都有Upgradeable后缀。
```
-import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
+import "@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol";

-contract MyCollectible is ERC721 {
+contract MyCollectible is ERC721Upgradeable {
```

构造函数被内部初始化函数所替代，遵循命名约定__{ContractName}_init。由于这些是内部的，您必须始终定义自己的公共初始化函数并调用您扩展的合约的父初始化函数。
```
-    constructor() ERC721("MyCollectible", "MCO") public {
+    function initialize() initializer public {
+        __ERC721_init("MyCollectible", "MCO");
     }
```

>CAUTION
在使用多重继承时需要特别注意。请参见下面标题为*“多重继承”*的部分。

一旦设置并编译了此合约，您可以使用*Upgrades插件*部署它。以下代码片段展示了使用Hardhat的示例部署脚本。

```
// scripts/deploy-my-collectible.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const MyCollectible = await ethers.getContractFactory("MyCollectible");

  const mc = await upgrades.deployProxy(MyCollectible);

  await mc.deployed();
  console.log("MyCollectible deployed to:", mc.address);
}

main();
```

## 进一步说明

### 多重继承
初始化函数不像构造函数一样被编译器线性化。因此，每个__{ContractName}_init函数都嵌入了所有父初始化器的线性化调用。因此，调用这些init函数中的两个可以潜在地初始化相同的合约两次。

每个合约中都有一个名为__{ContractName}_init_unchained的函数，它是初始化函数减去对父级初始化器的调用，可以用来避免双重初始化问题，但不建议手动执行此操作。我们希望能够在未来的Upgrades插件版本中实现安全检查。

### 存储间隙
您可能会注意到每个合约都包括一个名为__gap的状态变量。这是在可升级合约中放置的空保留存储空间。它允许我们在未来自由地添加新的状态变量，而不会影响与现有部署的存储兼容性。

简单地添加状态变量是不安全的，因为它会“向下移动”继承链下面的所有状态变量。这使得存储布局不兼容，如在*编写可升级合约*中所解释的那样。__gap数组的大小是根据合约使用的存储量总是添加到相同的数字（在此情况下为50个存储槽）来计算的。