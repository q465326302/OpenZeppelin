# Overview
Nile是一个用于开发或与Cairo编写的StarkNet项目进行交互的命令行工具。它包含了开发、编译、测试和部署智能合约和dApp的不同组件，提供了执行任务的*命令行界面*和用于脚本编写的*运行时环境（NRE）*。

该软件包被设计为可扩展和可定制的，可以通过使用插件进行定制。本指南将带您完成我们推荐的安装设置，但由于我们预计插件将提供许多功能，您可以自由定制它。

## 快速入门
通过使用一个示例合约，我们将探索创建Nile项目的基础知识。我们将在本地进行测试，将一个账户和合约部署到开发网络节点，并通过账户向合约发送交易。

### 要求

#### GMP是GNU多精度算术库的缩写，它是一个用于高精度计算的库。
在安装Cairo之前，您需要安装gmp:
```
sudo apt install -y libgmp3-dev # linux
or
brew install gmp # mac
```
> TIP
如果您在Apple M1计算机上安装gmp时遇到任何问题，这里是一些可能的[解决方案列表](https://github.com/OpenZeppelin/nile/issues/22)。

#### 支持的Python版本
一些Nile依赖项具有特定的Python版本要求。因此，我们建议使用类似[pyenv](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/src/openzeppelin/access/ownable/library.cairo)的Python版本管理器和虚拟环境来避免冲突。

> IMPORTANT
目前支持的Python版本是>=3.9且<3.10。

### 安装
创建一个用于您的项目的文件夹并进入该文件夹:
```
mkdir myproject && cd myproject
```
创建一个虚拟环境并激活它:
```
python3 -m venv env
source env/bin/activate
```
安装Nile:
```
pip install cairo-nile
```
使用nile init快速设置您的开发环境:
```
nile init
```
```
🗄 Creating project directory tree
⛵️ Nile project ready! Try running:

nile compile
```

### 测试
nile init为您创建了一个示例的Cairo合约和测试。请查看contracts/contract.cairo和tests/test_contract.py以查看源代码。

运行pytest来运行智能合约的测试套件。
```
pytest tests/
```

> TIP
要了解有关并行性和覆盖率测试的更深入指南，请查阅我们的*测试指南*。

### 编译
默认情况下，使用nile compile命令编译contracts/文件夹中的合约。
```
nile compile
```
```
🤖 Compiling all Cairo contracts in the contracts directory
🔨 Compiling contracts/contract.cairo
✅ Done
```

> TIP
要查看Nile命令选项的完整参考，请查阅*CLI参考*部分。

### 与Starknet交互

#### 运行本地节点
默认情况下，如果您不使用--network选项指定网络，nile命令与starknet节点直接交互时会假设您正在使用本地节点（默认为localhost）。

使用nile node命令在开发环境中运行本地devnet节点。
```
nile node
```

#### 部署账户
1. 在.env文件中添加一个环境变量，其中包含账户的私钥。
```
# .env
ACCOUNT_1 = 0x1234
```
> CAUTION
不要在生产环境中使用此私钥！

使用nile counterfactual-address来预先计算给定签名者的账户合约的部署地址。
```
nile counterfactual-address ACCOUNT_1 --salt 123
```

> NOTE
不同的盐值会计算出不同的地址，即使使用相同的私钥。

3. 估计执行该交易的费用。
```
nile setup ACCOUNT_1 --salt 123 --estimate_fee
```
```
The estimated fee is: 52147932632842 WEI (0.000052 ETH).
Gas usage: 5063
Gas price: 10299808934 WEI
```
在生产环境中，您需要使用以太币为反事实地址提供资金，以支付部署费用，这在测试网中很快将成为强制性要求。目前正在实施多个桥接器和水龙头，如[Starkgate](https://goerli.starkgate.starknet.io/)，以支持开发者的体验。

对于使用开发网络节点进行本地开发，您可以使用预先部署的账户来为您的地址提供资金，使用*以下脚本*即可。

4. 在为预先计算的地址提供资金后，使用nile setup部署该账户。
```
nile setup ACCOUNT_1 --salt 123 --max_fee 52147932632842
```
> NOTE
如果未设置此选项，Nile将默认估算max_fee。对于涉及交易执行的其他命令，如declare、send或deploy，也适用此规则。
> IMPORTANT
在StarkNet中，用户只能部署先前声明的合约。[Openzeppelin Account](https://github.com/OpenZeppelin/cairo-contracts/blob/main/src/openzeppelin/account/presets/Account.cairo)会在每次发布devnet、testnets和mainnet后不久被声明，但是账户更新发布和devnet发布之间可能存在不同步的时间段。然后，您可以通过脚本使用预部署的*账户声明OZ Account*。

#### 检查Declare the OZ Account脚本。

1. 部署合约
首先，声明合约以在网络上注册类哈希。只要合约的字节码不变，您只需要执行一次此操作。
```
nile declare ACCOUNT_1 contract
```

2. 使用Nile部署合约。
```
nile deploy ACCOUNT_1 contract --alias my_contract
```
> NOTE
别名选项允许您在不使用地址的情况下与合约进行交互。

#### 从合约中读取数据
使用nile call命令来读取合约的视图函数。
```
nile call my_contract get_balance
```

#### 写给合同
使用Nile发送来执行交易。
```
nile send ACCOUNT_1 my_contract increase_balance 2
```