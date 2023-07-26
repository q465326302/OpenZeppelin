# NRE Reference
在这里，您可以找到Nile运行时环境提供的函数的完整参考，以及账户方法的参考。

## NRE API
NRE对象提供的功能。

### compile

#### compile(contracts, cairo_path=None) → None
编译合同列表。

##### 参数
* contracts

要编译的合同列表。

* cairo_path

指定Cairo编译器解析导入的一组目录。

##### call

#### call(address_or_alias, method, params=None, abi=None) → output
在智能合约中调用一个视图函数。

##### 参数
* address_or_alias

要调用的合约的标识符（别名需要在部署中注册）。

* method

要调用的方法。

* params

调用的参数。

* abi

如果需要，可以覆盖abi。

##### 返回值
* output

来自底层starknet cli调用的输出。

#### get_deployment

#### get_deployment(address_or_alias) → (address, abi)
通过其标识符获取部署。

##### 参数
* address_or_alias

合同标识符。

##### 返回值
* address

已注册的合同地址。

* abi

已注册的合同abi。

### get_declaration

#### get_declaration(hash_or_alias) → class_hash
通过其标识符获取一个已声明的类。

##### 参数
* hash_or_alias

合约标识符。

##### 返回值
* class_hash

已声明的合约类哈希。

### get_or_deploy_account

#### get_or_deploy_account
获取或部署一个账户合约。

##### 参数
* signer

表示关联的私钥的别名。

* watch_mode

可以是None、track或debug。如果需要，阻塞执行以查询部署事务的状态。

##### 返回值
* account

与签名者匹配的账户实例。

### get_accounts

#### get_accounts(predeployed=False) → accounts
检索和管理已部署的账户。

##### 参数
* predeployed

从starknet-devnet节点获取预部署的账户。

##### 返回值
* accounts

已注册账户的列表。

### get_nonce

#### get_nonce(contract_address) → current_nonce
获取合约的nonce。

##### 参数
* contract_address

要查询的合约地址。

##### 返回值
* current_nonce

合约的nonce。

### get_balance

#### get_balance(contract_address) → balance
获取地址的以太币余额。

##### 参数
* contract_address

要查询的合约地址。

##### 返回值
* balance

合约的余额。

## Account API
账户抽象的公共API。

### send

#### async send(self, address_or_alias, method, calldata, nonce=None, max_fee=None) → transaction
返回表示调用交易的Transaction实例。

##### 参数
* address_or_alias

目标合约标识符（别名需要在部署中注册）。

* method

要执行的方法。

* calldata

调用的参数。

* nonce

账户nonce。如果为None，则会自动计算。

* max_fee

您愿意为交易执行支付的最大费用。

通常将此值保留为None，因为返回的交易允许估计和更新费用。

##### 返回值
* transaction

一个Transaction实例。

### declare

#### async declare(self, contract_name, nonce=None, max_fee=None, alias=None, overriding_path=None, nile_account=False) → transaction

返回一个表示声明交易的Transaction实例。

##### 参数
* contract_name

要声明的合约的名称（用于解析工件）。

* nonce

账户nonce。如果设置为None，则会自动计算。

* max_fee

您愿意为交易执行支付的最大费用。

通常将此值保留为None，因为返回的交易允许估算和更新费用。

* alias

用于注册声明的class_hash的别名（已弃用）。

* overriding_path

用于工件和abi解析的路径覆盖。

* nile_account

是否使用OZ账户工件。

##### 返回值
* transaction

一个Transaction实例。

## Transaction API
Transaction抽象的公共API。

### estimate_fee

#### async estimate_fee(self) → max_fee
返回执行交易的估计费用。

##### 返回值
* max_fee

估计的费用。

### simulate

#### async simulate(self) → trace
返回模拟执行的跟踪。

##### 返回值
* 跟踪

表示模拟的对象。

### execute

#### async execute(self, watch_mode=None) → (tx_status, log_output)
执行交易。

##### 参数
* watch_mode

允许等待交易被包含在一个区块中。可以是None、track或debug。track表示在挂起交易状态下继续探测网络。debug表示使用本地可用的合约来使被拒绝的交易的错误消息更明确（意味着track）。

默认为None（非阻塞）。

##### 返回值
* tx_status

一个交易状态对象。

* log_output

表示内部调用输出的字符串。

### update_fee

#### update_fee(self, max_fee) → self
更新max_fee，修改交易哈希。

##### 参数
* max_fee

要设置的新max_fee。

##### 返回值
* self

返回self以允许与execute链接。