# Automatic generation
作为@amxx/graphprotocol-utils包的一部分提供的graph-compile工具可以用来自动生成为您的应用量身定制的模式和清单。作为OpenZeppelin子图的一部分实现的模块与此兼容。

## 描述您的应用
为了生成模式和清单，您需要提供您希望索引的合约的描述。

以下是一个配置文件示例，它在主网上索引了6个合约:
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

每个合约都被表示为一个数据源，具有一个地址，一个（可选的）起始块和一系列特性。

## 生成模式和清单
可以使用以下命令编译此配置文件
```
npx graph-compiler \
  --config sample.json \
  --include node_modules/@openzeppelin/subgraphs/src/datasources \
  --export-schema \
  --export-subgraph
```

请注意，在我们的示例中，OpenZeppelin Subgraphs包是通过NPM安装的，这意味着模块描述位于@openzeppelin/subgraphs/src/datasources中。必须使用--include命令添加此路径，以便编译器可以找到它们。

这将产生两个文件，根据配置文件中提供的输出参数：

* generated/sample.schema.graphql包含一个定制的模式，它只包含索引模块所需的实体。

* generated/sample.subgraph.yaml包含子图清单。

## 部署你的子图
为了将你的子图部署到托管服务，或者任何其他图节点，你首先需要创建它。这个操作在[这里](https://thegraph.com/docs/developer/deploy-subgraph-hosted)有文档说明。一旦完成，你可以使用graph-cli工具编译和部署生成的子图:
```
npx graph-cli codegen generated/sample.subgraph.yaml
npx graph-cli build generated/sample.subgraph.yaml
npx graph-cli deploy                      \
  --ipfs https://api.thegraph.com/ipfs/   \
  --node https://api.thegraph.com/deploy/ \
  username/subgraphname                   \
  generated/sample.subgraph.yaml
```