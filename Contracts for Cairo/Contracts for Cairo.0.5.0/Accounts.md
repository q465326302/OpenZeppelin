# Accounts
与以太坊不同，StarkNet没有本地账户概念，账户是直接从私钥派生的。

因此，在StarkNet中，签名验证必须在合约层面上完成。为了减轻智能合约应用程序（如ERC20代币或交易所）的负担，我们使用账户合约来处理交易认证。

有关账户抽象的概述，请参阅StarkWare的[StarkNet Alpha 0.10](https://medium.com/starkware/starknet-alpha-0-10-0-923007290470)。关于这个主题的更详细讨论可以在[StarkNet账户抽象第1部分](https://community.starknet.io/t/starknet-account-abstraction-model-part-1/781)中找到。

## 目录
* 快速入门

* 账户入口点

* 标准接口

* 密钥、签名和签名者

    * 签名验证

    * 签名者

    * MockSigner实用程序

    * MockEthSigner实用程序

* 调用和AccountCallArray格式

    * 调用

    *  AccountCallArray

* 多调用事务

* API规范

    * 构造函数

    * getPublicKey

    * supportsInterface

    * setPublicKey

    * isValidSignature

    * __validate__

    * __validate_declare__

    * __execute__

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

### 账户入口点
账户合约只有三个入口点用于所有用户交互：

1. __validate_declare__ 在声明之前验证声明签名。从Cairo v0.10.0开始，合约类应从一个账户合约中声明。

2. __validate__ 在执行交易之前验证交易签名，然后再使用__execute__执行交易。

3. __execute__ 作为所有用户与任何合约进行交互的改变状态的入口点，包括管理账户合约本身。这就是为什么如果您想要更改控制账户的公钥，您将发送一个针对该账户合约的交易的原因。
```
await signer.send_transaction(
    account,
    account.contract_address,
    'set_public_key',
    [NEW_KEY]
)
```
或者，如果您想要在AccountRegistry合约上更新账户的L1地址，您可以执行以下操作
```
await signer.send_transaction(account, registry.contract_address, 'set_L1_address', [NEW_ADDRESS])
```

您可以在“账户消息方案讨论”中阅读有关消息结构和哈希的更多信息。有关多调用的设计选择和实现的更多信息，请阅读“账户多调用应如何工作讨论”。

“validate”和“execute”方法接受相同的参数；然而，“execute”方法返回一个交易响应。

```
func __validate__(
    call_array_len: felt, call_array: AccountCallArray*, calldata_len: felt, calldata: felt*) {
}

func __execute__(
    call_array_len: felt, call_array: AccountCallArray*, calldata_len: felt, calldata: felt*
) -> (response_len: felt, response: felt*) {
}
```

在哪里：

* call_array_len是调用数量。

* call_array是表示每个调用的数组。

* calldata_len是calldata参数的数量。

* calldata是表示函数参数的数组。

> NOTE
一旦StarkNet允许在结构数组中使用指针，构建__execute__方法中的多次调用事务的方案将发生变化。在这种情况下，可以将多个事务传递给（而不是在内部构建）__execute__。

## 标准接口
[IAccount.cairo](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/src/openzeppelin/account/IAccount.cairo)合约接口包含了[#41](https://github.com/OpenZeppelin/cairo-contracts/discussions/41)提出的标准账户接口，并被OpenZeppelin和Argent采用。它实现了[EIP-1271](https://eips.ethereum.org/EIPS/eip-1271)，并且与签名验证无关。此外，协议级别处理nonce管理。


```
struct Call {
    to: felt,
    selector: felt,
    calldata_len: felt,
    calldata: felt*,
}

// Tmp struct introduced while we wait for Cairo to support passing `[Call]` to __execute__
struct CallArray {
    to: felt,
    selector: felt,
    data_offset: felt,
    data_len: felt,
}


@contract_interface
namespace IAccount {
    func supportsInterface(interfaceId: felt) -> (success: felt) {
    }

    func isValidSignature(hash: felt, signature_len: felt, signature: felt*) -> (isValid: felt) {
    }

    func __validate__(
        call_array_len: felt, call_array: AccountCallArray*, calldata_len: felt, calldata: felt*
    ) {
    }

    func __validate_declare__(class_hash: felt) {
    }

    func __execute__(
        call_array_len: felt, call_array: AccountCallArray*, calldata_len: felt, calldata: felt*
    ) -> (response_len: felt, response: felt*) {
    }
}
```

## 密钥、签名和签署者
尽管接口与签名验证方案无关，但该实现假设存在一个控制账户的公私钥对。这就是为什么构造函数期望一个public_key参数来设置它。由于还有一个setPublicKey()方法，因此可以有效地转移账户。

### 签名验证
签名验证在Cairo v0.10中与执行分离。在接收到交易时，账户合约首先调用__validate__。只有当签名有效时，账户才会执行交易。这种解耦允许在协议级别上区分无效和回滚的交易。请参阅*账户入口点*。

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

* calldata每个调用的参数列表。

* sig_r交易签名。

* sig_s交易签名。

虽然Signer类执行了大部分发送交易所需的工作，但它既不管理nonce，也不在Account合约上调用实际的交易。为了简化账户管理，大部分工作都通过MockSigner进行了抽象。

### MockSigner实用程序
[signers.py](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/tests/signers.py)中的MockSigner类用于在给定的账户上执行交易，构建交易并管理nonce。

交易的流程始于检查nonce并将每个调用的合约地址转换为十六进制格式。十六进制转换是必要的，因为Nile的Signer将地址转换为一个十六进制整数（需要一个字符串参数）。注意，直接转换为字符串最终会导致一个超过Cairo的FIELD_PRIME的整数。

交易中包含的值被传递给Nile的Signer的sign_transaction方法，该方法创建并返回一个签名。最后，MockSigner实例调用账户合约的__execute__方法来执行交易数据。

> NOTE
StarkNet的测试框架目前不支持从账户合约进行事务调用。因此，MockSigner利用StarkNet的API网关来手动执行
InvokeFunction进行测试。

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
[signers.py](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.6.1/tests/signers.py)中的MockEthSigner类用于使用secp256k1曲线密钥对在给定的账户上执行交易，构建交易并管理nonce。它与MockSigner实现不同之处在于：

* 不使用公钥，而是使用派生的地址（公钥的keccak256哈希的最后20个字节，并在开头添加0x）。

* 使用secp256k1曲线地址对消息进行签名。

### 调用和AccountCallArray格式
这个想法是将所有用户意图编码为表示智能合约调用的Call。用户还可以将多个消息打包到单个交易中（创建多个调用事务）。Cairo目前不支持带有指针的结构体数组，这意味着__execute__函数无法正确迭代多个调用。因此，该实现使用了AccountCallArray结构的解决方法。请参阅*多调用事务*。

#### 调用
单个调用的结构如下：
```
struct Call {
    to: felt
    selector: felt
    calldata_len: felt
    calldata: felt*
}
```

* to是消息的目标合约地址。

* selector是要在目标合约上调用的函数的选择器。

* calldata_len是calldata参数的数量。

* calldata是表示函数参数的数组。

### AccountCallArray
AccountCallArray的结构如下：
```
struct AccountCallArray {
    to: felt
    selector: felt
    data_offset: felt
    data_len: felt
}
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

然后，[Nile的签名者](https://github.com/OpenZeppelin/nile/blob/main/src/nile/signer.py)中的from_call_to_call_array方法将每个调用转换为AccountCallArray格式，并将每个调用的calldata累积存储到一个单独的数组中。接下来，这两个数组（以及发送者、nonce和max_fee）被用来创建交易哈希。然后，签名者使用签名调用_execute_并传递AccountCallArray、calldata和nonce作为参数。

最后，__execute__方法接受AccountCallArray和calldata，并构建一个Calls（MultiCall）数组。

> NOTE
每个交易都使用AccountCallArray。一个单独的Call被视为一个带有一条消息的捆绑。

## API规范
这是Account合约的公共API的简要概述
```
namespace Account {
    func constructor(publicKey: felt) {
    }

    func getPublicKey() -> (publicKey: felt) {
    }

    func supportsInterface(interfaceId: felt) -> (success: felt) {
    }

    func setPublicKey(newPublicKey: felt) {
    }

    func isValidSignature(hash: felt, signature_len: felt, signature: felt*) -> (isValid: felt) {
    }

    func __validate__(
        call_array_len: felt, call_array: AccountCallArray*, calldata_len: felt, calldata: felt*
    ) -> (response_len: felt, response: felt*) {
    }

    func __validate_declare__(
        call_array_len: felt, call_array: AccountCallArray*, calldata_len: felt, calldata: felt*
    ) -> (response_len: felt, response: felt*) {
    }

    func __execute__(
        call_array_len: felt, call_array: AccountCallArray*, calldata_len: felt, calldata: felt*
    ) -> (response_len: felt, response: felt*) {
}
```

### 构造函数
初始化并设置Account合约的公钥。

参数:
```
publicKey: felt
```
返回：无。

### getPublicKey
返回与账户关联的公钥。

参数：无。

返回值：公钥。
```
publicKey: felt
```

### supportsInterface
设置将控制此账户的公钥。它可以用于在安全性方面旋转密钥，更改被破坏密钥的情况，甚至转移账户的所有权。

参数:
```
newPublicKey: felt
```
返回：无。

### isValidSignature
该函数受[EIP-1271](https://eips.ethereum.org/EIPS/eip-1271)的启发，如果给定的签名有效，则返回TRUE，否则会回滚。将来，如果给定的签名无效，它将返回FALSE（有关更多信息，请查[看此问题](https://github.com/OpenZeppelin/cairo-contracts/issues/327)）。

参数:
```
hash: felt
signature_len: felt
signature: felt*
```

返回:
```
isValid: felt
```
> NOTE
如果给定的签名无效，将来可能会返回FALSE（请参[考此问题](https://github.com/OpenZeppelin/cairo-contracts/issues/327)的讨论）。

### __validate_declare__
验证交易签名，并在__执行__之前调用。

参数:
```
call_array_len: felt
call_array: AccountCallArray*
calldata_len: felt
calldata: felt*
```

### __validate_declare__
验证声明交易的签名。

参数:
```
class_hash: felt
```
返回：无。

### __execute__
这是与Account合约进行交互的唯一外部入口。它：

1. 使用预期的函数选择器和calldata参数调用目标合约。

2. 将合约调用的响应数据作为返回值转发。

参数:
```
call_array_len: felt
call_array: AccountCallArray*
calldata_len: felt
calldata: felt*
```

> NOTE
当前的签名方案期望一个包含两个元素的数组，例如 [sig_r, sig_s]。

返回值:
```
response_len: felt
response: felt*
```

## 预设
以下合约预设已准备就绪，可用于快速原型设计和测试。每个预设在使用的账户签名类型上有所不同。

### 账户
[账户](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.6.1/src/openzeppelin/account/presets/Account.cairo)预设使用StarkNet密钥验证交易。

### 以太坊账户
[以太坊账户](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.6.1/src/openzeppelin/account/presets/EthAccount.cairo)预设支持以太坊地址，使用secp256k1密钥验证交易。

## 带ERC165的账户自省
某些合约，如ERC721，需要一种区分账户合约和非账户合约的方法。为了将合约声明为账户，它应该按照[#100](https://github.com/OpenZeppelin/cairo-contracts/discussions/100)中提议的方式实现[ERC165](https://eips.ethereum.org/EIPS/eip-165)。为了符合ERC165规范，我们的想法是计算IAccount的EVM选择器（而不是StarkNet选择器）的异或值。IAccount的结果魔法值是0x50b70dcb。

我们在StarkNet上的ERC165集成受到了OpenZeppelin的Solidity实现*ERC165Storage*的启发，该实现存储了实现合约支持的接口。对于账户合约，使用IAccount的魔法值查询账户地址的supportsInterface应该返回TRUE。

> NOTE
对于账户合约，ERC165支持是静态的，不需要账户合约进行注册。

## 扩展账户合约
账户合约可以通过遵循*可扩展性模式*进行扩展。

要实现自定义账户合约，StarkNet编译器要求其包含三个入口函数__validate__、__validate_declare__和__execute__。

__validate__和__validate_declare__应包含相同的签名验证方法；而__execute__只需处理实际的交易。只需通过__validate__和__validate_declare__调用新的验证方案。

这就是为什么账户库提供了不同类型的签名验证方法，如is_valid_eth_signature和普通的is_valid_signature。

鼓励账户合约开发者实现[标准的账户接口](https://github.com/OpenZeppelin/cairo-contracts/discussions/41)，然后再加入自定义逻辑。

> IMPORTANT
由于测试框架与实际的StarkNet网络之间存在当前的不一致性，当集成新的账户合约时应格外谨慎。已经出现过在本地节点上账户功能测试通过且交易执行正确，但在公共网络上失败的情况。因此，强烈建议在公共测试网络上部署和测试新的账户合约。有关更多信息，请参见问题[#386](https://github.com/OpenZeppelin/cairo-contracts/issues/386)。

未来可能要注意的其他验证方案：

* 多重签名。

* 像[Argent账户](https://github.com/argentlabs/argent-contracts-starknet/blob/de5654555309fa76160ba3d7393d32d2b12e7349/contracts/ArgentAccount.cairo)中的守护逻辑。


## L1逃生机制

## 支付燃料费用