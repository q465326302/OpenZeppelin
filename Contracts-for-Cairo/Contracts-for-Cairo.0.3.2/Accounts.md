# Accounts
与以太坊不同，StarkNet没有本地账户概念，账户是直接从私钥派生的。

因此，在StarkNet中，签名验证必须在合约层面上完成。为了减轻智能合约应用程序（如ERC20代币或交易所）的负担，我们使用账户合约来处理交易认证。

有关该主题的更详细的写作可以在[Perama的博客文章](https://perama-v.github.io/cairo/account-abstraction/)中找到。

## 目录
* 快速入门

* 标准接口

* 密钥、签名和签名者

    * 签名者

    * MockSigner实用程序

    * MockEthSigner实用程序

* 账户入口点

* 调用和AccountCallArray格式

    * 调用

    *  AccountCallArray

* 多调用事务

* API规范

  * get_public_key

  * get_nonce

  * set_public_key

  * is_valid_signature

  * __execute__

  * is_valid_eth_signature

  * eth_execute

  * _unsafe_execute

* 预设

    * 账户

    * 以太坊账户

* 使用ERC165进行账户内省

* 扩展账户合约

* L1逃生机制

* 支付燃料费用

## 快速入门
一般的工作流程如下：

1. 将账户合约部署到StarkNet。

2. 现在可以将签名的交易发送到账户合约进行验证和执行。

在Python中，代码如下：
```
from starkware.starknet.testing.starknet import Starknet
from utils import get_contract_class
from signers import MockSigner

signer = MockSigner(123456789987654321)

starknet = await Starknet.empty()

# 1. Deploy Account
account = await starknet.deploy(
    get_contract_class("Account"),
    constructor_calldata=[signer.public_key]
)

# 2. Send transaction through Account
await signer.send_transaction(account, some_contract_address, 'some_function', [some_parameter])
```

## 标准接口
[IAccount.cairo](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/account/IAccount.cairo)合约接口包含了[#41](https://github.com/OpenZeppelin/cairo-contracts/discussions/41)提出的标准账户接口，并被OpenZeppelin和Argent采用。它实现了[EIP-1271](https://eips.ethereum.org/EIPS/eip-1271)，并且与签名验证无关。此外，协议级别处理nonce管理。


```
@contract_interface
namespace IAccount:
    #
    # Getters
    #

    func get_nonce() -> (res : felt):
    end

    #
    # Business logic
    #

    func is_valid_signature(
            hash: felt,
            signature_len: felt,
            signature: felt*
        ) -> (is_valid: felt):
    end

    func __execute__(
            call_array_len: felt,
            call_array: AccountCallArray*,
            calldata_len: felt,
            calldata: felt*,
            nonce: felt
        ) -> (response_len: felt, response: felt*):
    end
end
```

## 密钥、签名和签署者
尽管接口与签名验证方案无关，但该实现假设存在一个控制账户的公私钥对。这就是为什么构造函数期望一个public_key参数来设置它。由于还有一个setPublicKey()方法，因此可以有效地转移账户。

### 签署者
签署者负责使用用户的私钥为给定的交易创建交易签名。该实现利用[Nile的Signer类](https://github.com/OpenZeppelin/nile/blob/main/src/nile/signer.py)通过Signer方法sign_transaction创建交易签名。

sign_transaction对每个交易期望以下参数：

* sender调用tx的合约地址。

* calls包含每个要发送的调用的子列表的列表。每个子列表必须包含：

1. to消息的目标合约的地址。

2. selector要在目标合约上调用的函数。

3. calldata给定选择器的参数。

* nonce防止交易重放的唯一标识符。

* max_fee用户将支付的最大费用。

返回：
* calls 要在交易中捆绑的调用列表

* calldata每个调用的参数列表。

* sig_r交易签名。

* sig_s交易签名。

虽然Signer类执行了大部分发送交易所需的工作，但它既不管理nonce，也不在Account合约上调用实际的交易。为了简化账户管理，大部分工作都通过MockSigner进行了抽象。

### MockSigner实用程序
[utils.py](https://github.com/OpenZeppelin/cairo-contracts/blob/main/tests/utils.py)中的MockSigner类用于在给定的账户上执行交易，构建交易并管理nonce。

交易的流程始于检查nonce并将每个调用的合约地址转换为十六进制格式。十六进制转换是必要的，因为Nile的Signer将地址转换为一个十六进制整数（需要一个字符串参数）。注意，直接转换为字符串最终会导致一个超过Cairo的FIELD_PRIME的整数。

交易中包含的值被传递给Nile的Signer的sign_transaction方法，该方法创建并返回一个签名。最后，MockSigner实例调用账户合约的__execute__方法来执行交易数据。

用户只需要与以下公开的方法交互来执行交易：

* send_transaction(account, to, selector_name, calldata, nonce=None, max_fee=0)返回一个已签名的交易的future，准备发送。

* send_transactions(account, calls, nonce=None, max_fee=0)返回一个已批处理的已签名交易的future，准备发送。

要使用MockSigner，在实例化类时传递一个私钥。
```
from utils import MockSigner

PRIVATE_KEY = 123456789987654321
signer = MockSigner(PRIVATE_KEY)
```

然后使用send_transaction方法发送单个交易。
```
await signer.send_transaction(account, contract_address, 'method_name', [])
```

如果使用multicall，可以使用send_transactions方法发送多个交易。
```
await signer.send_transactions(
    account,
    [
        (contract_address, 'method_name', [param1, param2]),
        (contract_address, 'another_method', [])
    ]
)
```

### MockEthSigner utility
[utils.py](https://github.com/OpenZeppelin/cairo-contracts/blob/main/tests/utils.py)中的MockEthSigner类用于使用secp256k1曲线密钥对在给定的账户上执行交易，构建交易并管理nonce。它与MockSigner实现不同之处在于：

* 不使用公钥，而是使用派生的地址（公钥的keccak256哈希的最后20个字节，并在开头添加0x）

* 使用secp256k1曲线地址对消息进行签名

## Account entrypoint
__execute__作为所有用户与任何合约进行交互的单个入口点，包括管理账户合约本身。这就是为什么如果您想要更改控制账户的公钥，您将发送一个针对该账户合约的交易的原因:
```
await signer.send_transaction(account, account.contract_address, 'set_public_key', [NEW_KEY])
```
如果您想要在AccountRegistry合约中更新账户的L1地址，您可以执行以下操作
```
await signer.send_transaction(account, registry.contract_address, 'set_L1_address', [NEW_ADDRESS])
```
您可以在[账户消息方案讨论](https://github.com/OpenZeppelin/cairo-contracts/discussions/24)中阅读更多关于消息结构和哈希的信息。有关多调用的设计选择和实现的更多信息，请阅读[账户多调用应如何工作讨论](https://github.com/OpenZeppelin/cairo-contracts/discussions/27)。

__execute__方法具有以下接口:
```
func __execute__(
        call_array_len: felt,
        call_array: AccountCallArray*,
        calldata_len: felt,
        calldata: felt*,
        nonce: felt
    ) -> (response_len: felt, response: felt*):
end
```
在这里：

* call_array_len是调用数量

* call_array是表示每个调用的数组

* calldata_len是calldata参数的数量

* calldata是表示函数参数的数组

* nonce是防止交易重放的唯一标识符。当前实现要求nonce是递增的

> NOTE
一旦StarkNet允许在结构数组中使用指针，构建__execute__方法中的多重调用事务的方案将发生变化。在这种情况下，可以将多个事务传递给（而不是在内部构建）__execute__方法。

### 调用和AccountCallArray格式
这个想法是将所有用户意图编码为表示智能合约调用的Call。用户还可以将多个消息打包到单个交易中（创建多个调用事务）。Cairo目前不支持带有指针的结构体数组，这意味着__execute__函数无法正确迭代多个调用。因此，该实现使用了AccountCallArray结构的解决方法。请参阅*多调用事务*。

#### 调用
单个调用的结构如下：
```
struct Call:
    member to: felt
    member selector: felt
    member calldata_len: felt
    member calldata: felt*
end
```

* to是消息的目标合约地址。

* selector是要在目标合约上调用的函数的选择器。

* calldata_len是calldata参数的数量。

* calldata是表示函数参数的数组。

### AccountCallArray
AccountCallArray的结构如下：
```
struct AccountCallArray:
    member to: felt
    member selector: felt
    member data_offset: felt
    member data_len: felt
end
```

* to是消息的目标合约的地址。

* selector是要在目标合约上调用的函数的选择器。

* data_offset是保存调用数据的calldata数组的起始位置。

* data_len是Call中calldata元素的数量。


## 多次调用交易
多调用事务将每个调用的to、selector、calldata_offset和calldata_len打包到AccountCallArray结构中，并将每个调用的累积calldata保留在单独的数组中。__execute__函数通过将AccountCallArray与其calldata（由该特定调用指定的偏移量和calldata长度界定）组合来重建每个消息。重建逻辑在内部的_from_call_array_to_call中设置。

基本流程如下：

首先，用户通过Signer实例化将事务的消息发送出去，其形式如下：
```
 await signer.send_transaction(
         account, [
             (contract_address, 'contract_method', [arg_1]),
             (contract_address, 'another_method', [arg_1, arg_2])
         ]
     )
```

然后，在[utils.py](https://github.com/OpenZeppelin/cairo-contracts/blob/main/tests/utils.py)中，_from_call_to_call_array方法将每个调用转换为AccountCallArray格式，并将每个调用的calldata累积存储到一个单独的数组中。接下来，这两个数组（以及发送者、nonce和最大费用）被用来创建交易哈希。然后，签名者使用签名调用__execute__，并将AccountCallArray、calldata和nonce作为参数传递。

最后，__execute__方法接受AccountCallArray和calldata，并构建一个Calls（MultiCall）数组。

> NOTE
每个交易都使用AccountCallArray。一个单独的Call被视为一个带有一条消息的捆绑。

## API规范
这是Account合约的公共API的简要概述
```
func get_public_key() -> (res: felt):
end

func get_nonce() -> (res: felt):
end

func set_public_key(new_public_key: felt):
end

func is_valid_signature(hash: felt,
        signature_len: felt,
        signature: felt*
    ) -> (is_valid: felt):
end

func __execute__(
        call_array_len: felt,
        call_array: AccountCallArray*,
        calldata_len: felt,
        calldata: felt*,
        nonce: felt
    ) -> (response_len: felt, response: felt*):
end
```

#### get_public_key
返回与账户合约关联的公钥。

参数：无。

返回值：
```
public_key: felt
```

#### get_nonce
返回账户的当前交易nonce。

参数：无。

返回值：
```
nonce: felt
```

#### set_public_key
设置将控制此账户的公钥。可以用于安全地更换密钥、在密钥被泄露的情况下更换密钥，甚至转移账户的所有权。

参数：
```
public_key: felt
```
返回值：无。

#### is_valid_signature
这个函数受到[EIP-1271](https://eips.ethereum.org/EIPS/eip-1271)的启发，如果给定的签名有效，则返回TRUE，否则会回滚。将来，如果给定的签名无效，它将返回FALSE（有关更多信息，请查看[此问题](https://github.com/OpenZeppelin/cairo-contracts/issues/327)）。
参数：
```
hash: felt
signature_len: felt
signature: felt*
```
返回值：
```
is_valid: felt
```

> NOTE
如果给定的签名无效，将来可能返回FALSE（请参考[此问题](https://github.com/OpenZeppelin/cairo-contracts/issues/327)的讨论）。

#### __execute__
这是与Account合约进行交互的唯一外部入口。它执行以下操作：

1. 验证交易签名与消息匹配（包括nonce）

2. 增加nonce

3. 使用预期的函数选择器和calldata参数调用目标合约

4. 将合约调用的响应数据作为返回值转发

参数:
```
call_array_len: felt
call_array: AccountCallArray*
calldata_len: felt
calldata: felt*
nonce: felt
```

> NOTE
当前的签名方案期望一个包含两个元素的数组，格式为 [sig_r, sig_s]。

返回：
```
response_len: felt
response: felt*
```

#### is_valid_eth_signature
如果在secp256k1曲线上给定的签名有效，则返回TRUE，否则将还原。将来，如果给定的签名无效，它将返回FALSE（有关更多信息，请查看[此问题](https://github.com/OpenZeppelin/cairo-contracts/issues/327)）。

参数:
```
signature_len: felt
signature: felt*
```
返回：
```
is_valid: felt
```

> NOTE
如果给定的签名无效，将来可能返回FALSE（请参考[此问题](https://github.com/OpenZeppelin/cairo-contracts/issues/327)的讨论）。

#### eth_execute
这个版本的execute与原始版本的execute遵循相同的思路，唯一的区别在于签名验证是基于secp256k1曲线。

参数:
```
call_array_len: felt
call_array: AccountCallArray*
calldata_len: felt
calldata: felt*
nonce: felt
```
返回:
```
response_len: felt
response: felt*
```
> NOTE
当前的签名方案期望一个包含7个元素的数组，格式如下：[sig_v, uint256_sig_r_low, uint256_sig_r_high, uint256_sig_s_low, uint256_sig_s_high, uint256_hash_low, uint256_hash_high]，其中验证的参数比一个felt更大。

#### _unsafe_execute
这是一个*内部方法*，执行以下任务：

1. 递增nonce。

2. 为每个迭代的消息构建一个调用。有关更多信息，请参阅*多合一交易*。

3. 使用预期的函数选择器和calldata参数调用目标合约。

4. 将合约调用的响应数据作为返回值转发。

## 预设
以下合约预设已准备就绪，可用于快速原型设计和测试。每个预设在使用的账户签名类型上有所不同。

### 账户
[账户](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.6.1/src/openzeppelin/account/presets/Account.cairo)预设使用StarkNet密钥验证交易。

### 以太坊账户
[以太坊账户](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.6.1/src/openzeppelin/account/presets/EthAccount.cairo)预设支持以太坊地址，使用secp256k1密钥验证交易。

## 带ERC165的账户自省
某些合约，如ERC721，需要一种区分账户合约和非账户合约的方法。为了将合约声明为账户，它应该按照[#100](https://github.com/OpenZeppelin/cairo-contracts/discussions/100)中提议的方式实现[ERC165](https://eips.ethereum.org/EIPS/eip-165)。为了符合ERC165规范，我们的想法是计算IAccount的EVM选择器（而不是StarkNet选择器）的异或值。IAccount的结果magic value是0x50b70dcb。

我们在StarkNet上的ERC165集成受到了OpenZeppelin的Solidity实现*ERC165Storage*的启发，该实现存储了实现合约支持的接口。对于账户合约，使用IAccount的magic value查询账户地址的supportsInterface应该返回TRUE。

> NOTE
对于账户合约，ERC165支持是静态的，不需要账户合约进行注册。

## 扩展账户合约
账户合约可以通过遵循*可扩展性模式*来进行扩展。

要实现自定义账户合约，应该暴露一对验证函数和执行函数。这就是为什么Account库提供了不同类型的这样的函数对，比如原始的is_valid_signature和execute，或者以太坊风格的is_valid_eth_signature和eth_execute。

鼓励账户合约开发者实现[标准的Account接口](https://github.com/OpenZeppelin/cairo-contracts/discussions/41)，然后再加入自定义逻辑。

要实现替代的执行函数，在调用_unsafe_execute构建块之前，确保检查相应的验证函数，就像当前的预设函数所做的那样。不要直接暴露_unsafe_execute。

> IMPORTANT
新方法中应包含ecdsa_ptr隐式参数，该参数在调用_unsafe_execute时使用（即使ecdsa_ptr未被使用）。否则，账户的功能可能在测试和本地开发网络环境中都能正常工作，但在公共网络上可能因为[SignatureBuiltinRunner](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/lang/builtins/signature/signature_builtin_runner.py)而失败。有关更多信息，请参见[问题＃386](https://github.com/OpenZeppelin/cairo-contracts/issues/386)。

未来可能要注意的其他验证方案：

* 多重签名。

* 像[Argent账户](https://github.com/argentlabs/argent-contracts-starknet/blob/de5654555309fa76160ba3d7393d32d2b12e7349/contracts/ArgentAccount.cairo)中的守护逻辑。


## L1逃生机制

## 支付燃料费用