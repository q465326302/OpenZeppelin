# Upgrades PluginsUpgrades Plugins

将升级集成到现有工作流程中。使用[Hardhat](https://hardhat.org/)和[Truffle](https://www.trufflesuite.com/truffle)的插件来部署和管理可升级的以太坊合约。

* 部署可升级的合约。
* 升级已部署的合约。
* 管理代理管理员权限。
* 在测试中轻松使用。

> TIP
升级插件只是部署和保护可升级智能合约的OpenZeppelin工具的一部分。查看完整的资源列表。

## 概述

## 安装

### 安装 Hardhat
```
npm install --save-dev @openzeppelin/hardhat-upgrades '@nomiclabs/hardhat-ethers@^2.0.0' 'ethers@^5.0.0'
```

这将安装我们的Hardhat插件以及必要的对等依赖项。

您还需要在Hardhat配置文件中加载它：
```
// hardhat.config.js
require('@openzeppelin/hardhat-upgrades');
```

```
// hardhat.config.ts
import '@openzeppelin/hardhat-upgrades';
```

### Truffle安装
```
npm install --save-dev @openzeppelin/truffle-upgrades
```

## 使用
请参阅使用[Truffle Upgrades](./API-Reference/Truffle-Upgrades.md)和[Hardhat Upgrades](./API-Reference/Hardhat-Upgrades.md)的文档，或查看下面的示例代码片段。

### Hardhat使用
Hardhat用户将能够编写使用该插件部署或升级合约的[脚本](https://hardhat.org/guides/scripts.html)，并管理代理管理员权限。

```
const { ethers, upgrades } = require("hardhat");

async function main() {
  // Deploying
  const Box = await ethers.getContractFactory("Box");
  const instance = await upgrades.deployProxy(Box, [42]);
  await instance.deployed();

  // Upgrading
  const BoxV2 = await ethers.getContractFactory("BoxV2");
  const upgraded = await upgrades.upgradeProxy(instance.address, BoxV2);
}

main();
```

Truffle的使用方式
Truffle用户可以编写使用插件部署或升级合约的[迁移脚本](https://www.trufflesuite.com/docs/truffle/getting-started/running-migrations)，或管理代理管理员权限。

```
const { deployProxy, upgradeProxy } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');
const BoxV2 = artifacts.require('BoxV2');

module.exports = async function (deployer) {
  const instance = await deployProxy(Box, [42], { deployer });
  const upgraded = await upgradeProxy(instance.address, BoxV2, { deployer });
}
```

### 使用测试
无论您使用Hardhat还是Truffle，您都可以在测试中使用插件来确保一切按预期工作。

```
it('works before and after upgrading', async function () {
  const instance = await upgrades.deployProxy(Box, [42]);
  assert.strictEqual(await instance.retrieve(), 42);

  await upgrades.upgradeProxy(instance.address, BoxV2);
  assert.strictEqual(await instance.retrieve(), 42);
});
```

## 插件的工作原理
这两个插件提供了管理可升级合约部署的功能。

例如，deployProxy执行以下操作：

1. 验证实现是否可以[升级](./Frequently-Asked-Questions.md#什么是合约的升级安全性)。
2. 为您的项目部署[代理管理员](./Frequently-Asked-Questions.md#代理管理员是什么)（如果需要）。
3. 部署[实现合约](./Frequently-Asked-Questions.md#什么是实现合约)。
4. 创建并初始化代理合约。

当您调用upgradeProxy时：
1. 验证新的实现是否可以[升级](./Frequently-Asked-Questions.md#什么是合约的升级安全性)并且与之前的版本[兼容](./Frequently-Asked-Questions.md#实现是兼容的意味着什么)。
2. 检查是否已部署了具有相同字节码的[实现合约](./Frequently-Asked-Questions.md#什么是实现合约)，如果没有，则部署一个新的实现合约。
3. 升级代理以使用新的实现合约。

插件将在项目根目录的.openzeppelin文件夹中跟踪您已部署的所有实现合约，以及代理管理员。您将在该文件夹中找到每个网络的一个文件。建议您将除了开发网络之外的所有网络的文件提交到源代码控制中（您可能会看到它们的文件名为.openzeppelin/unknown-*.json）。

    注意：.openzeppelin文件夹中的文件格式与OpenZeppelin CLI的文件格式不兼容。如果您想在现有的OpenZeppelin CLI项目中使用Upgrades插件，可以使用指南进行迁移。

## 代理模式
这些插件支持UUPS、透明和beacon代理模式。UUPS和透明代理可以分别升级，而任意数量的beacon代理可以通过升级它们指向的beacon同时进行原子升级。有关可用的不同代理模式的更多详细信息，请参阅*Proxies*的文档。

对于UUPS和透明代理，可以像上面显示的那样使用deployProxy和upgradeProxy。对于beacon代理，使用deployBeacon、deployBeaconProxy和upgradeBeacon。有关示例，请参阅*Hardhat Upgrades*和*Truffle Upgrades*的文档。

## 管理所有权
透明代理定义了一个具有升级权限的管理员地址。默认情况下，管理员是在幕后部署的*代理管理员合约*。您可以通过在插件中调用admin.changeProxyAdmin函数来更改代理的管理员。请注意，代理的管理员只能升级它，而不能与实现合约进行交互。有关此限制的更多信息，请阅读*透明代理和函数冲突的文档*。

代理管理员合约还定义了一个所有者地址，该地址具有操作它的权限。默认情况下，此地址是部署期间使用的外部拥有的账户。您可以通过在插件中调用admin.transferProxyAdminOwnership函数来更改代理管理员的所有者。请注意，更改代理管理员所有者实际上将升级项目中的任何代理的权力转移到新的所有者，因此请谨慎使用。有关管理员功能的更多详细信息，请参考每个插件的文档。

UUPS和beacon代理不使用管理员地址。UUPS代理依赖于重写*_authorizeUpgrade*函数来包含对升级机制的访问限制，而beacon代理只能由其对应的beacon的所有者进行升级。

一旦您将升级代理或beacon的权限转移给另一个地址，您仍然可以使用本地设置来验证和部署实现合约。插件包含一个prepareUpgrade函数，该函数将验证新的实现是否可以升级并且与之前的版本兼容，并使用您的本地以太坊账户部署它。然后，您可以从管理员或所有者地址执行升级本身。您还可以使用proposeUpgrade函数在*Defender Admin*中自动设置升级。