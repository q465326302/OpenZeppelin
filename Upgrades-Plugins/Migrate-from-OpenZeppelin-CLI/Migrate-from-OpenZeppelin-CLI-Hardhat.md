# Migrate from OpenZeppelin CLI

> NOTE
这个指南是为了从一个已经被弃用的CLI迁移过来的。

这个指南将指导你如何将一个项目从OpenZeppelin CLI迁移到OpenZeppelin Upgrades插件，无论是用于Truffle还是Hardhat。

> NOTE
Truffle和Hardhat都有指导。使用这个切换开关选择你的偏好！
切换Hardhat或Truffle

> TIP
如果你想在一个样本OpenZeppelin CLI项目上试验这些指令，你可以克隆[OpenZeppelin/openzeppelin-upgrades-migration-example](https://github.com/OpenZeppelin/openzeppelin-upgrades-migration-example)，然后按照readme中的设置指令进行操作，然后再继续。

## Differences
CLI和插件之间的主要区别在于，前者曾经为你跟踪你的可升级（和不可升级）的部署。在某些上下文中，这很方便，因为你不需要过多地担心代理、实现或地址，你只需要关注像升级或发送交易到你的合约这样的操作，只需要通过它们的名字就可以了。

但是，让CLI跟踪你的部署的代价是限制了你与不同工具和工作流集成的自由。而且，由于插件被设计为独立于CLI工作，我们取消了这个限制，所以现在你有灵活性来跟踪你的代理，你认为最好的方式。

除此之外，其他一切都保持不变，因为CLI和插件都使用相同的已知的Proxy和ProxyAdmin合约，只是提供了两种不同的接口来管理它们。这意味着迁移你的项目不会触及链上的任何东西，一切都是安全和本地的。

## Installation
[安装Hardhat](https://hardhat.org/tutorial/creating-a-new-hardhat-project.html)并在初始化时选择Create an empty hardhat.config.js选项。

```
npm install --save-dev hardhat
npx hardhat
```
然后安装Upgrades插件：

```
npm install --save-dev @openzeppelin/hardhat-upgrades
npm install --save-dev @nomiclabs/hardhat-ethers ethers # peer dependencies
```

完成后，在Hardhat配置文件中注册插件，添加以下行：

```
// hardhat.config.js
require('@openzeppelin/hardhat-upgrades');

module.exports = {
  // ...
};
```

迁移CLI项目
这是一个单向过程。确保保留您的.openzeppelin/文件夹的备份或版本控制副本。
现在，让我们通过运行以下命令迁移我们的项目：

```
npx migrate-oz-cli-project
```

```
✔ Successfully migrated .openzeppelin/rinkeby.json
✔ Migration data exported to openzeppelin-cli-export.json
✔ Deleting .openzeppelin/project.json

These were your project's compiler options:
{
  "compilerSettings": {
    "optimizer": {
      "enabled": false,
      "runs": "200"
    }
  },
  "typechain": {
    "enabled": false
  },
  "manager": "openzeppelin",
  "solcVersion": "0.6.12",
  "artifactsDir": "build/contracts",
  "contractsDir": "contracts"
}
```

这个脚本是与插件一起安装的，它的作用是删除CLI项目文件，并将您的旧网络文件（所有这些文件都位于.openzeppelin目录下）转换为其Upgrades插件等效项。再次强调，链上没有任何改变，只有本地文件。请注意，一旦您运行了这个，除非通过备份或版本控制恢复更改，否则您将无法再使用CLI来管理这个项目的合约。

迁移脚本还会将一个openzeppelin-cli-export.json文件导出到您的工作目录，该文件包含CLI曾经为您管理的所有数据，现在您可以自由地使用它，无论您认为最好。这包括您的编译器设置，这些设置也会在迁移结束时打印出来，以方便使用。让我们将它们添加到我们的新项目配置中：

将编译器设置复制到Hardhat配置文件中的[solidity字段](https://hardhat.org/config/#available-config-options)

```
// hardhat.config.js

// ...

module.exports = {
  // ...
  solidity: {
    version: "0.6.12",
    settings: {
      optimizer: {
        enabled: false,
        runs: 200
      }
    }
  }
}
```

> NOTE
truffle-config.js和hardhat.config.js文件中的solidity编译器配置格式是不同的

就这样，您已经成功迁移了您的CLI项目。现在让我们尝试使用您的新设置升级您迁移的合约中的一个。

## Upgrade to a new version
假设我们在CLI项目中有一个Box合约，部署到Rinkeby网络。然后如果我们打开我们的导出文件，我们会看到类似这样的东西：

```
// openzeppelin-cli-export.json
{
  "networks": {
    "rinkeby": {
      "proxies": {
        "openzeppelin-upgrades-migration-example/Box": [
          {
            "address": "0x500D1d6A4c7D8Ae28240b47c8FCde034D827fD5e",
            "version": "1.0.0",
            "implementation": "0x038B86d9d8FAFdd0a02ebd1A476432877b0107C8",
            "admin": "0x1A1FEe7EeD918BD762173e4dc5EfDB8a78C924A8",
            "kind": "Upgradeable"
          }
        ]
      }
    }
  },
  "compiler": {
    // we'll ignore compiler settings for this
  }
}
```

我们在这里看到的是我们可升级合约的地址的JSON表示：

* address：代理地址（代理合约包含您的可升级合约状态）

* implementation：实现地址（您的可升级合约逻辑）

* admin：代理管理员的地址，除非您另行设置，否则可能属于ProxyAdmin合约

如果我们决定将我们的Box合约升级到BoxV2合约，使用插件和这个导出文件，这就是它的样子：

这些脚本只是如何使用导出数据的示例。我们对是否保留该文件的状态或如何处理其数据没有任何建议。这现在取决于用户。

使用Hardhat，我们会写一个脚本（你可以在[这里](https://hardhat.org/guides/scripts.html)阅读更多关于Hardhat脚本的信息，以及在[这里](../API-Reference/Hardhat-Upgrades.md)阅读更多关于使用hardhat-upgrades插件的信息）：
```
// scripts/upgradeBoxToV2.js

const { ethers, upgrades } = require("hardhat");
const OZ_SDK_EXPORT = require("../openzeppelin-cli-export.json");

async function main() {
  const [ Box ] = OZ_SDK_EXPORT.networks.rinkeby.proxies["openzeppelin-upgrades-migration-example/Box"];
  const BoxV2 = await ethers.getContractFactory("BoxV2");
  await upgrades.upgradeProxy(Box.address, BoxV2);
}

main();
```

```
npx hardhat run scripts/upgradeBoxToV2.js --network rinkeby
```
就这样！您已经将您的OpenZeppelin CLI项目迁移到了Truffle或Hardhat，并使用插件进行了升级。