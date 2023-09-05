# Using with Hardhat
此软件包为您的Hardhat脚本添加了功能，以便您可以部署和升级合约的代理。依赖于ethers.js。

> TIP
请查看[逐步教程](https://forum.openzeppelin.com/t/openzeppelin-buidler-upgrades-step-by-step-tutorial/3580)，从创建、测试和部署开始，一直到使用Gnosis Safe升级。

## 安装
```
npm install --save-dev @openzeppelin/hardhat-upgrades
npm install --save-dev '@nomiclabs/hardhat-ethers@^2.0.0' 'ethers@^5.0.0' # peer dependencies
```

并在您的hardhat.config.js中注册插件：
```
require('@openzeppelin/hardhat-upgrades');
```

## 在脚本中的使用

### 代理

您可以在[Hardhat脚本](https://hardhat.org/guides/scripts.html)中使用此插件通过deployProxy函数部署一个可升级的合约实例：
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

这将自动检查Box合约是否可以升级，如果需要的话，设置代理管理员，部署Box合约的实现合约（除非之前已经有一个），创建代理，并通过调用initialize(42)来初始化它。

然后，在另一个脚本中，您可以使用upgradeProxy函数将部署的实例升级到新版本。新版本可以是不同的合约（如BoxV2），或者您可以只修改现有的Box合约并重新编译它-插件会注意到它已更改。
```
// scripts/upgrade-box.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const BoxV2 = await ethers.getContractFactory("BoxV2");
  const box = await upgrades.upgradeProxy(BOX_ADDRESS, BoxV2);
  console.log("Box upgraded");
}

main();
```

    注意：尽管此插件会跟踪您在每个网络上部署的所有实现合约，以便重用它们并验证存储兼容性，但它不会跟踪您已部署的代理。这意味着您需要手动跟踪每个部署地址，以在需要时提供给升级函数。

该插件将负责比较BoxV2与先前版本，以确保它们在升级时兼容，部署新的BoxV2实现合约（除非已经有一个来自先前的部署），并将现有代理升级到新的实现版本。

### Beacon代理

您还可以使用此插件使用deployBeacon函数部署一个可升级的Beacon合约，然后使用deployBeaconProxy函数部署一个或多个指向该Beacon合约的Beacon代理。
```
// scripts/create-box.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const Box = await ethers.getContractFactory("Box");

  const beacon = await upgrades.deployBeacon(Box);
  await beacon.deployed();
  console.log("Beacon deployed to:", beacon.address);

  const box = await upgrades.deployBeaconProxy(beacon, Box, [42]);
  await box.deployed();
  console.log("Box deployed to:", box.address);
}

main();
```

然后，在另一个脚本中，您可以使用upgradeBeacon函数将Beacon升级到新版本。当Beacon升级时，所有指向它的Beacon代理将使用新的合约实现。

```
// scripts/upgrade-box.js
const { ethers, upgrades } = require("hardhat");

async function main() {
  const BoxV2 = await ethers.getContractFactory("BoxV2");

  await upgrades.upgradeBeacon(BEACON_ADDRESS, BoxV2);
  console.log("Beacon upgraded");

  const box = BoxV2.attach(BOX_ADDRESS);
}

main();
```

## 在测试中的使用
如果您想要为升级您的合约添加测试（这是您应该做的！），您还可以在Hardhat测试中使用插件的功能。API与脚本中的使用方式相同。

### 代理
```
const { expect } = require("chai");

describe("Box", function() {
  it('works', async () => {
    const Box = await ethers.getContractFactory("Box");
    const BoxV2 = await ethers.getContractFactory("BoxV2");

    const instance = await upgrades.deployProxy(Box, [42]);
    const upgraded = await upgrades.upgradeProxy(instance.address, BoxV2);

    const value = await upgraded.value();
    expect(value.toString()).to.equal('42');
  });
});
```

### Beacon代理
```
const { expect } = require("chai");

describe("Box", function() {
  it('works', async () => {
    const Box = await ethers.getContractFactory("Box");
    const BoxV2 = await ethers.getContractFactory("BoxV2");

    const beacon = await upgrades.deployBeacon(Box);
    const instance = await upgrades.deployBeaconProxy(beacon, Box, [42]);

    await upgrades.upgradeBeacon(beacon, BoxV2);
    const upgraded = BoxV2.attach(instance.address);

    const value = await upgraded.value();
    expect(value.toString()).to.equal('42');
  });
});
```
