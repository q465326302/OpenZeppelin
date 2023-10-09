# Contracts for Cairo
一个用于StarkNet的Cairo**安全智能合约开发库**，[StarkNet](https://starkware.co/product/starknet/)是一个去中心化的零知识Rollup解决方案。

> WARNING
这个仓库包含了高度实验性的代码。请预期会有快速迭代。**请自行承担使用风险**。

## 安装
该库可以作为[Scarb](https://docs.swmansion.com/scarb)包获取。在进行下一步之前，请按照[此指南](https://docs.swmansion.com/scarb/download.html)在机器上安装Cairo和Scarb，并运行以下命令以检查安装是否成功。
```
scarb --version

scarb 0.7.0 (58cc88efb 2023-08-23)
cairo: 2.2.0 (https://crates.io/crates/cairo-lang-compiler/2.2.0)
sierra: 1.3.0
```

### 设置你的项目
创建一个空目录，并进入它:
```
mkdir my_project/ && cd my_project/
```

初始化一个新的Scarb项目:
```
scarb init
```

我的my_project/的内容现在应该是这样的:
```
ls

Scarb.toml src
```

### 安装库
在项目的Scarb.toml文件中声明它作为依赖项来安装库:
```
[dependencies]
openzeppelin = { git = "https://github.com/OpenZeppelin/cairo-contracts.git", tag = "v0.7.0" }
```

> WARNING
确保标签与目标发布版本匹配。

## 基本用法
这是使用*账户模块*构建账户合约的样子。将代码复制到src/lib.cairo中。
```
#[starknet::contract]
mod MyAccount {
    use openzeppelin::account::Account;
    use openzeppelin::account::account::PublicKeyTrait;
    use openzeppelin::account::interface;
    use openzeppelin::introspection::interface::ISRC5;
    use starknet::account::Call;

    // 此合约使用的存储成员在每个使用`unsafe_state`的导入模块中定义。\
    //将来添加组件后，这种设计将得到改进。
    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(ref self: ContractState, public_key: felt252) {
        let mut unsafe_state = _unsafe_state();
        Account::InternalImpl::initializer(ref unsafe_state, public_key);
    }

    #[external(v0)]
    impl SRC6Impl of interface::ISRC6<ContractState> {
        fn __execute__(self: @ContractState, mut calls: Array<Call>) -> Array<Span<felt252>> {
            Account::SRC6Impl::__execute__(@_unsafe_state(), calls)
        }

        fn __validate__(self: @ContractState, mut calls: Array<Call>) -> felt252 {
            Account::SRC6Impl::__validate__(@_unsafe_state(), calls)
        }

        fn is_valid_signature(
            self: @ContractState, hash: felt252, signature: Array<felt252>
        ) -> felt252 {
            Account::SRC6Impl::is_valid_signature(@_unsafe_state(), hash, signature)
        }
    }

    #[external(v0)]
    impl SRC5Impl of ISRC5<ContractState> {
        fn supports_interface(self: @ContractState, interface_id: felt252) -> bool {
            Account::SRC5Impl::supports_interface(@_unsafe_state(), interface_id)
        }
    }

    #[external(v0)]
    impl PublicKeyImpl of PublicKeyTrait<ContractState> {
        fn get_public_key(self: @ContractState) -> felt252 {
            Account::PublicKeyImpl::get_public_key(@_unsafe_state())
        }

        fn set_public_key(ref self: ContractState, new_public_key: felt252) {
            let mut unsafe_state = _unsafe_state();
            Account::PublicKeyImpl::set_public_key(ref unsafe_state, new_public_key);
        }
    }

    #[external(v0)]
    fn __validate_deploy__(
        self: @ContractState,
        class_hash: felt252,
        contract_address_salt: felt252,
        _public_key: felt252
    ) -> felt252 {
        Account::__validate_deploy__(
            @_unsafe_state(), class_hash, contract_address_salt, _public_key
        )
    }

    #[inline(always)]
    fn _unsafe_state() -> Account::ContractState {
        Account::unsafe_new_contract_state()
    }
}
```

你现在可以编译它了:
```
scarb build
```