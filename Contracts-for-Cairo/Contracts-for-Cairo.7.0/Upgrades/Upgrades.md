# Upgrades
在不同的区块链中，已经开发出多种使合约可升级的模式，包括广泛采用的代理模式。

Starknet通过一个系统调用（syscall）原生支持可升级性，该系统调用更新合约的源代码，从而无需代理。

## 替换合约类别
为了更好地理解Starknet中的可升级性如何工作，理解合约和其合约类别之间的区别非常重要。

[合约类别](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/contract-classes/)代表程序的源代码。所有合约都与一个类别相关联，许多合约可以是同一类别的实例。类别通常由[类别哈希](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/class-hash/)表示，而在部署一个类别的合约之前，需要声明类别哈希。

### replace_class_syscall
[replace_class](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/system-calls-cairo1/#replace_class)系统调用允许一个已部署的合约通过替换其类别哈希来更新其源代码。
```
/// Upgrades the contract source code to the new contract class.
fn _upgrade(new_class_hash: ClassHash) {
    assert(!new_class_hash.is_zero(), 'Class hash cannot be zero');
    starknet::replace_class_syscall(new_class_hash).unwrap();
}
```

> NOTE
如果一个合约在没有这种机制的情况下被部署，其类哈希仍然可以通过[库调用](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/system-calls-cairo1/#library_call)被替换。

## 可升级模块
OpenZeppelin为Cairo合约提供了[Upgradeable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/upgrades/upgradeable.cairo)，以增加对你的合约的可升级支持。

### 使用
升级通常是非常敏感的操作，通常需要一些形式的访问控制，以避免未经授权的升级。在这个例子中，我们使用了*Ownable*模块。

> NOTE
我们将使用以下模块来实现在API参考部分描述的*IUpgradeable*接口。

```
#[starknet::contract]
mod UpgradeableContract {
    use openzeppelin::access::ownable::Ownable;
    use openzeppelin::upgrades::Upgradeable;
    use openzeppelin::upgrades::interface::IUpgradeable;
    use starknet::ClassHash;
    use starknet::ContractAddress;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(self: @ContractState, owner: ContractAddress) {
        let mut unsafe_state = Ownable::unsafe_new_contract_state();
        Ownable::InternalImpl::initializer(ref unsafe_state, owner);
    }

    #[external(v0)]
    impl UpgradeableImpl of IUpgradeable<ContractState> {
        fn upgrade(ref self: ContractState, new_class_hash: ClassHash) {
            // This function can only be called by the owner
            let ownable_state = Ownable::unsafe_new_contract_state();
            Ownable::InternalImpl::assert_only_owner(@ownable_state);

            // Replace the class hash upgrading the contract
            let mut upgradeable_state = Upgradeable::unsafe_new_contract_state();
            Upgradeable::InternalImpl::_upgrade(ref upgradeable_state, new_class_hash);
        }
    }

    (...)
}
```

## 代理和Starknet
代理使得可以实现各种模式，如升级和克隆。但由于Starknet以不同的方式实现了同样的功能，因此不支持实现它们。

在合约升级的情况下，只需改变合约的类哈希就可以实现。至于克隆，合约已经像实现它们的类的克隆一样。

在Starknet中实现代理模式有一个重要的限制：没有用于将每个可能的函数调用重定向到实现的回退机制。这意味着不能实现一个通用的代理合约。相反，一个有限的代理合约可以实现特定的函数，将它们的执行转发到另一个合约类。这仍然可以用于例如升级一些函数的逻辑。