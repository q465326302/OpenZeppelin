# Using with Upgrades
如果您的合约将使用可升级性部署，例如使用*OpenZeppelin Upgrades插件*，您将需要使用OpenZeppelin Contracts的升级安全变体。

该变体作为一个名为@openzeppelin/contracts-upgradeable的单独包可用，该包托管在[OpenZeppelin/openzeppelin-contracts-upgradeable存储库](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable)中。

它遵循*编写可升级合约*的所有规则：构造函数被替换为初始化函数，状态变量在初始化函数中进行初始化，并且我们还会在不同次要版本之间检查存储不兼容性。

## 概述
### 安装
```
npm install @openzeppelin/contracts-upgradeable
```
### 使用
该包复制了主要的OpenZeppelin Contracts包的结构，但每个文件和合约都有Upgradeable后缀。
```
-import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
+import "@openzeppelin/contracts-upgradeable/token/ERC721/ERC721Upgradeable.sol";

-contract MyCollectible is ERC721 {
+contract MyCollectible is ERC721Upgradeable {
```

构造函数被内部初始化函数所取代，遵循命名约定__{ContractName}_init。由于这些函数是内部函数，您必须始终定义自己的公共初始化函数，并调用您扩展的合约的父初始化函数。

```
-    constructor() ERC721("MyCollectible", "MCO") public {
+    function initialize() initializer public {
+        __ERC721_init("MyCollectible", "MCO");
     }
```

> CAUTION
使用多重继承需要特别注意。请参阅下面标题为"*多重继承*"的部分。

一旦设置并编译了这个合约，你可以使用*Upgrades插件*部署它。以下代码片段展示了使用Hardhat的示例部署脚本。

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

初始化函数不像构造函数一样由编译器进行线性化。因此，每个__{ContractName}_init函数都嵌入了对所有父级初始化函数的线性化调用。因此，调用两个这些init函数可能会对同一个合约进行两次初始化。

每个合约中都可以找到__{ContractName}_init_unchained函数，它是初始化函数减去对父级初始化函数的调用，可以用于避免双重初始化问题，但不建议手动进行此操作。我们希望能够在未来版本的Upgrades插件中实现对此的安全检查。

### 存储间隙

您可能会注意到每个合约都包含一个名为__gap的状态变量。这是在可升级合约中放置的空的保留存储空间。它允许我们在将来自由地添加新的状态变量，而不会影响与现有部署的存储兼容性。

简单地添加一个状态变量是不安全的，因为它会将下面继承链中的所有状态变量"向下移动"。这会使存储布局不兼容，如在*编写可升级合约*中所解释的那样。__gap数组的大小是根据合约使用的存储量总和计算的（在这种情况下为50个存储槽）。