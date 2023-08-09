# 开发智能合约
欢迎来到智能合约开发的激动人心的世界！本指南将让您开始编写Solidity合约，以下是内容：

* 设置Solidity项目
 
* 编译Solidity源代码

* 添加更多合约

* 使用OpenZeppelin合约
  
## 关于Solidity

在本指南中，我们不会涉及诸如语法或关键字等语言概念。对此，您可以查看以下精选内容，其中包含适合新手和经验丰富的开发人员的优秀学习资源：

* 如果您想了解以太坊和智能合约的一般概述，[官方网站](https://ethereum.org/learn/)上有一个“了解以太坊”部分，其中包含许多适合初学者的内容。

* 如果您是新手，请将[官方Solidity文档](https://solidity.readthedocs.io/en/latest/introduction-to-smart-contracts.html)作为有用的资源。请查看他们的[安全建议](https://solidity.readthedocs.io/en/latest/security-considerations.html)，这很好地介绍了区块链和传统软件平台之间的区别。

* Consensys的[最佳实践](https://consensys.github.io/smart-contract-best-practices/)非常广泛，包括可供学习的[已证实模式](https://consensys.github.io/smart-contract-best-practices/development-recommendations/)和应避免的[已知陷阱](https://consensys.github.io/smart-contract-best-practices/attacks/)。

* 基于Web的[Ethernaut](https://ethernaut.openzeppelin.com/)项目将让您在逐步提高难度的级别中寻找智能合约中微妙的漏洞。

有了这些基础知识，让我们开始吧！

## 设置项目
创建项目后的第一步是[安装开发工具](../Setting%20up%20a%20Node%20project/Setting%20up%20a%20Node%20project.md)。

以太坊最流行的开发框架是[Hardhat](https://hardhat.org/)，并且我们使用[ethers.js](https://docs.ethers.io/)介绍其最常见用法。其次最流行的是[Truffle](https://www.trufflesuite.com/truffle)，它使用[web3.js](https://web3js.readthedocs.io/)。每个工具都有其优点，熟练掌握所有工具是很有用的。

在这些指南中，我们将展示如何使用Truffle和Hardhat开发、测试和部署智能合约。

> NOTE
Truffle和Hardhat都提供了说明。使用此切换选择您的首选项！

为了开始使用Truffle，我们将在[项目目录](../Setting%20up%20a%20Node%20project/Setting%20up%20a%20Node%20project.md)中安装它。
```
npm install --save-dev truffle
```
安装完成后，我们可以初始化Truffle。这将在我们的项目目录中创建一个空的Truffle项目。
```
npx truffle init

Starting init...
================

> Copying project files to /home/openzeppelin/learn

Init successful, sweet!
```



## 第一份合约
我们将我们的 Solidity 源文件 (.sol) 存储在一个名为 contracts 的目录中。这相当于你可能熟悉的其他语言中的 src 目录。

现在，我们可以编写我们的第一个简单的智能合约，称为 Box：它将允许人们存储一个值，该值可以稍后检索。

我们将把这个文件保存为 contracts/Box.sol。每个 .sol 文件应该有一个单独的合约代码，并以其命名。
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

## Solidity的编译

以太坊虚拟机（EVM）无法直接执行Solidity代码：我们需要先将其编译为EVM字节码。

我们的Box.sol合约使用Solidity 0.8，因此我们需要先[配置Truffle使用适当的solc版本](https://truffleframework.com/docs/truffle/reference/configuration#compiler-configuration)。

我们在truffle-config.js中指定了Solidity 0.8 solc版本。
```
// truffle-config.js
  ...

  // Configure your compilers
  compilers: {
    solc: {
      version: "0.8.4",    // Fetch exact version from solc-bin (default: truffle's version)
      // docker: true,        // Use "0.5.1" you've installed locally with docker (default: false)
      // settings: {          // See the solidity docs for advice about optimization and evmVersion
      //  optimizer: {
      //    enabled: false,
      //    runs: 200
      //  },
      //  evmVersion: "byzantium"
      // }
    }
  },

  ...
```
编译可以通过运行一个单独的编译命令来实现：

> NOTE
如果您不熟悉npx命令，请查看我们的[Node项目设置指南](../Setting%20up%20a%20Node%20project/Setting%20up%20a%20Node%20project.md)。
```
npx truffle compile

Compiling your contracts...
===========================
✔ Fetching solc version list from solc-bin. Attempt #1
✔ Downloading compiler. Attempt #1.
> Compiling ./contracts/Box.sol
> Compiling ./contracts/Migrations.sol
> Artifacts written to /home/openzeppelin/learn/build/contracts
> Compiled successfully using:
   - solc: 0.8.4+commit.c7e474f2.Emscripten.clang
```
内置的[编译](https://www.trufflesuite.com/docs/truffle/reference/truffle-commands#compile)命令将自动查找contracts目录中的所有合约，并使用[truffle-config.js](https://www.trufflesuite.com/docs/truffle/reference/configuration#compiler-configuration)中的配置使用Solidity编译器将它们编译。

您会注意到创建了一个build/contracts目录：它保存了编译后的工件（字节码和元数据），这些是.json文件。将此目录添加到您的.gitignore是一个好主意。

## 添加更多合约
随着您的项目增长，您将开始创建更多相互交互的合约：每个合约应存储在自己的.sol文件中。

为了看看这是什么样子，让我们向我们的Box合约添加一个简单的访问控制系统：我们将在一个名为Auth的合约中存储管理员地址，并只允许Auth允许的帐户使用Box。

因为编译器将捕捉contracts目录和子目录中的所有文件，所以您可以按照自己的意愿组织代码。在这里，我们将Auth合约存储在access-control子目录中：
```
// contracts/access-control/Auth.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Auth {
    address private _administrator;

    constructor(address deployer) {
        // Make the deployer of the contract the administrator
        _administrator = deployer;
    }

    function isAdministrator(address user) public view returns (bool) {
        return user == _administrator;
    }
}
```

要使用来自Box的此合约，我们使用一个导入语句，通过其相对路径引用Auth：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Auth from the access-control subdirectory
import "./access-control/Auth.sol";

contract Box {
    uint256 private _value;
    Auth private _auth;

    event ValueChanged(uint256 value);

    constructor() {
        _auth = new Auth(msg.sender);
    }

    function store(uint256 value) public {
        // Require that the caller is registered as an administrator in Auth
        require(_auth.isAdministrator(msg.sender), "Unauthorized");

        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```
将关注点分离到多个合约中是保持每个合约简单的好方法，通常是一种良好的实践。

然而，这并不是将代码分割成模块的唯一方法。您还可以使用继承来封装和重用 Solidity 中的代码，接下来我们将看到。

## 使用OpenZeppelin Contracts
可重用的模块和库是优秀软件的基石。[OpenZeppelin Contracts](../../Contracts/Contracts.4.x/Overview.md) 包含许多有用的智能合约构建模块。当你在这些模块上构建时，你可以放心：它们经过多次审计，其安全性和正确性经过了实战检验。

### 关于继承
库中的许多合约不是独立的，也就是说，你不需要将它们原封不动地部署。相反，你将使用它们作为起点，通过添加功能来构建自己的合约。Solidity 提供了多重继承作为实现此目的的机制：请查看 [Solidity 文档](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)以获取更多详细信息。

例如，_Ownable_ 合约将部署者帐户标记为合约的所有者，并提供了一个名为 onlyOwner 的修改器。当应用于函数时，onlyOwner 会导致所有不是由所有者帐户发起的函数调用失败。还提供了_transfer_ 和 _renounce_ 所有权的函数。

使用这种方式，继承成为一种强大的机制，允许进行模块化，而不需要强制部署和管理多个合约。

### 导入OpenZeppelin Contracts
最新发布的OpenZeppelin Contracts库可以通过运行以下命令下载：
```
npm install @openzeppelin/contracts
```
> NOTE
你应该始终使用这些已发布的版本库：将库源代码复制粘贴到你的项目中是一种危险的做法，这样很容易在你的合约中引入安全漏洞。

要使用OpenZeppelin Contracts之一，请在其路径前缀中加上@openzeppelin/contracts进行导入。例如，为了替换我们自己的_Auth_合约，我们将导入@openzeppelin/contracts/access/Ownable.sol以向Box添加访问控制。
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Ownable from the OpenZeppelin Contracts library
import "@openzeppelin/contracts/access/Ownable.sol";

// Make Box inherit from the Ownable contract
contract Box is Ownable {
    uint256 private _value;

    event ValueChanged(uint256 value);

    // The onlyOwner modifier restricts who can call the store function
    function store(uint256 value) public onlyOwner {
        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```
[OpenZeppelin合约文档](../../Contracts/Contracts.4.x/Overview.md)是学习开发安全智能合约系统的绝佳资源。它包括指南和详细的API参考：例如，可以参考访问[控制指南](../../Contracts/Contracts.4.x/Access%20Control.md)了解在上面的代码示例中使用的Ownable合约。

## 下一步
编写和编译Solidity合约只是在以太坊网络上运行您的去中心化应用程序的旅程中的第一步。一旦您熟悉了这个设置，您将想要转向更高级的任务：

* [部署和交互](../Deploying%20and%20interacting/Deploying%20and%20interacting-truffle.md)

* [编写自动化测试](../Writing%20automated%20tests/Writing%20automated%20smart%20contract%20tests-truffle.md)

* [连接到公共测试网络](../Connecting%20to%20public%20test%20networks/Connecting%20to%20public%20test%20networks-truffle.md)