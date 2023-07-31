# CLI Reference
在这里，您可以找到Nile默认提供的核心命令的完整参考。

## 事务执行
执行或查询事务状态的命令。

### setup

#### nile setup <PRIVATE_KEY_ALIAS>
部署与给定私钥相关联的账户。

> IMPORTANT
该命令使用别名而不是实际私钥，以避免意外泄露私钥。该别名与同名的环境变量关联，其值为私钥。

> NOTE
1. 创建或更新存储与账户管理相关的所有数据的localhost.accounts.json文件。
2. 创建或更新存储与部署相关的所有数据的localhost.deployments.txt文件。

##### 参数
* PRIVATE_KEY_ALIAS

指定将拥有要部署的账户的私钥。查找具有私钥别名的环境变量。

##### 选项
* --network

选择网络：localhost、integration、goerli、goerli2、mainnet之一。

默认为localhost。

* --salt

指定地址生成的salt。

* --max_fee

指定您愿意为交易支付的最大费用。

* --simulate和--estimate_fee

用于查询网络而不执行交易的标志。

* --debug和--track

监视账户部署事务的状态。有关完整描述，请参阅*status*命令。

### 声明

#### nile declare <PRIVATE_KEY_ALIAS> <CONTRACT_NAME>
通过账户声明合约。

##### 参数
* PRIVATE_KEY_ALIAS

指定要使用的账户的别名。

* CONTRACT_NAME

指定要声明的合约构件的名称。

##### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

* --max_fee

指定您愿意为交易支付的最大费用。

* --alias

用于注册已声明的class_hash的别名（已弃用）。

* --overriding_path

覆盖构件发现的目录路径。

* --simulate和--estimate_fee

用于在不执行交易的情况下查询网络的标志。

* --debug和--track

监视账户部署交易的状态。有关完整描述，请参见*status*命令。

### 部署

#### nile deploy <PRIVATE_KEY_ALIAS> <CONTRACT> [arg1, arg2...]
通过账户部署合约。

> NOTE
1. 创建或更新localhost.deployments.txt文件，存储与部署相关的所有数据。

##### 参数
* PRIVATE_KEY_ALIAS

指定要使用的账户的别名。

* CONTRACT

指定要部署的合约构件的名称。

* ARGS

构造函数的可选calldata参数。

##### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

* --salt

指定地址生成的盐。

* --max_fee

指定您愿意为交易支付的最大费用。

* --alias

为Nile本地注册目的指定别名。这样可以在不直接使用地址的情况下与合约进行交互。

* --unique

指定账户地址是否应考虑用于目标地址生成。

* --abi

覆盖要注册的构件abi。对于代理很有用。

* --deployer_address

如果需要，指定部署者合约。

* --simulate和--estimate_fee

查询网络而不执行交易的标志。

* --debug和--track

监视账户部署交易的状态。有关完整描述，请参阅status命令。

### 发送

#### nile send <PRIVATE_KEY_ALIAS> <CONTRACT_ID> <METHOD> [arg1, arg2...]
通过账户执行交易。

##### 参数
* PRIVATE_KEY_ALIAS

指定要使用的账户的别名。

* CONTRACT_ID

指定要调用的合约（可以是别名或地址）。

* METHOD

指定要执行的方法。

* ARGS

方法执行的可选calldata参数。

##### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

* --max_fee

指定您愿意为交易支付的最大费用。

* --simulate和--estimate_fee

用于在不执行交易的情况下查询网络的标志。

* --debug和--track

监视账户部署交易的状态。有关完整描述，请参见status命令。

### 状态

#### nile status <TX_HASH>
查询交易的当前状态。

##### 参数
* TX_HASH

指定要查询的交易哈希。

##### 选项
* --network

选择网络：其中之一（localhost、integration、goerli、goerli2、mainnet）。

默认为localhost。

* --track

在挂起的交易状态下继续探测网络。

* --debug

使用本地可用的合约，使被拒绝的交易的错误消息更明确。

意味着--track。

* --contracts_file

覆盖部署文件以查询合约工件。

默认为<NETWORK>.deployments.txt。

#### nile debug <TX_HASH>
nile status --debug 的别名是 nile 状态 --调试。

## 查询
用于查询区块链的工具。

### 调用

#### nile call <CONTRACT_ID> <METHOD> [arg1, arg2...]
执行网络上的读取操作。

##### 参数
* CONTRACT_ID

指定要调用的合约（别名或地址）。

* METHOD

指定要调用的方法。

* ARGS

方法查询的可选 calldata 参数。

##### 选项
* --network

选择网络：其中之一（localhost、integration、goerli、goerli2、mainnet）。

默认为 localhost。

### get-nonce

#### nile get-nonce <ADDRESS>
检索合同（通常是账户）的nonce。

##### 参数
* ADDRESS

指定要查询的合同的地址。

##### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

### get-balance

#### nile get-balance <ADDRESS>
检索合约的以太币余额。

##### 参数
* ADDRESS

指定要查询的合约地址。

##### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

## 项目管理
用于管理项目的工具。

### init

#### nile init
搭建一个简单的Nile项目。

### node

#### nile node
运行一个本地的[starknet-devnet](https://github.com/Shard-Labs/starknet-devnet/)节点。

##### 选项
* --host

指定要监听的地址。

默认为127.0.0.1（使用程序启动时输出的地址）。

* --port

指定要监听的端口。默认为5050。

* --seed

指定要部署的账户的随机种子。

* --lite-mode

通过禁用块哈希和部署哈希计算等功能应用所有lite-mode优化。

### compile

#### nile compile [PATH_TO_CONTRACT]
编译Cairo合约。

编译结果将被写入artifacts/目录。

##### 参数
* PATH_TO_CONTRACT

指定要编译的合约的路径。

##### 选项
* --directory

指定要从中编译合约的目录。

* --account_contract

从[cairo-lang](https://github.com/starkware-libs/cairo-lang) v0.8.0开始，用户必须使用--account_contract标志来编译账户合约。如果合约的名称以Account结尾，例如Account.cairo，EthAccount.cairo，Nile会自动插入该标志。否则，用户必须包含该标志。
```
nile compile contracts/NewAccountType.cairo --account_contract # compiles account contract
```

* --cairo_path

指定编译器必须用于解析Cairo导入的目录。

请参阅[StarkNet文档](https://starknet.io/docs/how_cairo_works/imports.html?highlight=cairo%20path#import-search-paths)中的导入搜索路径。

* --disable-hint-validation

允许编译时未列入白名单的提示。

### run

#### nile run <PATH_TO_SCRIPT>
在NRE的上下文中执行脚本。

##### 参数
* PATH_TO_SCRIPT

要运行的脚本的路径。

##### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

### clean

#### nile clean
删除artifacts/文件夹和部署文件。

### version

#### nile version
打印出Nile版本.

## 实用程序
其他实用程序。

### get-accounts

#### nile get-accounts
检索一个可供脚本集成的就绪账户列表。

> NOTE
账户列表仅包括存在于本地<NETWORK>.accounts.json文件中的账户。在最近的一个版本中，我们添加了一个标志来获取预部署的账户，如果您连接的网络是一个starknet-devnet实例。
通过预部署的账户发送交易可以通过脚本完成，但当前的CLI版本不允许使用这些账户进行nile send。

##### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

* --predeployed

查询开发网络节点的预部署账户。

### counterfactual-address

#### nile counterfactual-address <PRIVATE_KEY_ALIAS>
预计算一个账户合约的部署地址。

##### 参数
* PRIVATE_KEY_ALIAS

指定要使用的私钥的别名。

##### 选项
* --salt

指定地址生成的salt。

默认为0。