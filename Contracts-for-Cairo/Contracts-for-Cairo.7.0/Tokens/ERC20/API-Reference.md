# ERC20
关于ERC20合约的接口和实用程序的参考。

> TIP
要了解ERC20的概述，请阅读我们的*ERC20指南*。

## 核心

### [IERC20](https://github.com/OpenZeppelin/cairo-contracts/blob/cairo-2/src/token/erc20/interface.cairo#L6-L19)
```
use openzeppelin::token::erc20::interface::IERC20;
```

IERC20标准的接口，如[EIP-20](https://eips.ethereum.org/EIPS/eip-20)中定义的。

**FUNCTIONS**
name()

symbol()

decimals()

total_supply()

balance_of()

allowance(owner, spender)

transfer(recipient, amount)

transfer_from(sender, recipient, amount)

approve(spender, amount)

**EVENTS**
Transfer(from, to, value)

Approval(owner, spender, value)

#### Functions

##### name() → felt252
external#
返回令牌的名称。

##### symbol() → felt252
external#
返回代币的符号代码。

##### decimals() → u8
external#
返回代币使用的小数位数 - 例如，8表示将代币数量除以100000000以获得其用户可读的表示。

例如，如果小数位数等于2，那么505个代币的余额应该显示给用户为5.05（505 / 10 ** 2）。

代币通常选择18作为值，模仿以太和魏的关系。这是此函数返回的默认值，除非使用了自定义实现。

> NOTE
这些信息仅用于显示目的：它绝不影响合约的任何算术。

##### total_supply() → u256
external#
返回存在的代币数量。

##### balance_of(account: ContractAddress) → u256
external#
返回账户拥有的代币数量。

##### allowance(owner: ContractAddress, spender: ContractAddress) → u256
external#
返回spender被允许代表所有者通过*transfer_from*花费的剩余代币数量。默认情况下，这个值为零。

当调用*approve*或*transfer_from*时，这个值会改变。

##### transfer(recipient: ContractAddress, amount: u256) → bool
external#
将代币数量从调用者的代币余额转移到to。成功返回true，否则回滚。

触发一个*Transfer*事件。

##### transfer_from(sender: ContractAddress, recipient: ContractAddress, amount: u256) → bool
external#
使用许可机制从发送者移动金额代币到接收者。然后从调用者的许可中扣除金额。成功返回真，否则恢复。

触发一个*Transfer*事件。

##### approve(spender: ContractAddress, amount: u256) → bool
external#
将金额设定为支出者对调用者令牌的允许额度。成功返回true，否则恢复。

发出一个*Approval* 事件。

#### Events

##### Transfer(from: ContractAddress, to: ContractAddress, value: u256)
event#
当价值代币从一个地址（from）移动到另一个地址（to）时发出。

请注意，值可能为零。

##### Approval(owner: ContractAddress, spender: ContractAddress, value: u256)
event#
当为所有者设置支出者的额度时发出。值是新的额度。

### [ERC20](https://github.com/OpenZeppelin/cairo-contracts/blob/cairo-2/src/token/erc20/erc20.cairo)
```
use openzeppelin::token::erc20::ERC20;
```

实现IERC20接口。

**CONSTRUCTOR**
constructor(self, name, symbol, initial_supply, recipient)

**EXTERNAL FUNCTIONS**
ERC20IMPL
name(self)

symbol(self)

decimals(self)

total_supply(self)

balance_of(self, account)

allowance(self, owner, spender)

transfer(self, recipient, amount)

transfer_from(self, sender, recipient, amount)

approve(self, spender, amount)

NON-STANDARD
increase_allowance(self, spender, added_value)

decrease_allowance(self, spender, subtracted_value)

ERC20CAMELONLYIMPL
totalSupply(self)

balanceOf(self, account)

transferFrom(self, sender, recipient, amount)

increaseAllowance(self, spender, addedValue)

decreaseAllowance(self, spender, subtractedValue)

**INTERNAL FUNCTIONS**
INTERNALIMPL
initializer(self, name, symbol)

_transfer(self, sender, recipient, amount)

_approve(self, owner, spender, amount)

_mint(self, recipient, amount)

_burn(self, account, amount)

_increase_allowance(self, spender, added_value)

_decrease_allowance(self, spender, subtracted_value)

_spend_allowance(self, owner, spender, amount)

**EVENTS**
Transfer(from, to, value)

Approval(owner, spender, value)

#### Constructor

##### constructor(ref self: ContractState, name: felt252, symbol: felt252, initial_supply: u256, recipient: ContractAddress)
constructor#
设置代币名称和符号，并将初始供应量铸造给接收者。请注意，一旦通过构造函数设置，代币名称和符号就无法更改。

#### External functions

##### name(@self: ContractState) → felt252
external#
请查阅 *IERC20::name*.

##### symbol(@self: ContractState) → felt252
external#
请查阅 *IERC20::symbol*.

##### decimals(@self: ContractState) → u8
external#
请查阅 *IERC20::decimals*.

##### total_supply(@self: ContractState) → u256
external#
请查阅 *IERC20::balance_of*.

##### allowance(@self: ContractState, owner: ContractAddress, spender: ContractAddress) → u256
external#
请查阅 *IERC20::allowance*.

##### transfer(ref self: ContractState, recipient: ContractAddress, amount: u256) → bool
external#
请查阅*IERC20 :: transfer*。

要求：
* 接收者不能是零地址。

* 调用者必须至少拥有金额的余额。

##### transfer_from(ref self: ContractState, sender: ContractAddress, recipient: ContractAddress, amount: u256) → bool
external#
请查阅*IERC20 :: transfer_from*。

要求：
* 发送者不能是零地址。

* 发送者必须至少拥有数量的余额。

* 接收者不能是零地址。

* 调用者必须至少有发送者代币的数量的额度。

##### approve(ref self: ContractState, spender: ContractAddress, amount: u256) → bool
external#
请查阅*IERC20::approve*。

要求：
* 授权者不能是零地址。
  
##### increase_allowance(ref self: ContractState, spender: ContractAddress, added_value: u256) → bool
external#
增加从呼叫者到调用者的允许额度added_value。成功返回true，否则回滚。

发出一个*Approval*事件。

要求：
* 调用者不能是零地址。

##### decrease_allowance(ref self: ContractState, spender: ContractAddress, subtracted_value: u256) → bool
external#
减少从调用者到支出者的允许额度，减少的值为subtracted_value。成功返回true。

触发一个*Approval*事件。

要求：
* 支出者不能是零地址。

* 支出者必须至少有调用者的subtracted_value的额度。

##### totalSupply(self: @ContractState) → u256
external#
查看*IERC20::total_supply*。

支持在[此处](https://github.com/OpenZeppelin/cairo-contracts/discussions/34)讨论的Cairo v0约定，将外部方法以camelCase。

##### balanceOf(self: @ContractState, account: ContractAddress) → u256
external#
请查阅IERC20::balance_of。

支持在[此处](https://github.com/OpenZeppelin/cairo-contracts/discussions/34)讨论的Cairo v0约定，将外部方法写成camelCase。

##### transferFrom(ref self: ContractState, sender: ContractAddress, recipient: ContractAddress) → bool
external#
请查阅*IERC20::transfer_from*。

支持在[此处](https://github.com/OpenZeppelin/cairo-contracts/discussions/34)讨论的Cairo v0约定，将外部方法写成camelCase。

##### increaseAllowance(ref self: ContractState, spender: ContractAddress, addedValue: u256) → bool
external#
请查阅*increase_allowance*。

支持在[此处](https://github.com/OpenZeppelin/cairo-contracts/discussions/34)讨论的Cairo v0约定，即以camelCase编写外部方法。

##### decreaseAllowance(ref self: ContractState, spender: ContractAddress, subtractedValue: u256) → bool
external#
请查阅*decrease_allowanc*e。

支持在[此处](https://github.com/OpenZeppelin/cairo-contracts/discussions/34)讨论的Cairo v0约定，即以camelCase编写外部方法。

#### Internal functions

##### initializer(ref self: ContractState, name: felt252, symbol: felt252)
internal#
通过设置代币名称和符号来初始化合约。这应在合约的构造函数中使用。

##### _transfer(ref self: ContractState, sender: ContractAddress, recipient: ContractAddress, amount: u256)
internal#
将代币数量从一个地址转移到另一个地址。

这个内部函数不检查访问权限，但可以作为构建块，例如实现自动代币费用，削减机制等。

发出一个*Transfer*事件。

要求：
* from不能是零地址。

* to不能是零地址。

* from必须拥有至少数量的余额。

##### _approve(ref self: ContractState, owner: ContractAddress, spender: ContractAddress, amount: u256)
internal#
设置金额作为支出者对所有者代币的津贴。

这个内部函数不检查访问权限，但可以作为构建块，例如实现其他地址的自动津贴。

触发一个Approval 事件。

要求：
* 所有者不能是零地址。

* 支出者不能是零地址。

##### _mint(ref self: ContractState, recipient: ContractAddress, amount: u256)
internal#
创建一定数量的代币并将它们分配给接收者。

发出一个*Transfer*事件，其中来自地址为零。

要求：
* 接收者不能是零地址。

##### _burn(ref self: ContractState, account: ContractAddress, amount: u256)
internal#
从账户中销毁数量的代币。

发出一个*Transfer*事件，将to设置为零地址。

要求：
* 账户不能是零地址。

##### _increase_allowance(ref self: ContractState, spender: ContractAddress, added_value: u256)
internal#
增加了从呼叫者到花费者的额度，增加值为added_value。

触发一个*Approval*事件。

##### _decrease_allowance(ref self: ContractState, spender: ContractAddress, subtracted_value: u256)
internal#
减少从呼叫者到花费者的允许额度，减少的值为subtracted_value

触发一个Approval事件。

##### _spend_allowance(ref self: ContractState, owner: ContractAddress, spender: ContractAddress, amount: u256)
internal#
根据已花费的金额更新所有者对支出者的津贴。

如果津贴无限，此内部函数不会更新津贴值。

可能会发出一个*Approval*事件。

#### Events

##### Transfer(from: ContractAddress, to: ContractAddress, value: u256)
event#
请查阅 *IERC20::Transfer*.

##### Approval(owner: ContractAddress, spender: ContractAddress, value: u256)
event#
请查阅 *IERC20::Approval*.