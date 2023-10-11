# Subgraphs
OpenZeppelin合约活动的模块化索引。

从npm安装为[@openzeppelin/subgraphs](https://www.npmjs.com/package/@openzeppelin/subgraphs)。

在GitHub上浏览[OpenZeppelin/openzeppelin-subgraphs](https://github.com/OpenZeppelin/openzeppelin-subgraphs)。

## 使用方法
子图使用三个组件描述：

* **graphql模式**，通常命名为schema.graphql，描述数据库实体和链接。

* **子图清单**，通常命名为subgraph.yaml，描述应监听的活动（合约地址，事件处理程序，函数处理程序）。

* **汇编脚本编写的索引逻辑**，将处理区块链活动并相应地更新数据库。

OpenZeppelin子图提供模式描述，对应的索引逻辑和构建子图清单的模板。

与OpenZeppelin合约提供可以组装以便于构建应用程序的固态代码的方式类似，OpenZeppelin子图提供专门用于索引与这些功能对应的活动的模块。这些模块可以组合以索引复杂的链上活动，而无需实际编写大部分功能的索引逻辑。

### 构建您的清单
可以使用每个模块提供的模板为您的应用程序构建清单。这些模板可以在src/datasource/<module-name>.yaml中找到。对于每个数据源，您将需要填写您的合约的名称，网络，地址和startBlock。如果一个合约实现了多个模块，您可能希望有多个数据源监听同一个地址（每个模块一个）。

**注意：**为了使索引逻辑工作，您将需要为每个使用的模块命名一个数据源，名称为模块的名称。

@amxx/graphprotocol-utils提供了*自动生成清单的工具*。

### 组装您的模式
根据您使用的模块，您的模式将需要包括相应的实体。组装模式可能很困难，因为graphql模式本身不支持导入和合并操作。我们确实为每个模块提供了预编译的模式，在generated/<module-name>.schema.graphql中。我们还提供了一个包含所有模块的所有实体的模式，在generated/all.schema.graphql中。

与清单类似，@amxx/graphprotocol-utils提供了**自动生成模式的工具**。

## 模块

|module name|availability|
|--|--|
|erc20|✔|
|erc20votes|Planned|
|erc721|✔|
|erc777|Planned|
|erc1155|✔|
|erc1967upgrade|✔|
|ownable|✔|
|accesscontrol|✔|
|pausable|✔|
|timelock|✔|
|governor|✔|

### 使用方法
通过在子图中结合多个模块和数据源，你可以构建如下查询，该查询检查了在其上具有访问控制的ERC20令牌的详细信息，并返回管理员的余额。
```
{
  erc20Contract(id: "<erc20-with-accesscontrol-address-in-lowercase>") {
    name
    symbol
    decimals
    totalSupply { value }
    asAccount {
      asAccessControl {
        admins: roles(where: { role: "0x0000000000000000000000000000000000000000000000000000000000000000" }) {
          members {
            account {
              address: id
              balance: ERC20balances(where: { contract: "<erc20-with-accesscontrol-address-in-lowercase>" }) {
                value
              }
            }
          }
        }
      }
    }
  }
}
```