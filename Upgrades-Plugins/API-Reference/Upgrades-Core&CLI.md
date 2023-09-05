# OpenZeppelin Upgrades Core & CLI
@openzeppelin/upgrades-core软件包提供了一个validate命令，用于检查可升级合约的升级安全性和存储布局兼容性。它可以在开发过程中使用，以确保您的合约是升级安全的并且与先前版本兼容。

它还提供了编程接口来以程序化的方式执行这些检查，并包含了这些检查的核心逻辑，以便与OpenZeppelin Upgrades插件一起执行这些检查。

## Validate命令
从包含构建信息文件的目录中检测可升级合约，并验证它们是否是升级安全的。如果您想要从命令行、脚本或作为CI/CD流程的一部分验证项目中的所有可升级合约，请使用此命令。

> TIP
"构建信息文件"由您的编译工具链（Hardhat、Foundry）生成，包含编译过程的输入和输出。

### 先决条件
在使用validate命令之前，您必须定义可升级合约，以便可以检测和验证它们，定义存储布局比较的参考合约，并编译您的合约。

#### 定义可升级合约
validate命令对看起来像可升级合约的合约执行升级安全性检查。具体来说，它对满足以下任一条件的实现合约执行检查：

* 继承[Initializable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/proxy/utils/Initializable.sol)。

* 具有upgradeTo(address)函数。这适用于继承[UUPSUpgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/proxy/utils/UUPSUpgradeable.sol)的合约。

* 具有NatSpec注释@custom:oz-upgrades。

* 根据下面的[定义参考合约](#先决条件)，具有NatSpec注释@custom:oz-upgrades-from <reference>。

> TIP
只需向每个实现合约添加NatSpec注释@custom:oz-upgrades或@custom:oz-upgrades-from <reference>，以便可以将其检测为用于验证的可升级合约。

#### 定义参考合约

> IMPORTANT
如果一个实现合约是作为对现有代理的升级部署的，您**必须**定义一个参考合约，用于进行存储布局比较。否则，如果存在任何存储布局不兼容性，您将不会收到错误。

通过向您的实现合约添加NatSpec注释@custom:oz-upgrades-from <reference>来定义参考合约，其中<reference>是用于存储布局比较的参考合约的合约名称或完全限定合约名称。合约不需要在范围内，如果在项目中是明确的，则合约名称就足够了。

示例：
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/// @custom:oz-upgrades-from MyContractV1
contract MyContractV2 {
  ...
}
```

或：
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

/// @custom:oz-upgrades-from contracts/MyContract.sol:MyContractV1
contract MyContractV2 {
  ...
}
```

#### 编译具有存储布局的合约
编译您的合约，并确保您的构建已配置为在构建信息目录中输出带有Solidity编译器输入和输出的JSON文件。编译器输出必须包括存储布局。如果存在任何先前的构建工件，则必须首先清理它们以避免重复的合约定义。

#### Hardhat
配置 hardhat.config.js 或 hardhat.config.ts 以在输出选择中包含存储布局：
```
module.exports = {
  solidity: {
    settings: {
      outputSelection: {
        '*': {
          '*': ['storageLayout'],
        },
      },
    },
  },
};
```

然后编译你的合约。
```
npx hardhat clean && npx hardhat compile
```

#### Foundry
配置foundry.toml以包括构建信息和存储布局:
```
[profile.default]
build_info = true
extra_output = ["storageLayout"]
```

然后编译您的合约。
```
forge clean && forge build
```

### Usage
在执行先决条件之后，运行npx @openzeppelin/upgrades-core validate命令来验证你的合约。
```
npx @openzeppelin/upgrades-core validate [<BUILD_INFO_DIR>] [<OPTIONS>]
```

如果发现任何错误，命令将以非零退出代码退出，并将错误的详细报告打印到控制台。

**参数：**

* <BUILD_INFO_DIR> - 可选的构建信息目录的路径，该目录包含包含Solidity编译器输入和输出的JSON文件。对于Hardhat项目，默认为artifacts/build-info，对于Foundry项目，默认为out/build-info。如果您的项目使用自定义输出目录，您必须在此处指定其构建信息目录。

**选项：**

* unsafeAllow "<VALIDATION_ERRORS>" - 选择性地禁用一个或多个验证错误。逗号分隔的列表，可以包含以下一个或多个：state-variable-assignment，state-variable-immutable，external-library-linking，struct-definition，enum-definition，constructor，delegatecall，selfdestruct，missing-public-upgradeto

* unsafeAllowRenames - 配置存储布局检查以允许变量重命名。

* unsafeSkipStorageCheck - 跳过存储布局兼容性错误的检查。这是一种危险的选项，应作为最后的手段使用。

## 高级API
高级API是[validate命令](#validate命令)的程序化等效物。如果您想从JavaScript或TypeScript环境中验证项目的所有可升级合约，请使用此API。

### 先决条件
与[validate命令](#validate命令)相同的先决条件。

### 用法
导入validateUpgradeSafety函数：
```
import { validateUpgradeSafety } from '@openzeppelin/upgrades-core';
```

然后调用该函数来验证您的合约，并获取一个包含验证结果的项目报告。

#### validateUpgradeSafety
```
validateUpgradeSafety(
  buildInfoDir?: string,
  contract?: string,
  reference?: string,
  opts: ValidateUpgradeSafetyOptions = {},
): Promise<ProjectReport>
```

检测来自构建信息目录的可升级合约，并验证它们是否可以安全升级。返回一个包含结果的[项目报告](#projectreport)。

注意，此函数不直接抛出验证错误。相反，您必须使用项目报告来确定是否发现了任何错误。

**参数：**

* buildInfoDir - 指向包含Solidity编译器输入和输出的JSON文件的构建信息目录的路径。对于Hardhat项目，默认为artifacts/build-info，对于Foundry项目，默认为out/build-info。如果您的项目使用自定义输出目录，则必须在此处指定其构建信息目录。

* contract - 要验证的合约的名称或完全限定名称。如果未指定，则将验证构建信息目录中的所有可升级合约。

* reference - 用于存储布局比较的参考合约的名称或完全限定名称。只能与合约一起使用。如果未指定，则使用正在验证的合约中的@custom:oz-upgrades-from注解。

* opts - 一个对象，其中定义了以下选项，如[Common Options](./Hardhat-Upgrades.md#常见选项)中所定义：

  * unsafeAllow

  * unsafeAllowRenames

  * unsafeSkipStorageCheck

**返回：**

一个[项目报告](#projectreport)。

#### ProjectReport
```
interface ProjectReport {
  ok: boolean;
  explain(color?: boolean): string;
  numPassed: number;
  numTotal: number;
}
```

一个表示升级安全检查和存储布局比较结果的对象，并包含所有找到的错误的报告。

**成员：**

* ok - 如果找到任何错误，则为false，否则为true。

* explain() - 如果有错误，返回详细解释错误的消息。

* numPassed - 通过升级安全检查的合约数量。

* numTotal - 检测到的可升级合约的总数。

## 低级API
低级API使用[Solidity输入和输出的JSON对象](https://docs.soliditylang.org/en/latest/using-the-compiler.html#compiler-input-and-output-json-description)，可以对单个合约执行升级安全检查和存储布局比较。如果您想验证特定的合约而不是整个项目，请使用此API。

### 先决条件
编译您的合约以生成Solidity输入和输出的JSON对象。编译器的输出必须包括存储布局。

请注意，低级API不自动检测可升级合约，因此不需要[验证命令](#validate命令)的其他先决条件。相反，您必须为每个要验证的实现合约创建UpgradeableContract的实例，并调用其函数以获取升级安全性和存储布局报告。

### 用法
导入UpgradeableContract类：
```
import { UpgradeableContract } from '@openzeppelin/upgrades-core';
```

然后为您想要验证的每个实现合约创建一个UpgradeableContract的实例，并调用.getErrorReport()和/或.getStorageLayoutReport()方法来获取升级安全性和存储布局报告。

#### UpgradeableContract
这个类表示可升级合约的实现，并提供访问错误报告的功能。

#### constructor UpgradeableContract
构造函数 UpgradeableContract
```
constructor UpgradeableContract(
  name: string,
  solcInput: SolcInput,
  solcOutput: SolcOutput,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): UpgradeableContract
```

创建UpgradeableContract的新实例。

**参数：**

* name - 实现合约的名称，可以是完全限定名称或合约名称。如果多个合约具有相同的名称，则必须使用完全限定名称，例如contracts/Bar.sol:Bar。

* solcInput - 实现合约的Solidity输入JSON对象。

* solcOutput - 实现合约的Solidity输出JSON对象。

* opts - 一个带有以下选项的对象，如[Common Options](./Hardhat-Upgrades.md#常见选项)中定义的那样：

    * kind

    * unsafeAllow

    * unsafeAllowRenames

    * unsafeSkipStorageCheck

> TIP
在Hardhat中，solcInput和solcOutput可以从Build Info文件中获取，该文件本身可以使用hre.artifacts.getBuildInfo来检索。

#### .getErrorReport
```
getErrorReport(): Report
```

**返回：**

关于代理合约错误的报告，例如selfdestruct的使用。

#### .getStorageUpgradeReport
```
getStorageUpgradeReport(
  upgradedContract: UpgradeableContract,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Report
```

比较可升级合约的存储布局与提议升级的存储布局。

**参数：**

* upgradedContract - 另一个表示提议升级的UpgradeableContract实例。

* opts - 一个包含以下选项的对象，如[Common Options](./Hardhat-Upgrades.md#常见选项)中定义的：

    * kind

    * unsafeAllow

    * unsafeAllowRenames

    * unsafeSkipStorageCheck

**返回：**

* 关于代理合约错误的报告，例如使用selfdestruct和存储布局冲突的报告。

#### Report
```
interface Report {
  ok: boolean;
  explain(color?: boolean): string;
}
```
一个表示分析结果的对象。

**成员：**

* ok - 如果发现任何错误，则为false，否则为true。

* explain() - 如果有错误，则返回详细解释错误的消息。
