# Using with OpenZeppelin Platform

Hardhat升级包可以使用OpenZeppelin平台来进行部署，而不是使用ethers.js，这样可以实现功能，如估算燃气价格、重新提交以及自动字节码和源代码验证。

**注意**：OpenZeppelin平台处于测试版阶段，这里描述的功能可能会有所变化。

## 配置
在hardhat.config.js或hardhat.config.ts文件的platform部分中创建一个OpenZeppelin平台的部署环境，并提供团队API密钥和密钥。
```
module.exports = {
  platform: {
    apiKey: process.env.API_KEY,
    apiSecret: process.env.API_SECRET,
  }
}
```

## 使用
在使用*Hardhat Upgrades API函数*时，可以通过以下任何一种方式启用OpenZeppelin平台部署。

> NOTE
只有在API参考中具有usePlatformDeploy选项的函数才支持通过OpenZeppelin平台进行部署。如果您启用了以下选项，但使用不支持usePlatformDeploy的函数，则第一种方式将导致这些函数使用ethers.js进行部署，而第二种和第三种方式将导致这些函数报错。

* 推荐方法：在hardhat.config.js或hardhat.config.ts中，在platform下设置usePlatformDeploy: true。例如：
```
module.exports = {
  platform: {
    apiKey: process.env.API_KEY,
    apiSecret: process.env.API_SECRET,
    usePlatformDeploy: true,
  }
}
```

```
// scripts/create-box.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const Box = await ethers.getContractFactory("Box");
  const box = await upgrades.deployProxy(Box, [42]);
  await box.deployed();
  console.log("Box deployed to:", box.address);
}

main();
```
* 使用platform模块而不是从Hardhat Runtime Environment中进行升级。如果您想确保使用Platform并且希望在函数不支持Platform时看到错误，请使用此方法。例如：

```
// scripts/create-box.js
const { ethers, platform } = require("hardhat");

async function main() {
  const Box = await ethers.getContractFactory("Box");
  const box = await platform.deployProxy(Box, [42]);
  await box.deployed();
  console.log("Box deployed to:", box.address);
}

main();
```

* 使用usePlatformDeploy常见选项。设置此选项将针对特定函数覆盖上述设置。例如：

```
// scripts/create-box.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const Box = await ethers.getContractFactory("Box");
  const box = await upgrades.deployProxy(Box, [42], { usePlatformDeploy: true });
  await box.deployed();
  console.log("Box deployed to:", box.address);
}

main();
```
