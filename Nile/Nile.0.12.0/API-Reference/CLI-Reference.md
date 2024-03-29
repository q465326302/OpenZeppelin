# CLI Reference
在这里，你可以找到Nile提供的默认核心命令的完整参考。

**TRANSACTIONS**
[setup](#nile-setup-private_key_alias)

[declare](#nile-declare-private_key_alias-contract_name)

[deploy](#nile-deploy-private_key_alias-arg1-arg2)

[send](#nile-send-private_key_alias-contract_id-arg1-arg2)

[status](#nile-status-tx_hash)

[debug](#nile-debug-tx_hash)

**QUERIES**
[call](#nile-call-contract_id-arg1-arg2)

[get-nonce](#nile-get-nonce)

[get-balance](#nile-get-balance)

**PROJECT MANAGEMENT**
[init](#nile-init)

[node](#nile-node)

[compile](#nile-compile-path_to_contract)

[run](#nile-run-path_to_script)

[clean](#nile-clean)

[version](#nile-version)

**UTILS**
获取账户

合约地址

### nile node
运行一个本地的[starknet-devnet](https://github.com/Shard-Labs/starknet-devnet/)节点。

#### 选项
* --host

指定要监听的地址。

默认为127.0.0.1（使用程序启动时输出的地址）。

* --port

指定要监听的端口。默认为5050。

* --seed

指定要部署的账户的随机种子。

* --lite-mode

通过禁用块哈希和部署哈希计算等功能，应用所有的lite-mode优化。

### nile compile [PATH_TO_CONTRACT]
编译Cairo合约。

编译结果将写入artifacts/目录。

#### 参数
* PATH_TO_CONTRACT

指定要编译的合约的路径。

#### 选项
* --directory

指定要从中编译合约的目录。

* --account_contract

从[cairo-lang](https://github.com/starkware-libs/cairo-lang) v0.8.0开始，用户必须使用--account_contract标志编译帐户合约。如果合约的名称以Account结尾（例如Account.cairo，EthAccount.cairo），Nile会自动插入该标志。否则，用户必须包含该标志。
```
nile compile contracts/NewAccountType.cairo --account_contract # compiles account contract
```

* --cairo_path

指定编译器必须使用的目录来解析Cairo的导入。

> TIP
请参阅[StarkNet文档](https://starknet.io/docs/how_cairo_works/imports.html?highlight=cairo%20path#import-search-paths)中的导入搜索路径。

* --disable-hint-validation

允许编译时未列入白名单的提示。

### nile setup <PRIVATE_KEY_ALIAS>
部署与给定私钥相关联的账户。

> IMPORTANT
该命令使用别名而不是实际私钥，以避免意外泄露私钥。该别名与同名的环境变量关联，其值为私钥。

> NOTE
1. 创建或更新localhost.accounts.json文件，存储所有与账户管理相关的数据。
2. 创建或更新localhost.deployments.txt文件，存储所有与部署相关的数据。

#### 参数
* PRIVATE_KEY_ALIAS

指定将拥有要部署的账户的私钥。查找具有私钥别名的环境变量。

选项
* --network

选择网络：localhost、integration、goerli、goerli2、mainnet之一。

默认为localhost。

* --debug和--track

监视账户部署交易的状态。有关完整描述，请参见[status](#nile-status-tx_hash)命令。

### nile declare <PRIVATE_KEY_ALIAS> <CONTRACT_NAME>
通过账户声明一个合约。

#### 参数
* PRIVATE_KEY_ALIAS

指定要使用的账户的别名。

* CONTRACT_NAME

指定要声明的合约构件的名称。

#### 选项
* --network

选择网络：其中之一（localhost、integration、goerli、goerli2、mainnet）。

默认为localhost。

* --max_fee

指定你愿意为交易支付的最大费用。

* --overriding_path

重写构件发现的目录路径。

* --token

用于向Alpha Mainnet声明合约。

* --debug和--track

监视账户部署交易的状态。有关完整描述，请参见[status](#nile-status-tx_hash)命令。

### nile deploy <PRIVATE_KEY_ALIAS> <CONTRACT> [arg1, arg2...]
通过账户部署合约。

> NOTE
1. 创建或更新localhost.deployments.txt文件，存储与部署相关的所有数据。

#### 参数
* PRIVATE_KEY_ALIAS

指定要使用的账户的别名。

* CONTRACT

指定要部署的合约构件的名称。

* ARGS

构造函数的可选calldata参数。

#### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

* --max_fee

指定你愿意为交易支付的最大费用。

* --salt

设置地址生成的基本salt。

* --unique

指定账户地址是否应考虑用于目标地址生成。

* --abi

重写要注册的构件abi。对于代理很有用。

* --deployer_address

如果需要，指定部署者合约。

* --ignore_account

不使用账户进行部署（已弃用）。

* --token

用于将合约部署到Alpha Mainnet。

* --debug和--track

监视账户部署交易的状态。有关完整描述，请参见[status](#nile-status-tx_hash)命令。

### nile call <CONTRACT_ID> <METHOD> [arg1, arg2...]
执行针对网络的读取操作。

#### 参数
* CONTRACT_ID

指定要调用的合约（别名或地址）。

* METHOD

指定要调用的方法。

* ARGS

方法查询的可选calldata参数。

选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

### nile send <PRIVATE_KEY_ALIAS> <CONTRACT_ID> <METHOD> [arg1, arg2...]
执行一个通过账户的交易。

#### 参数
* PRIVATE_KEY_ALIAS

指定代表要使用的账户的别名。

* CONTRACT_ID

指定要调用的合约（别名或地址）。

* METHOD

指定要执行的方法。

* ARGS

方法执行的可选calldata参数。

#### Options
* --network

选择网络：（localhost，integration，goerli，goerli2，mainnet）之一。

默认为localhost。

默认为localhost。

* --max_fee

指定你愿意为交易支付的最高费用。

* --simulate and --estimate_fee

查询网络而不执行交易的标志。

* --debug and --track

观察账户部署交易的状态。查看状态命令以获取完整描述。

### nile counterfactual-address <PRIVATE_KEY_ALIAS>
预计算一个账户合约的部署地址。

#### 参数
* PRIVATE_KEY_ALIAS

指定要使用的私钥的别名。

#### 选项
* --salt

指定地址生成的salt。

默认为0。

### nile status <TX_HASH>
查询交易的当前状态。

#### 参数
* TX_HASH

指定要查询的交易的哈希值。

#### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

* --track

在挂起的交易状态下继续探测网络。

* --debug

使用本地可用的合约，使被拒绝的交易的错误消息更明确。

意味着--track。

* --contracts_file

重写部署文件以查询合约的构件。

默认为<NETWORK>.deployments.txt。

### nile debug <TX_HASH>
nile状态的别名 --debug。

### nile get-accounts
检索一个可用于简单脚本集成的账户列表。

> NOTE
账户列表仅包括存在于本地<NETWORK>.accounts.json文件中的账户。在最近的版本中，我们添加了一个标志到该命令，以获取预部署的账户，如果你连接的网络是starknet-devnet实例。
通过预部署的账户发送交易可以通过脚本完成，但当前的CLI版本不允许使用这些账户进行nile发送。

#### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

* --predeployed

从devnet节点查询预部署的账户。

### nile get-nonce <ADDRESS>
检索合约（通常是账户）的nonce。

#### 参数
* ADDRESS

指定要查询的合约的地址。

#### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

### nile get-balance <ADDRESS>
查询合约的以太余额。

#### 参数
* ADDRESS

指定要查询的合约地址。

#### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

### nile get-balance <ADDRESS>
检索合约的以太余额。

#### 参数
* ADDRESS

指定要查询的合约地址。

#### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

默认为localhost。

### nile run <PATH_TO_SCRIPT>
在NRE的上下文中执行脚本。

#### 参数
* PATH_TO_SCRIPT

要运行的脚本的路径。

#### 选项
* --network

选择网络：其中之一（localhost，integration，goerli，goerli2，mainnet）。

默认为localhost。

### nile clean
删除 artifacts/ 文件夹和 deployments 文件。

### nile init
创建一个简单的 Nile 项目。

### nile version
打印出 Nile 版本。