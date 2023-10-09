# Introspection
为了顺利实现互操作性，通常标准要求智能合约实现[内省机制](https://en.wikipedia.org/wiki/Type_introspection)。

在以太坊中，[EIP165](https://eips.ethereum.org/EIPS/eip-165)标准定义了合约应如何声明对给定接口的支持，以及其他合约如何查询这种支持。

Starknet提供了由[SRC5](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md)标准定义的类似的接口内省机制。

## SRC5
与其以太坊的对应物类似，[SRC5](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md)标准要求合约实现supports_interface函数，其他合约可以使用它来查询是否支持给定的接口。
```
trait ISRC5 {
    /// 查询合约是否实现了接口。
    /// 接收SRC-5中指定的接口标识符。
    /// 如果合约实现了interface_id，则返回true，否则返回false。
    fn supports_interface(interface_id: felt252) -> bool;
}
```

### Computing the interface ID
接口ID，按照标准规定，是所有接口的[扩展函数选择器](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md#extended-function-selector)的[XOR](https://en.wikipedia.org/wiki/Exclusive_or)。我们强烈建议阅读SNIP以理解计算这些扩展功能选择器的具体细节。有一些工具，如[src5-rs](https://github.com/ericnordelo/src5-rs)，可以帮助进行这个过程。

### 注册接口
为了让一个合约声明其对给定接口的支持，该合约应该导入SRC5模块并注册其支持。建议在合约部署时通过构造函数直接或间接（作为初始化器）注册接口支持，如下所示：
```
#[starknet::contract]
mod MyContract {
    use openzeppelin::account::interface;
    use openzeppelin::introspection::src5::SRC5;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(ref self: ContractState) {
        let mut unsafe_state = SRC5::unsafe_new_contract_state();
        SRC5::InternalImpl::register_interface(ref unsafe_state, interface::ISRC6_ID);
    }

    (...)
}
```

### 查询接口
使用supports_interface函数来查询合约对给定接口的支持。
```
#[starknet::contract]
mod MyContract {
    use openzeppelin::account::interface;
    use openzeppelin::introspection::interface::ISRC5DispatcherTrait;
    use openzeppelin::introspection::interface::ISRC5Dispatcher;
    use openzeppelin::introspection::src5::SRC5;
    use starknet::ContractAddress;

    #[storage]
    struct Storage {}

    #[external(v0)]
    fn query_is_account(self: @ContractState, target: ContractAddress) -> bool {
        let dispatcher = ISRC5Dispatcher { contract_address: target };
        dispatcher.supports_interface(interface::ISRC6_ID)
    }
}
```

> TIP
如果你不确定一个合约是否实现了SRC5，你可以按照[这里](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md#how-to-detect-if-a-contract-implements-src-5)描述的过程进行操作。