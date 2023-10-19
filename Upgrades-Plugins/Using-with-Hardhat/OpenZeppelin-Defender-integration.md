# OpenZeppelin Defender integration
Hardhat Upgrades包可以使用[OpenZeppelin Defender](/Defender/Overview.md)进行部署，而不是使用ethers.js，这样可以使用诸如gas价格估计、重新提交以及自动化字节码和源代码验证等功能。

> NOTE
OpenZeppelin Defender部署目前处于测试阶段，这里描述的功能可能会发生变化。

## 配置
在OpenZeppelin Defender上创建一个部署环境，并在你的hardhat.config.js或hardhat.config.ts文件中的defender下提供团队API密钥和秘密：
```
module.exports = {
  defender: {
    apiKey: process.env.API_KEY,
    apiSecret: process.env.API_SECRET,
  }
}
```

## 用法
在使用[Hardhat Upgrades API函数](../API-Reference/Hardhat-Upgrades.md)时，使用以下任何一种方式启用OpenZeppelin Defender部署。

> NOTE
只有在API参考中有useDefenderDeploy选项的函数才支持通过OpenZeppelin Defender进行部署。如果你启用了以下内容，但使用的函数不支持useDefenderDeploy，那么下面的第一种方式会导致这些函数使用ethers.js进行部署，而第二种和第三种方式会导致这些函数出错。

* 推荐：在hardhat.config.js或hardhat.config.ts中，将useDefenderDeploy: true设置在defender下。例如：
```
module.exports = {
  defender: {
    apiKey: process.env.API_KEY,
    apiSecret: process.env.API_SECRET,
    useDefenderDeploy: true,
  }
}
```

```
// scripts/create-box.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const Box = await ethers.getContractFactory("Box");
  const box = await upgrades.deployProxy(Box, [42]);
  await box.waitForDeployment();
  console.log("Box deployed to:", await box.getAddress());
}

main();
```

* 使用defender模块代替Hardhat运行时环境的upgrades。如果你想确保使用Defender并希望在函数不支持Defender时看到错误，可以使用这种方式。例如：
```
// scripts/create-box.js
const { ethers, defender } = require("hardhat");

async function main() {
  const Box = await ethers.getContractFactory("Box");
  const box = await defender.deployProxy(Box, [42]);
  await box.waitForDeployment();
  console.log("Box deployed to:", await box.getAddress());
}

main();
```

* 使用useDefenderDeploy通用选项。设置此选项可以覆盖上述特定函数。例如：
```
// scripts/create-box.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const Box = await ethers.getContractFactory("Box");
  const box = await upgrades.deployProxy(Box, [42], { useDefenderDeploy: true });
  await box.waitForDeployment();
  console.log("Box deployed to:", await box.getAddress());
}

main();
```