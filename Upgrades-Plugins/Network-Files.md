# Network Files
OpenZeppelin Upgrades会在项目根目录的.openzeppelin文件夹中跟踪你部署的所有合约版本，以及代理管理员。你将在该文件夹中找到每个网络的一个文件。建议你将所有网络的文件都提交到源代码控制中，除了开发网络（你可能会在.openzeppelin/unknown-*.json中看到这些文件）。

> NOTE
.openzeppelin文件夹中的文件格式与OpenZeppelin CLI的文件格式不兼容。如果你想在现有的[OpenZeppelin CLI](https://docs.openzeppelin.com/cli/2.8/)项目中使用这些插件，你必须先进行迁移。请参阅从[CLI迁移](./Migrate-from-OpenZeppelin-CLI/Migrate-from-OpenZeppelin-CLI-Hardhat.md)的说明。

## <network_name>.json
OpenZeppelin Upgrades会为你工作的每个网络（ropsten、mainnet等）生成一个文件。这些文件具有相同的结构：
```
// .openzeppelin/<network_name>.json
{
  "manifestVersion": "3.0",
  "impls": {
    "...": {
      "address": "...",
      "txHash": "...",
      "layout": {
        "storage": [...],
        "types": {...}
      }
    },
    "...": {
      "address": "...",
      "txHash": "...",
      "layout": {
        "storage": [...],
        "types": {...}
      }
    }
  },
  "admin": {
    "address": "...",
    "txHash": "..."
  }
}
```

除了部署地址之外，每个逻辑合约还会跟踪以下信息：
* types：跟踪合约或其祖先中使用的所有类型，从基本类型如uint256到自定义的结构体类型。
* storage：跟踪线性化合约的存储布局，引用types部分中定义的类型，并用于[验证后续版本之间的任何存储布局更改是否兼容](./Frequently-Asked-Questions.md#实现是兼容的意味着什么)。

文件的命名将是<network_name>.json，但请注意，<network_name>不是从Truffle或Hardhat配置文件中网络条目的名称中获取的，而是从与条目关联的链ID推断出来的。

有一组有限的公共链；未在列表中的链如Ethereum Classic将使用unknown-<chain_id>.json命名网络文件。

## 版本控制中的配置文件
公共网络文件如mainnet.json或ropsten.json应该被追踪到版本控制中。这些文件包含了关于项目在相应网络中状态的宝贵信息，比如已部署的合约版本的地址。这样的文件对于项目的所有贡献者来说应该是相同的。

然而，本地网络文件如unknown-<chain_id>.json只代表项目在临时本地网络（如ganache-cli）中的部署，只和项目的单个贡献者相关，不应该被追踪到版本控制中。

一个示例的.gitignore文件可能包含以下条目：
```
// .gitignore
# OpenZeppelin
.openzeppelin/unknown-*.json
```

## 临时文件
使用Hardhat版本2.12.3或更高版本的Hardhat开发网络将在操作系统的临时目录下的openzeppelin-upgrades文件夹中写入网络文件。这些文件的名称为hardhat-<chain_id>-<instance_id>.json，其中<instance_id>是一个以0x为前缀的十六进制id，用于唯一标识Hardhat网络的一个实例/运行。当相应的Hardhat网络实例不再活动时，例如在Hardhat测试完成后，可以安全地删除临时文件夹中的文件。

你可以通过以下方式确定临时网络文件的位置：

1. 运行export DEBUG=@openzeppelin:*以启用调试日志记录。
2. 运行你的Hardhat测试或脚本。
3. 找到包含文本"development manifest file:"的日志消息。