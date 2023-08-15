# Automatic generation
@amxx/graphprotocol-utils软件包提供的图形编译工具可以用于自动生成适合您的应用程序的模式和清单。作为OpenZeppelin Subgraphs的一部分实现的模块与此兼容。

## 描述您的应用程序
要生成模式和清单，您必须提供您希望索引的合约的描述。

以下是一个示例配置文件，用于在主网上索引6个合约：
```
{
  "output": "generated/sample.",
  "chain": "mainnet",
  "datasources": [
    { "address": "0xA3B26327482312f70E077aAba62336f7643e41E1", "startBlock": 11633151, "module": [ "erc20",    "accesscontrol" ] },
    { "address": "0xB1C52075b276f87b1834919167312221d50c9D16", "startBlock":  9917641, "module": [ "erc721",   "ownable"       ] },
    { "address": "0x799DAa22654128d0C64d5b79eac9283008158730", "startBlock":  9917642, "module": [ "erc721",   "ownable"       ] },
    { "address": "0xC76A18c78B7e530A165c5683CB1aB134E21938B4", "startBlock":  9917639, "module": [ "erc721",   "ownable"       ] },
    { "address": "0x001d1cd0bcf2e9021056c6fe4428ce15d977cfe0", "startBlock": 11127634, "module": [ "erc1155",  "ownable"       ] },
    { "address": "0x3d85004fa4723de6563909fabbcafee509ee6a52", "startBlock": 12322496, "module": [ "timelock", "accesscontrol" ] }
  ]
}
```
每个合约都被表示为数据源，具有地址、（可选的）起始块和功能列表。

## 生成架构和清单
可以使用以下命令编译此配置文件。
```
npx graph-compiler \
  --config sample.json \
  --include node_modules/@openzeppelin/subgraphs/src/datasources \
  --export-schema \
  --export-subgraph
```
请注意，在我们的示例中，OpenZeppelin Subgraphs 包是使用 NPM 安装的，这意味着模块描述在 @openzeppelin/subgraphs/src/datasources 中。必须使用 --include 命令添加此路径，以便编译器可以找到它们。

这将产生两个文件，根据配置文件中提供的输出参数：

* generated/sample.schema.graphql 包含定制的模式，仅包含所需的实体。

* generated/sample.subgraph.yaml 包含子图清单。

## 部署您的子图
为了将您的子图部署到托管服务或任何其他图节点，您必须首先创建它。此操作在[此处](https://thegraph.com/docs/developer/deploy-subgraph-hosted)有文档记录。完成此操作后，您可以使用 graph-cli 工具编译和部署生成的子图：
```
npx graph-cli codegen generated/sample.subgraph.yaml
npx graph-cli build generated/sample.subgraph.yaml
npx graph-cli deploy                      \
  --ipfs https://api.thegraph.com/ipfs/   \
  --node https://api.thegraph.com/deploy/ \
  username/subgraphname                   \
  generated/sample.subgraph.yaml
```
