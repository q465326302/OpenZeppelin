# Setting up a Node project
新的软件行业通常从每个项目共享相同的技术栈开始。以太坊生态系统也不例外，它语言选择是[JavaScript](https://en.wikipedia.org/wiki/JavaScript)。包括OpenZeppelin软件在内的所有最常用的以太坊库都是用JavaScript或其变体编写的。

传统上，JavaScript代码在网页浏览器上作为网站的一部分运行，但也可以使用[Node](https://nodejs.org/)作为独立进程执行。

本指南将帮助你设置Node开发环境，这是使用OpenZeppelin工具和其他第三方产品的必要步骤。

> TIP
如果你已经熟悉Node、npm和Git，可以选择跳过本指南！

## 安装Node
有多种方法可以在你的计算机上获取Node：你可以通过[软件包管理器](https://nodejs.org/en/download/package-manager/)获取它，也可以[直接下载安装程序](https://nodejs.org/en/download/)。

> TIP
如果你正在运行Windows，建议尽可能使用[Windows子系统来运行Linux](https://docs.microsoft.com/en-us/windows/nodejs/setup-on-wsl2)，因为许多生态系统都是为Linux编写的。

完成后，在终端上运行 node --version 检查你的安装：14.x或16.x系列的任何版本都应与大多数以太坊软件兼容。
```
node --version
v16.17.1
```

## 创建项目
JavaScript软件通常以包的形式捆绑在一起，这些包通过[npm注册表](https://www.npmjs.com/)分发。一个包只是一个包含名为package.json的文件的目录，描述了包的名称、版本、内容等。当你构建自己的项目时，即使你不打算分发，你也将创建一个包。

所有Node安装都包含一个用于npm注册表的命令行客户端，你将在开发自己的项目时使用它。要启动一个新项目，请创建一个目录：
```
mkdir learn && cd learn
```
然后我们可以进行初始化：
```
npm init -y
```
就是这样简单！你新创建的 package.json 文件将随着项目的发展而发展，例如当使用 npm install 安装依赖项时。

> TIP
JavaScript和npm是世界上最常用的软件工具之一：如果你有疑问，你会在网上找到大量关于它们的信息。

### 使用 npx
npm 注册表中有两种广义上的软件包：库和可执行文件。安装的库可以像任何其他 JavaScript 代码一样使用，但可执行文件是特殊的。

在安装 node 时，第三个二进制文件是[ npx](https://blog.npmjs.org/post/162869356040/introducing-npx-an-npm-package-runner)。它用于在项目中本地安装的可执行文件中运行。

虽然[ Truffle ](https://www.trufflesuite.com/truffle)和[ Hardhat ](https://hardhat.org/)可以全局安装，但我们建议在每个项目中本地安装，以便你可以按项目控制版本。

为了清晰起见，在我们的指南中，我们将显示包括 npx 的完整命令，以免由于二进制文件不在系统路径中而出现错误。
```
truffle init
truffle: command not found
npx truffle init
- Fetching solc version list from solc-bin. Attempt #1

Starting init...
================

> Copying project files to ...

Init successful, sweet!
```

> WARNING
在运行 npx 命令时，请确保你在项目目录中！否则，它将再次下载完整的可执行文件，仅仅为了运行该命令，这在大多数情况下都不是你想要的。

## 使用版本控制跟踪
在开始编码之前，你应该为项目添加[版本控制软件](https://en.wikipedia.org/wiki/Version_control)来跟踪更改。

到目前为止，最常用的工具是[Git](https://git-scm.com/)，通常与[GitHub](https://github.com/)一起使用以进行托管。事实上，你将在我们的[GitHub存储库](https://github.com/OpenZeppelin)中找到所有OpenZeppelin软件的完整源代码和历史记录。

> TIP
如果你以前从未使用过Git，一个好的起点是[Git手册](https://guides.github.com/introduction/git-handbook/)。

> WARNING
不要将助记词、私钥和API密钥等 secrets 信息提交到版本控制！确保在[.gitignore](https://git-scm.com/docs/gitignore)文件中忽略包含 secrets 信息的文件。