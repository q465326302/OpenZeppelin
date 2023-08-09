# NRE Reference
这里可以找到尼罗运行环境提供的函数的完整参考。

**FUNCTIONS**
编译

部署

调用

获取部署

获取声明

获取或部署账户

获取账户

获取nonce

获取余额

### compile(contracts, cairo_path=None) → None
编译合同列表。

#### 参数
* contracts

要编译的合同列表。

* cairo_path

指定Cairo编译器解析导入的一组目录。

### deploy(contract, arguments=None, alias=None, overriding_path=None, abi=None, mainnet_token=None, watch_mode=None) → (address, abi)
部署智能合约。

> WARNING
已弃用，推荐通过账户进行部署。

#### 参数
* contract

要部署的合约。

* arguments

构造函数的参数列表（calldata）。

* alias

部署注册的别名。

* overriding_path

重写合约和ABI的路径。

* abi

用于注册的不同ABI（对于代理很有用）。

* mainnet_token

主网部署的代币。

* watch_mode

跟踪或调试模式（检查状态命令）。

#### 返回值
* address

新合约的地址。

* abi

在部署中注册的ABI。

### call(address_or_alias, method, params=None, abi=None) → output
在智能合约中调用一个视图函数。

#### 参数
* address_or_alias

要调用的合约的标识符（别名需要在部署中注册）。

* method

要调用的方法。

* params

调用的参数。

* abi

如果需要，可以重写abi。

#### 返回值
* output

来自底层starknet cli调用的输出。

### get_deployment(address_or_alias) → (address, abi)
通过标识符获取部署。

#### 参数
* address_or_alias

合约标识符。

#### 返回值
* address

已注册的合约地址。

* abi

已注册的合约abi。

### get_declaration(hash_or_alias) → class_hash
通过其标识符获取一个已声明的类。

#### 参数
* hash_or_alias

合同标识符。

#### 返回值
* class_hash

声明的合同类哈希。

### get_or_deploy_account(signer, watch_mode=None) → account
获取或部署一个账户合约。

#### 参数
* signer

表示关联的私钥的别名。

* watch_mode

可以是None、track或debug。如果需要，阻塞执行以查询部署交易的状态。

#### 返回值
* account

与签名者匹配的账户。

### get_accounts(predeployed=False) → accounts
检索和管理已部署的账户。

#### 参数
* predeployed

从 starknet-devnet 节点获取预部署的账户。

#### 返回值
* accounts

已注册的账户。

### get_nonce(contract_address) → current_nonce
获取合约的nonce。

#### 参数
* contract_address

要查询的合约地址。

#### 返回值
* current_nonce

合约的nonce。

### get_balance(contract_address) → balance
获取地址的以太币余额。

#### 参数
* contract_address

要查询的合约地址。

#### 返回值
* balance

合约的余额。