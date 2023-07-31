# 升级智能合约
使用*OpenZeppelin Upgrades插件*部署的智能合约可以进行**升级**，修改其代码，同时保留其地址、状态和余额。这使您能够逐步为项目添加新功能，或修复在*生产*中发现的任何错误。

在本指南中，我们将学习：

* 为什么升级很重要

* 使用Upgrades插件升级我们的Box

* 学习升级在幕后是如何工作的

* 学习如何编写可升级的合约

> NOTE
有关Truffle和Hardhat的说明都可用。使用此切换选择您的首选项！

## 升级包含什么内容
以太坊中的智能合约默认情况下是不可变的。一旦创建，就没有办法修改它们，有效地成为参与者之间不可打破的合约。

然而，对于某些情况，修改它们是可取的。想象一下两个当事方之间的传统合约：如果他们都同意更改它，他们将能够这样做。在以太坊上，他们可能希望修改智能合约以修复他们发现的错误（这甚至可能导致黑客窃取他们的资金！），添加其他功能，或仅仅是更改其执行的规则。

要修复无法升级的合约中的错误，您需要执行以下操作：

1. 部署合约的新版本

2. 手动迁移从旧合约到新合约的所有状态（这可能非常昂贵，涉及燃气费用！）

3. 更新与旧合约交互的所有合约以使用新部署的地址

4. 联系所有用户并说服他们开始使用新部署（并处理同时使用两个合约，因为用户迁移速度较慢）

为了避免经历这种混乱，我们已经将合约升级直接构建到我们的插件中。这允许我们**更改合约代码，同时保留状态、余额和地址**。让我们看看它的实际应用。

## 使用升级插件进行升级
每当您使用*OpenZeppelin Upgrades插件*中的deployProxy部署新合约时，该合约实例可以稍后**升级**。默认情况下，只有最初部署合约的地址才有权升级它。

deployProxy将创建以下交易：

1. 部署实现合约（我们的Box合约）

2. 部署ProxyAdmin合约（我们代理的管理者）。

3. 部署代理合约并运行任何初始化函数。

让我们看看它是如何工作的，通过使用与*我们之前部署*时相同的设置，部署可升级版本的Box合约：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```
我们首先需要安装Upgrades插件。

安装*Hardhat Upgrades*插件。
```
npm install --save-dev @openzeppelin/hardhat-upgrades
```
我们需要配置Hardhat以使用@openzeppelin/hardhat-upgrades插件。为此，请在您的hardhat.config.js文件中添加插件，如下所示。
```
// hardhat.config.js
...
require('@nomiclabs/hardhat-ethers');
require('@openzeppelin/hardhat-upgrades');
...
module.exports = {
...
};
```
为了升级像 Box 这样的合约，我们需要先将其部署为可升级合约，这是与我们之前看到的不同的部署流程。我们将通过调用 store 方法并传入值为 42 来初始化我们的 Box 合约。

Hardhat 目前没有本地的部署系统，因此我们使用[脚本](https://hardhat.org/guides/deploying.html)来部署合约。

我们将创建一个脚本，使用 [deployProxy](https://hardhat.org/guides/deploying.html) 部署我们的可升级 Box 合约。我们将把这个文件保存为 scripts/deploy_upgradeable_box.js。
```
// scripts/deploy_upgradeable_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await upgrades.deployProxy(Box, [42], { initializer: 'store' });
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main();
```
然后，我们可以部署可升级的合约。

使用run命令，我们可以将Box合约部署到开发网络。
```
npx hardhat run --network localhost scripts/deploy_upgradeable_box.js
Deploying Box...
Box deployed to: 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0
```
然后，我们可以与我们的 Box 合约进行交互，以检索在初始化期间存储的值。

我们将使用 [Hardhat 控制台](https://hardhat.org/guides/hardhat-console.html)与我们升级后的 Box 合约进行交互。

我们需要指定我们部署 Box 合约时代理合约的地址。
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0');
undefined
> (await box.retrieve()).toString();
'42'
```

为了举例，假设我们想添加一个新功能：一个函数，可以将存储在新版本的Box中的值递增。
```
// contracts/BoxV2.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BoxV2 {
    // ... code from Box.sol

    // Increments the stored value by 1
    function increment() public {
        _value = _value + 1;
        emit ValueChanged(_value);
    }
}
```
创建 Solidity 文件后，我们现在可以使用 upgradeProxy 函数升级我们之前部署的实例。

upgradeProxy 将创建以下事务：

1. 部署实现合约（我们的 BoxV2 合约）

2. 调用 ProxyAdmin 更新代理合约以使用新的实现。

我们将创建一个脚本，使用upgradeProxy将我们的Box合约升级为BoxV2。我们将把这个文件保存为scripts/upgrade_box.js。我们需要指定我们部署Box合约时代理合约的地址。
```
// scripts/upgrade_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const BoxV2 = await ethers.getContractFactory('BoxV2');
  console.log('Upgrading Box...');
  await upgrades.upgradeProxy('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0', BoxV2);
  console.log('Box upgraded');
}

main();
```
我们可以部署可升级的合约。

使用运行命令，我们可以在开发网络上升级Box合约。

```
npx hardhat run --network localhost scripts/upgrade_box.js
Compiling 1 file with 0.8.4
Compilation finished successfully
Upgrading Box...
Box upgraded
```
完成了！我们的Box实例已经升级到最新版本的代码，**同时保持其状态和之前相同的地址**。我们不需要在新地址部署新的Box，也不需要手动将值从旧的Box复制到新的Box。

让我们尝试一下调用新的增量函数，并在此之后检查值：

我们需要指定我们部署Box合约时代理合约的地址。

```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const BoxV2 = await ethers.getContractFactory('BoxV2');
undefined
> const box = await BoxV2.attach('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0');
undefined
> await box.increment();
...
> (await box.retrieve()).toString();
'43'
```
就是这样！请注意，无论您是在本地区块链、测试网络还是主网络上工作，Box 的价值和地址都在升级过程中得到保留。

让我们看看 *OpenZeppelin Upgrades 插件*是如何实现这一点的。

## 升级是如何工作的
本节内容将比其他部分更加理论化，如果您感到不感兴趣，可以跳过，稍后再回来查看。

当您创建一个新的可升级合约实例时，OpenZeppelin Upgrades插件实际上会部署三个合约：

1. 您编写的合约，也就是包含逻辑的实现合约。

2. 一个ProxyAdmin来作为代理的管理员。

3. 一个代理合约指向实现合约，这是您实际上与之交互的合约。

在这里，代理是一个简单的合约，只是将所有调用委托给一个实现合约。委托调用类似于普通调用，只是所有代码都在调用者的上下文中执行，而不是被调用者的上下文中执行。因此，实现合约中的转账实际上将转移代理的余额，对合约存储的任何读写都将从代理自己的存储中读取或写入。

这使我们能够将合约的状态和代码分离：代理保存状态，而实现合约提供代码。它还使我们能够通过让代理委托给不同的实现合约来更改代码。

升级涉及以下步骤：

1. 部署新的实现合约。

2. 向代理发送交易，将其实现地址更新为新的地址。

> NOTE
您可以使用相同的实现合约拥有多个代理，因此如果您计划部署多个相同合约的副本，使用此模式可以节省 gas。

任何智能合约的用户都始终与代理交互，代理永远不会更改其地址。这样可以在不要求用户做任何更改的情况下，推出升级或修复错误-他们只需像往常一样与相同的地址交互。

> NOTE
如果您想了解更多关于OpenZeppelin代理如何工作的信息，请查看*代理*。

## 合约升级的限制
虽然任何智能合约都可以被设计成可升级的，但是 Solidity 语言的一些限制需要加以解决。这些限制在编写合约的初始版本和升级版本时都会出现。

### 初始化
可升级合约不能有构造函数。为了帮助您运行初始化代码，*OpenZeppelin Contracts*提供了*Initializable*基础合约，允许您将方法标记为*初始化机器*，确保它只能运行一次。

例如，让我们编写一个带有初始化器的Box合约的新版本，将一个管理员的地址存储在其中，只允许该管理员更改其内容。
```
// contracts/AdminBox.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract AdminBox is Initializable {
    uint256 private _value;
    address private _admin;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    function initialize(address admin) public initializer {
        _admin = admin;
    }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}

    // Stores a new value in the contract
    function store(uint256 value) public {
        require(msg.sender == _admin, "AdminBox: not admin");
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```
在部署此合约时，我们需要指定初始化函数名称（仅当名称不是默认的initialize时），并提供我们想要使用的管理员地址。
```
// scripts/deploy_upgradeable_adminbox.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const AdminBox = await ethers.getContractFactory('AdminBox');
  console.log('Deploying AdminBox...');
  const adminBox = await upgrades.deployProxy(AdminBox, ['0xACa94ef8bD5ffEE41947b4585a84BdA5a3d3DA6E'], { initializer: 'initialize' });
  await adminBox.deployed();
  console.log('AdminBox deployed to:', adminBox.address);
}

main();
```
就实际目的而言，初始化器的作用类似于构造函数。然而，请记住，由于它是一个常规函数，您需要手动调用所有基础合约（如果有的话）的初始化器。

您可能已经注意到，我们包括了一个构造函数和一个初始化器。这个构造函数的目的是将实现合约保持在初始化状态，这是对某些潜在攻击的缓解。

要了解更多关于编写可升级合约时的注意事项以及其他信息，请查看我们的*编写可升级合约指南*。


### 升级
由于技术限制，当您将合约升级到新版本时，您无法更改该合约的**存储布局**。

这意味着，如果您已经在合约中声明了一个状态变量，您无法删除它，更改其类型或在其之前声明另一个变量。在我们的 Box 示例中，这意味着我们只能在value之后添加新的状态变量。
```
// contracts/Box.sol
contract Box {
    uint256 private _value;

    // We can safely add a new variable after the ones we had declared
    address private _owner;

    // ...
}
```
幸运的是，这个限制只影响状态变量。您可以随意更改合约的函数和事件。

> NOTE
如果你意外地搞乱了合约的存储布局，升级插件将在你尝试升级时发出警告。

要了解更多关于这个限制的信息，请转到“修改您的合约”指南。

## 测试
为了测试可升级合约，我们应该为实现合约创建单元测试，并创建更高级别的测试，以测试通过代理进行交互。我们可以在测试中使用deployProxy，就像我们在部署时一样。

当我们想要升级时，我们应该为新的实现合约创建单元测试，并创建更高级别的测试，以测试使用upgradeProxy进行升级后通过代理进行交互，并检查状态是否在升级后得以维护。

## 可能存在的问题
在学习如何升级合约时，您可能会发现自己处于本地环境中的冲突合约情况。为了解决这个问题，请考虑使用以下步骤：

停止使用npx hardhat node运行的节点ctrl+C。执行清理命令：npx hardhat clean。

## 下一步
现在您已经知道如何升级智能合约，并可以迭代开发项目，是时候将项目带到*测试网*和*生产环境*了！您可以放心，如果出现错误，您有工具来修改合约并改变它。
