# Accounts
与以太坊中的账户由私钥派生不同，所有的Starknet账户都是合约。这意味着在Starknet上没有外部拥有账户（Externally Owned Account，EOA）的概念。

相反，该网络具有原生的账户抽象，签名验证在合约级别进行。

关于账户抽象的一般概述，请参阅[Starknet的文档](https://docs.starknet.io/documentation/architecture_and_concepts/Accounts/introduction/)。关于这个话题的更详细的讨论可以在[Starknet Shaman的论坛](https://community.starknet.io/t/starknet-account-abstraction-model-part-1/781)中找到。

> TIP
关于使用和实施的详细信息，请查看API参考部分。

## 标准账户接口
Starknet中的账户是智能合约，因此它们可以像任何其他合约一样被部署和交互，并且可以被扩展以实现任何自定义逻辑。然而，账户是一种特殊类型的合约，用于验证和执行交易。因此，它必须实现一套入口点，协议用于此执行流程。[SNIP-6](https://github.com/ericnordelo/SNIPs/blob/feat/standard-account/SNIPS/snip-6.md)提案定义了一个账户的标准接口，支持这种执行流程和与生态系统中的DApps的互操作性。

### ISRC6 Interface
```
/// Represents a call to a target contract function.
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

/// Standard Account Interface
trait ISRC6 {
    /// Executes a transaction through the account.
    fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;

    /// Asserts whether the transaction is valid to be executed.
    fn __validate__(calls: Array<Call>) -> felt252;

    /// Asserts whether a given signature for a given hash is valid.
    fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}
```

[SNIP-6](https://github.com/ericnordelo/SNIPs/blob/feat/standard-account/SNIPS/snip-6.md)增加了is_valid_signature方法。该方法不被协议使用，但对于DApps来说，它在验证签名的有效性上非常有用，支持像Starknet登录这样的功能。

SNIP-6还规定，符合规定的账户必须按照[SNIP-5](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md)实现SRC5接口，作为通过自我检查来判断一个合约是否为账户的机制。

#### ISRC5 Interface
```
/// Standard Interface Detection
trait ISRC5 {
    /// Queries if a contract implements a given interface.
    fn supports_interface(interface_id: felt252) -> bool;
}
```

[SNIP-6](https://github.com/ericnordelo/SNIPs/blob/feat/standard-account/SNIPS/snip-6.md)兼容的账户在查询ISRC6接口Id时必须返回true。

尽管这些接口不是由协议强制执行的，但我们建议实现它们，以便与生态系统实现互操作性。

### Protocol-level methods
在这一部分，我们将描述协议用于抽象账户的方法。前两个方法是使账户能够用于执行交易所必需的。其余的是可选的：

1. __validate__ 验证要执行的交易的有效性。这通常用于验证签名，但入口点实现可以自定义为具有[一些限制](https://docs.starknet.io/documentation/architecture_and_concepts/Accounts/validate_and_execute/#validate_limitations)的任何验证机制。

2. __execute__ 如果验证成功，则执行交易。

3. __validate_declare__ 类似于__validate__的可选入口点，但用于声明其他合约的交易。

4. __validate_deploy__ 类似于__validate__的可选入口点，但用于*反事实部署*。

> NOTE
尽管这些入口点可供协议在其常规交易流程中使用，但它们也可以像任何其他方法一样被调用。

### 部署账户
在Starknet中，有两种部署智能合约的方式：使用deploy_syscall和进行反事实部署。前者可以通过通用*部署合约（UDC）*轻松完成，该合约包装并公开deploy_syscall，通过常规合约调用提供任意部署。但是，如果你没有账户来调用它，你可能会想使用后者。

要进行反事实部署，你需要实现另一个名为__validate_deploy__的协议级入口点。查看*反事实部署指南*以了解如何操作。