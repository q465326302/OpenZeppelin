# ERC20
ERC20代币标准是一种*可替代化代币*的规范，这种类型的代币所有单位都完全等于彼此。token::erc20::ERC20合约在Cairo的Starknet中实现了[EIP-20](https://eips.ethereum.org/EIPS/eip-20)的近似。

> WARNING
在[Contracts v0.7.0](https://github.com/OpenZeppelin/cairo-contracts/releases/tag/v0.7.0)之前，ERC20合约从存储中存储和读取小数点；然而，这个实现返回一个静态的18。如果升级一个旧的ERC20合约，其小数点值不是18，那么升级后的合约必须使用自定义的小数点实现。请参阅*自定义小数点指南*。

## 接口
以下接口代表了Cairo ERC20预设的完整ABI。该接口包括IERC20标准接口以及非标准函数，如increase_allowance。为了支持旧的ERC20部署，如*双重接口*中所述，ERC20还包括以camelCase的先前定义的函数。
```
trait ERC20ABI {
    // IERC20
    fn name() -> felt252;
    fn symbol() -> felt252;
    fn decimals() -> u8;
    fn total_supply() -> u256;
    fn balance_of(account: ContractAddress) -> u256;
    fn allowance(owner: ContractAddress, spender: ContractAddress) -> u256;
    fn transfer(recipient: ContractAddress, amount: u256) -> bool;
    fn transfer_from(
        sender: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool;
    fn approve(spender: ContractAddress, amount: u256) -> bool;

    // 非标准的
    fn increase_allowance(spender: ContractAddress, added_value: u256) -> bool;
    fn decrease_allowance(spender: ContractAddress, subtracted_value: u256) -> bool;

    // Camel兼容性案例
    fn totalSupply() -> u256;
    fn balanceOf(account: ContractAddress) -> u256;
    fn transferFrom(
        sender: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool;
    fn increaseAllowance(spender: ContractAddress, addedValue: u256) -> bool;
    fn decreaseAllowance(spender: ContractAddress, subtractedValue: u256) -> bool;
}
```

### ERC20 兼容性
虽然Starknet不兼容EVM，但这个实现的目标是尽可能接近ERC20标准。然而，仍然可以发现一些显著的差异，例如：

* Cairo短字符串被用来模拟名称和符号，因为Cairo还没有包含原生字符串支持。

* 合约提供了一个*双重接口*，支持snake_case和camelCase方法，而不仅仅是Solidity中的camelCase。

* transfer、transfer_from和approve永远不会返回任何不同于true的东西，因为它们会在任何错误上回滚。

* [Cairo](https://github.com/starkware-libs/cairo/blob/7dd34f6c57b7baf5cd5a30c15e00af39cb26f7e1/crates/cairo-lang-starknet/src/contract.rs#L39-L48)和[Solidity](https://solidity-by-example.org/function-selector/)之间的函数选择器的计算方式不同。

## 使用
> WARNING
以下示例使用unsafe_new_contract_state来访问另一个合约的状态。虽然这对于将它们用作模块很有用，但它被认为是不安全的，因为如果不仔细审查，存储成员可能会在使用的合约之间冲突。在引入[组件](https://community.starknet.io/t/cairo-1-contract-syntax-is-evolving/94794#extensibility-and-components-11)后，将重新审视可扩展性。

使用Cairo的合约，构建一个ERC20合约需要设置构造函数并暴露ERC20接口。以下是这个过程的样子:
```
#[starknet::contract]
mod MyToken {
    use starknet::ContractAddress;
    use openzeppelin::token::erc20::ERC20;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(
        self: @ContractState,
        initial_supply: u256,
        recipient: ContractAddress
    ) {
        let name = 'MyToken';
        let symbol = 'MTK';

        let mut unsafe_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref unsafe_state, name, symbol);
        ERC20::InternalImpl::_mint(ref unsafe_state, recipient, initial_supply);
    }

    #[external(v0)]
    impl MyTokenImpl of IERC20<ContractState> {
        fn name(self: @ContractState) -> felt252 {
            let mut unsafe_state = ERC20::unsafe_new_contract_state();
            ERC20::ERC20Impl::name(@unsafe_state)
        }

        (...)
    }
}
```

为了让MyToken合约扩展ERC20合约，它使用了unsafe_new_contract_state。不安全的合约状态允许访问ERC20的实现。有了这个访问权限，构造函数首先调用初始化器来设置代币的名称和符号。然后，构造函数调用_mint来创建固定供应的代币。

> TIP
有关ERC20代币机制的更完整指南，请参阅*创建ERC20供应*。

在外部实现中，注意到MyTokenImpl是IERC20的一个实现。这确保了外部实现将包括接口中定义的所有方法。

### 自定义小数
Cairo和Solidity一样，不支持[浮点数](https://en.wikipedia.org//wiki/Floating-point_arithmetic)。为了解决这个限制，ERC20提供了一个小数字段，该字段向外部接口（钱包，交易所等）传达了代币应该如何显示。例如，假设一个代币的小数值为3，总代币供应量为1234。一个外部接口会显示代币供应量为1.234。然而，在实际的合约中，供应量仍然是整数1234。换句话说，**小数字段并不改变实际的算术**，因为所有的操作仍然是在整数上进行的。

大多数合约使用18个小数，甚至有提议将其强制执行（参见[EIP讨论](https://github.com/ethereum/EIPs/issues/724)）。Cairo的ERC20实现默认返回18个小数，以节省燃气费。对于那些想要一个可配置小数数量的ERC20代币的人，以下指南展示了两种实现这一目标的方法。

#### 静态方法
自定义小数的最简单方法是在暴露小数方法时返回目标值。例如:
```
#[external(v0)]
impl MyTokenImpl of IERC20<ContractState> {
    fn decimals(self: @ContractState) -> u8 {
        // 将下面的`3`更改为所需的小数位数
        3
    }

    (...)
}
```

#### 对于存储方法
对于更复杂的情况，例如一个工厂部署多个具有不同小数值的令牌，可能需要一个灵活的解决方案。
```
#[starknet::contract]
mod MyToken {
    use starknet::ContractAddress;
    use openzeppelin::token::erc20::ERC20;

    #[storage]
    struct Storage {
        // 小数值被本地存储
        _decimals: u8,
    }

    #[constructor]
    fn constructor(
        ref self: ContractState,
        decimals: u8
    ) {
        // 调用将小数写入存储的内部函数
        self._set_decimals(decimals);

        // 初始化 ERC20
        let name = 'MyToken';
        let symbol = 'MTK';

        let mut unsafe_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref unsafe_state, name, symbol);
    }

    /// 这是一个简洁的独立函数。
    /// 建议创建IERC20的实现，以确保合约暴露了完整的ERC20接口。
    /// 请参见前面的例子。
    #[external(v0)]
    fn decimals(self: @ContractState) -> u8 {
        self._decimals.read()
    }

    #[generate_trait]
    impl InternalImpl of InternalTrait {
        fn _set_decimals(ref self: ContractState, decimals: u8) {
            self._decimals.write(decimals);
        }
    }
}
```

该合约在构造函数中需要一个小数参数，并使用内部函数将小数写入存储。请注意，_decimals状态变量必须存储在本地合约的存储中，因为这个变量在Cairo合约库中不存在。在公开的decimals方法中包含正确的逻辑非常重要，并且在这种特定情况下不要使用Cairo合约库的decimals实现。库的decimals实现不会从存储中读取，将返回18。