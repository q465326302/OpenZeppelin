# Interfaces and Dispatchers
这一部分描述了OpenZeppelin Contracts for Cairo提供的接口，并解释了其背后的设计选择。

接口可以在模块树的接口子模块下找到，例如token::erc20::interface。例如:
```
use openzeppelin::token::erc20::interface::IERC20;
```

和
```
use openzeppelin::token::erc20::dual20::DualCaseERC20;
```

> NOTE
为了简化，我们将以ERC20为例，但同样的概念也适用于其他模块。

## 接口特性
库提供了三种类型的特性来实现或与合约进行交互：

### 标准特性
这些特性与预定义的接口（如标准）相关联。这只包括接口中定义的函数，是与符合标准的合约进行交互的标准方式。
```
#[starknet::interface]
trait IERC20<TState> {
    fn name(self: @TState) -> felt252;
    fn symbol(self: @TState) -> felt252;
    fn decimals(self: @TState) -> u8;
    fn total_supply(self: @TState) -> u256;
    fn balance_of(self: @TState, account: ContractAddress) -> u256;
    fn allowance(self: @TState, owner: ContractAddress, spender: ContractAddress) -> u256;
    fn transfer(ref self: TState, recipient: ContractAddress, amount: u256) -> bool;
    fn transfer_from(
        ref self: TState, sender: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool;
    fn approve(ref self: TState, spender: ContractAddress, amount: u256) -> bool;
}
```

### ABI特性
它们描述了合约的完整接口。这对于与这个库提供的预设合约进行接口是非常有用的，比如包含了非标准函数如increase_allowance的ERC20预设。
```
#[starknet::interface]
trait ERC20ABI<TState> {
    fn name(self: @TState) -> felt252;
    fn symbol(self: @TState) -> felt252;
    fn decimals(self: @TState) -> u8;
    fn total_supply(self: @TState) -> u256;
    fn balance_of(self: @TState, account: ContractAddress) -> u256;
    fn allowance(self: @TState, owner: ContractAddress, spender: ContractAddress) -> u256;
    fn transfer(ref self: TState, recipient: ContractAddress, amount: u256) -> bool;
    fn transfer_from(
        ref self: TState, sender: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool;
    fn approve(ref self: TState, spender: ContractAddress, amount: u256) -> bool;
    fn increase_allowance(ref self: TState, spender: ContractAddress, added_value: u256) -> bool;
    fn decrease_allowance(
        ref self: TState, spender: ContractAddress, subtracted_value: u256
    ) -> bool;
}
```

### 调度程序特性
这是一个实用的特性，用于与未知接口的合约进行接口。在*双重情况调度器部分*中阅读更多信息。
```
#[derive(Copy, Drop)]
struct DualCaseERC20 {
    contract_address: ContractAddress
}

trait DualCaseERC20Trait {
    fn name(self: @DualCaseERC20) -> felt252;
    fn symbol(self: @DualCaseERC20) -> felt252;
    fn decimals(self: @DualCaseERC20) -> u8;
    fn total_supply(self: @DualCaseERC20) -> u256;
    fn balance_of(self: @DualCaseERC20, account: ContractAddress) -> u256;
    fn allowance(self: @DualCaseERC20, owner: ContractAddress, spender: ContractAddress) -> u256;
    fn transfer(self: @DualCaseERC20, recipient: ContractAddress, amount: u256) -> bool;
    fn transfer_from(
        self: @DualCaseERC20, sender: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool;
    fn approve(self: @DualCaseERC20, spender: ContractAddress, amount: u256) -> bool;
}
```

## 双重接口
根据大规模[接口迁移](https://community.starknet.io/t/the-great-interface-migration/92107)计划，我们在所有现有的snake_case（camelCase）合约中添加了蛇形命名法（snake_case）函数，目标是最终停止支持前者。

简单来说，我们提供两种类型的接口和处理它们的工具：

1.camelCase接口，这是我们迄今为止一直在使用的。

2.snake_case接口，这是我们正在迁移到的。

这意味着目前我们的大多数合约实现了双接口。例如，ERC20预设合约公开了transferFrom、transfer_from、balanceOf、balance_of等。

> NOTE
所有在OpenZeppelin Contracts for Cairo的早期版本（v0.6.1及以下）中存在的外部函数都有双接口可用。

### IERC20
ERC20接口特性的默认版本公开了snake_case函数:
```
#[starknet::interface]
trait IERC20<TState> {
    fn name(self: @TState) -> felt252;
    fn symbol(self: @TState) -> felt252;
    fn decimals(self: @TState) -> u8;
    fn total_supply(self: @TState) -> u256;
    fn balance_of(self: @TState, account: ContractAddress) -> u256;
    fn allowance(self: @TState, owner: ContractAddress, spender: ContractAddress) -> u256;
    fn transfer(ref self: TState, recipient: ContractAddress, amount: u256) -> bool;
    fn transfer_from(
        ref self: TState, sender: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool;
    fn approve(ref self: TState, spender: ContractAddress, amount: u256) -> bool;
}
```

## IERC20Camel
除此之外，我们还提供了同一接口的camelCase版本。
```
#[starknet::interface]
trait IERC20Camel<TState> {
    fn name(self: @TState) -> felt252;
    fn symbol(self: @TState) -> felt252;
    fn decimals(self: @TState) -> u8;
    fn totalSupply(self: @TState) -> u256;
    fn balanceOf(self: @TState, account: ContractAddress) -> u256;
    fn allowance(self: @TState, owner: ContractAddress, spender: ContractAddress) -> u256;
    fn transfer(ref self: TState, recipient: ContractAddress, amount: u256) -> bool;
    fn transferFrom(
        ref self: TState, sender: ContractAddress, recipient: ContractAddress, amount: u256
    ) -> bool;
    fn approve(ref self: TState, spender: ContractAddress, amount: u256) -> bool;
}
```

## DualCase调度程序

> WARNING
DualCase调度器在实现运行时的恐慌处理之前，将无法在实时链（主网或测试网）上工作。调度器在测试环境中运行良好。

为了简化这个过渡，OpenZeppelin为Cairo合约提供了我们所说的双重情况调度器，如DualCaseERC721或DualCaseAccount。

这些模块用兼容性层包装一个目标合约，无论底层合约使用什么样的大小写，都能暴露出一个snake_case接口。这样，一个AMM就不会因为他们的接口而在集成代币时遇到问题。

例如:
```
let token = DualCaseERC20 { contract_address: target };
token.transfer_from(OWNER(), RECIPIENT(), VALUE);
```

这是通过执行函数的snake_case版本（例如transfer_from）并在其以ENTRYPOINT_NOT_FOUND回退时回退到camelCase版本（例如transferFrom）来完成的，如下所示:
```
fn try_selector_with_fallback(
    target: ContractAddress, snake_selector: felt252, camel_selector: felt252, args: Span<felt252>
) -> SyscallResult<Span<felt252>> {
    match call_contract_syscall(target, snake_selector, args) {
        Result::Ok(ret) => Result::Ok(ret),
        Result::Err(errors) => {
            if *errors.at(0) == 'ENTRYPOINT_NOT_FOUND' {
                return call_contract_syscall(target, camel_selector, args);
            } else {
                Result::Err(errors)
            }
        }
    }
}
```

首先尝试snake_case接口会使camelCase调用稍微昂贵一些，因为在camelCase调用之前总会先进行一次失败的snake_case调用。这是一个设计选择，旨在鼓励采用/过渡到snake_case，符合大型[接口迁移](https://community.starknet.io/t/the-great-interface-migration/92107)的要求。