# ERC20

ERC20代币标准是一种对[可互换代币](/Contracts/Contracts.4.x/Tokens/Tokens.md#不同类型的代币)的规范，这种代币的所有单位都完全相等。ERC20.cairo合约在StarkNet上实现了对[EIP-20](https://eips.ethereum.org/EIPS/eip-20)的近似。

# 目录
- [ERC20](#erc20)
- [目录](#目录)
  - [接口](#接口)
    - [ERC20兼容性](#erc20兼容性)
  - [用途](#用途)
  - [可扩展性](#可扩展性)
  - [预设](#预设)
    - [ERC20 (basic)](#erc20-basic)
    - [ERC20Mintable](#erc20mintable)
    - [ERC20Pausable](#erc20pausable)
    - [ERC20Upgradeable](#erc20upgradeable)
  - [API Specification](#api-specification)
    - [方法](#方法)
      - [name](#name)
      - [symbol](#symbol)
      - [decimals](#decimals)
      - [totalSupply](#totalsupply)
      - [balanceOf](#balanceof)
      - [allowance](#allowance)
      - [transfer](#transfer)
      - [transferFrom](#transferfrom)
      - [approve](#approve)
      - [Events](#events)
      - [Transfer (event)](#transfer-event)
      - [Approval (event)](#approval-event)


## 接口
```
@contract_interface
namespace IERC20:
    func name() -> (name: felt):
    end

    func symbol() -> (symbol: felt):
    end

    func decimals() -> (decimals: felt):
    end

    func totalSupply() -> (totalSupply: Uint256):
    end

    func balanceOf(account: felt) -> (balance: Uint256):
    end

    func allowance(owner: felt, spender: felt) -> (remaining: Uint256):
    end

    func transfer(recipient: felt, amount: Uint256) -> (success: felt):
    end

    func transferFrom(
            sender: felt,
            recipient: felt,
            amount: Uint256
        ) -> (success: felt):
    end

    func approve(spender: felt, amount: Uint256) -> (success: felt):
    end
end
```

### ERC20兼容性
尽管StarkNet不兼容EVM，但该实现致力于尽可能接近ERC20标准，具体体现在以下方面：

* 它使用Cairo的uint256代替felt。

* 它返回TRUE作为成功。

* 它在构造函数的calldata中接受一个felt参数来表示小数位数，最大值为2^8（模拟uint8类型）。

* 它利用Cairo的短字符串来模拟名称和符号。

但仍然存在一些差异，例如：

* transfer、transferFrom和approve永远不会返回与TRUE不同的值，因为它们在任何错误时都会回滚。

* [Cairo](https://github.com/starkware-libs/cairo-lang/blob/7712b21fc3b1cb02321a58d0c0579f5370147a8b/src/starkware/starknet/public/abi.py#L25)和[Solidity](https://solidity-by-example.org/function-selector/)之间的函数选择器计算方式不同。

## 用途
使用案例从交换货币到投票权、质押等各种场景。

考虑到构造函数的方法如下：
```
func constructor(
    name: felt,               # Token name as Cairo short string
    symbol: felt,             # Token symbol as Cairo short string
    decimals: felt            # Token decimals (usually 18)
    initial_supply: Uint256,  # Amount to be minted
    recipient: felt           # Address where to send initial supply to
):
```

要创建代币，您需要像这样部署它:
```
erc20 = await starknet.deploy(
    "openzeppelin/token/erc20/presets/ERC20.cairo",
    constructor_calldata=[
        str_to_felt("Token"),     # name
        str_to_felt("TKN"),       # symbol
        18,                       # decimals
        (1000, 0),                # initial supply
        account.contract_address  # recipient
    ]
)
```

正如大多数StarkNet合约一样，它期望被另一个合约调用，并通过get_caller_address（类似于Solidity的this.address）来识别它。这就是为什么我们需要一个Account合约与之交互的原因。例如:

```
signer = MockSigner(PRIVATE_KEY)
amount = uint(100)

account = await starknet.deploy(
    "contracts/Account.cairo",
    constructor_calldata=[signer.public_key]
)

await signer.send_transaction(account, erc20.contract_address, 'transfer', [recipient_address, *amount])
```

## 可扩展性
ERC20合约可以通过遵循[可扩展性模式进行扩展](../Extensibility.md#可扩展性问题)。整合该模式的基本思想是从ERC20库中导入必要的ERC20方法，然后在此基础上加入扩展逻辑。例如，假设您想要实现一个暂停机制。合约应首先从[Pausable库](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/security/pausable/library.cairo)导入ERC20方法和扩展逻辑，即Pausable.pause，Pausable.unpause。接下来，合约应公开具有扩展逻辑的方法，如下所示。

```
@external
func transfer{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(recipient: felt, amount: Uint256) -> (success: felt):
    Pausable.assert_not_paused()          # imported extended logic
    ERC20.transfer(recipient, amount)     # imported library method
    return (TRUE)
end
```

请注意，可扩展性不一定只限于像上面例子中的基于库的方式。例如，具有暂停机制的ERC20合约可以直接在合约中定义暂停方法，甚至可以从库中导入可暂停的方法并进行进一步定制。

扩展ERC20合约的其他方式可能包括：

* 实现铸币机制。

* 创建时间锁。

* 添加角色，如所有者或铸币者。

有关在ERC20合约中使用可扩展性模式的完整示例，请参阅*预设*。

## 预设
以下合约预设已准备好部署，可以直接使用于快速原型设计和测试。每个预设都铸造了一个初始供应量，这对于不公开铸造方法的预设尤其必要。

### ERC20 (basic)
[ERC20](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20.cairo)预设提供了一个快速简便的设置，用于部署基本的ERC20代币。

### ERC20Mintable
[ERC20Mintable](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20Mintable.cairo)预设允许合约所有者铸造新的代币。

### ERC20Pausable
[ERC20Pausable](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20Pausable.cairo)预设允许合约所有者暂停/取消暂停所有修改状态的方法，例如转账、批准等。这个预设在以下情况下非常有用：在评估期结束之前阻止交易，并在出现严重错误时紧急关闭所有代币转移。

### ERC20Upgradeable
[ERC20Upgradeable](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20Upgradeable.cairo)预设允许合约所有者通过部署一个新的ERC20实现合约来升级合约，同时保留合约的状态。这个预设在消除错误和添加新功能等场景中非常有用。有关可升级性的更多信息，请参阅[合约升级](../Proxies-and-Upgrades.md#合约升级)。

## API Specification

### 方法
```
func name() -> (name: felt):
end

func symbol() -> (symbol: felt):
end

func decimals() -> (decimals: felt):
end

func totalSupply() -> (totalSupply: Uint256):
end

func balanceOf(account: felt) -> (balance: Uint256):
end

func allowance(owner: felt, spender: felt) -> (remaining: Uint256):
end

func transfer(recipient: felt, amount: Uint256) -> (success: felt):
end

func transferFrom(
        sender: felt,
        recipient: felt,
        amount: Uint256
    ) -> (success: felt):
end

func approve(spender: felt, amount: Uint256) -> (success: felt):
end
```

#### name
返回代币的名称。

参数：无。

返回值：
```
name: felt
```

#### symbol
返回代币的symbol代码。

参数：无。

返回：
```
symbol: felt
```

#### decimals
返回代币使用的小数位数 - 例如，8表示将代币数量除以100000000以获得其用户表示。

参数：无。

返回值：
```
decimals: felt
```

#### totalSupply
返回存在的代币数量。

参数：无。

返回值：代币数量。
```
totalSupply: Uint256
```

#### balanceOf
返回账户拥有的代币数量。

参数:
```
account: felt
```

返回值：
```
balance: Uint256
```

#### allowance
返回spender将被允许代表owner通过transferFrom进行消费的剩余代币数量。默认情况下，此值为零。

此值在调用approve或transferFrom时会发生变化。

参数:
```
owner: felt
spender: felt
```

返回值：
```
remaining: Uint256
```

#### transfer
将数量代币从调用者的账户转移到接收者的账户。它返回一个表示是否成功的布尔值1。

发出一个[Transfer](#transfer)事件。

参数:
```
recipient: felt
amount: Uint256
```

#### transferFrom
使用津贴机制将数量代币从发送者移动到接收者。然后从调用者的津贴中扣除数量。返回1表示成功。

发出一个[Transfer](#transfer)事件。

参数:
```
sender: felt
recipient: felt
amount: Uint256
```

返回值：
```
success: felt
```

#### approve
设置spender的津贴为caller的代币数量。它返回一个布尔值1，表示是否成功。

发出一个[Approval](#approval-event)事件。

参数:
```
spender: felt
amount: Uint256
```

返回值：
```
success: felt
```

#### Events
```
func Transfer(from_: felt, to: felt, value: Uint256):
end

func Approval(owner: felt, spender: felt, value: Uint256):
end
```

#### Transfer (event)
当价值代币从一个账户（from_）转移到另一个账户（to）时发出。

请注意，价值可能为零。

参数:
```
from_: felt
to: felt
value: Uint256
```

#### Approval (event)
当[approve](#approve)调用批准设置所有者对支出者的准许额度时发出。值是新的准许额度。

参数：
```
owner: felt
spender: felt
value: Uint256
```