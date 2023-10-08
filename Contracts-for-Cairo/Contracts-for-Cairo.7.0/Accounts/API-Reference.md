# Account
关于账户合约的接口、预设和实用程序的参考。

## 核心

### [ISRC6](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/account/interface.cairo#L12)
```
use openzeppelin::account::interface::ISRC6;
```

SRC6标准账户的接口，如[SNIP-6](https://github.com/ericnordelo/SNIPs/blob/feat/standard-account/SNIPS/snip-6.md)中定义的。

**SRC5 ID**
0x2ceccef7f994940b3962a6c67e0ba4fcd37df7d131417c604f91e03caecc1cd

**FUNCTIONS**
__execute__(calls)

__validate__(calls)

is_valid_signature(hash, signature)

#### Functions
##### __execute__(calls: Array<Call>) → Array<Span<felt252>>
external#
执行验证后的调用列表作为一个事务。

返回一个数组，包含每个调用的输出。

> NOTE
Call结构体在corelib中定义。

##### __validate__(calls: Array<Call>) → felt252
external#
在执行交易前进行验证。

如果有效，则返回短字符串'VALID'，否则它将撤销。

##### is_valid_signature(hash: felt252, signature: Array<felt252>) → felt252
external#
验证给定消息哈希的签名是否有效。

如果有效，返回短字符串'VALID'，否则它将撤销。

## [Account](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/account/account.cairo#L27)
```
use openzeppelin::account::Account;
```

实现扩展ISRC6的账户合约实现。

**CONSTRUCTOR**
constructor(self, _public_key)

**EXTERNAL FUNCTIONS**
__validate_deploy__(self, hash, signature)

SRC6IMPL
__execute__(self, calls)

__validate__(self, calls)

is_valid_signature(self, hash, signature)

SRC5IMPL
supports_interface(self, interface_id)

DECLARERIMPL
__validate_declare__(self, class_hash)

PUBLICKEYIMPL
set_public_key(self, new_public_key)

get_public_key(self)

**INTERNAL FUNCTIONS**
INTERNALIMPL
initializer(self, _public_key)

validate_transaction(self)

_set_public_key(self, new_public_key)

_is_valid_signature(self, hash, signature)

assert_only_self(self)

**EVENTS**
OwnerAdded(new_owner_guid)

OwnerRemoved(removed_owner_guid)

#### Constructor

##### constructor(ref self: ContractState, _public_key: felt252)
constructor#
使用给定的公钥初始化账户，并注册ISRC6接口ID。

触发一个*OwnerAdded*事件。

#### External Functions

##### __validate_deploy__(self: @ContractState, class_hash: felt252, contract_address_salt: felt252, _public_key: felt252) → felt252
external#
验证一个*DeployAccount交易*。参见*反事实部署*。

如果有效，返回短字符串'VALID'，否则它将撤销。

##### __execute__(ref self: ContractState, calls: Array<Call>) → Array<Span<felt252>>
external#
参见*ISRC6::__execute__*.

##### __validate__(self: @ContractState, calls: Array<Call>) → felt252
external#
参见*ISRC6::__validate__*.

##### is_valid_signature(self: @ContractState, hash: felt252, signature: Array<felt252>) → felt252
external#
参见*ISRC6::is_valid_signature*.

##### supports_interface(self: @ContractState, interface_id: felt252) → bool
external#
参见 *ISRC5::supports_interface*.

##### __validate_declare__(self: @ContractState, class_hash: felt252) → felt252
external#
验证一个[Declare交易](https://docs.starknet.io/documentation/architecture_and_concepts/Network_Architecture/Blocks/transactions/#declare-transaction)。

如果有效，则返回短字符串'VALID'，否则将其撤销。

##### set_public_key(ref self: ContractState, new_public_key: felt252)
external#
为账户设置新的公钥。只能通过__execute__由账户自身调用访问。

会触发*OwnerRemoved*和*OwnerAdded*事件。

##### get_public_key(self: @ContractState) → felt252
external#
返回账户的当前公钥。

#### Internal Functions

##### initializer(ref self: ContractState, _public_key: felt252)
internal#
使用给定的公钥初始化账户，并注册ISRC6接口ID。

触发一个*OwnerAdded*事件。

##### validate_transaction(self: @ContractState) → felt252
internal#
从全局上下文验证交易签名。

如果有效，则返回短字符串'VALID'，否则它将撤销。

#### _set_public_key(ref self: ContractState, new_public_key: felt252)
internal#
设置公钥而不验证来电者。

发出一个*OwnerAdded*事件。

> CAUTION
不建议在set_public_key函数外使用此方法。

##### _is_valid_signature(self: @ContractState, hash: felt252, signature: Span<felt252>) → bool
internal#
验证提供的签名对于哈希，使用账户当前的公钥。

#### assert_only_self(self: @ContractState)
internal#
验证调用者是否为账户本身。否则，它将撤销。

#### Events

##### OwnerAdded(new_owner_guid: felt252)
event#
当公钥被添加时发出。

##### OwnerRemoved(removed_owner_guid: felt252)
event#
当公钥被移除时发出。