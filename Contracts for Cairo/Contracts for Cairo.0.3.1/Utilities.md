# Utilities
以下文档为 tests/utils.py 中的方法和常量提供了上下文、原因和示例。

> CAUTION
预计这个模块会不断发展（就像它已经做过的那样）。

# 目录
* Constants

* Strings

  * str_to_felt

  * felt_to_str

* Uint256

  * uint

  * to_uint

  * from_uint

  * add_uint

  * sub_uint

* Assertions

  * assert_revert

  * assert_revert_entry_point

  * assert_events_emitted

* Memoization

  * get_contract_class

  * cached_contract

* State

* Account

* MockSigner

## Constants
为了提高Cairo合约的可读性，该项目包括可重用的[常量变量](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/utils/constants/library.cairo)，如UINT8_MAX，或者EIP165接口ID，如IERC165_ID或IERC721_ID。有关接口ID如何计算的更多信息，请参阅*ERC165文档*。

## 字符串
Cairo目前仅支持短字符串字面量（少于32个字符）。请注意，短字符串实际上不是字符串，而是Cairo字段元素的表示。以下方法提供了一种简单的字段元素与字符串之间的转换。

### str_to_felt
将ASCII字符串转换为字段元素，采用大端表示法。

### felt_to_str
将一个整数转换为ASCII字符串，通过去除空字节并解码剩余的位。

## Uint256
开罗的本地数据类型是一个字段元素（felt）。字段元素等于252位，这对于256位整数的集成造成了问题。为了解决位数不匹配的问题，开罗将256位整数表示为两个128位整数的结构体。此外，低位位于高位之前，例如。
```
1 = (1, 0)
1 << 128 = (0, 1)
(1 << 128) - 1 = (340282366920938463463374607431768211455, 0)
```

### uint
将一个简单的整数转换为类似于uint256的元组。

注意，应优先使用to_uint而不是uint，因为uint只返回元组的低位。

### to_uint
将一个整数转换为一个类似于uint256的元组。
```
x = to_uint(340282366920938463463374607431768211456)
print(x)
# prints (0, 1)
```

### from_uint
将一个类似于uint256的元组转换为整数。
```
x = (0, 1)
y = from_uint(x)
print(y)
# prints 340282366920938463463374607431768211456
```

### add_uint
对两个uint256-ish元组执行加法，并将和作为uint256-ish元组返回。
```
x = (0, 1)
y = (1, 0)
z = add_uint(x, y)
print(z)
# prints (1, 1)
```

### sub_uint
执行两个uint256-ish元组之间的减法，并将差作为uint256-ish元组返回。
```
x = (0, 1)
y = (1, 0)
z = sub_uint(x, y)
print(z)
# prints (340282366920938463463374607431768211455, 0)
```

### mul_uint
执行两个uint256-ish元组之间的乘法，并将乘积作为uint256-ish元组返回。
```
x = (0, 10)
y = (2, 0)
z = mul_uint(x, y)
print(z)
# prints (0, 20)
```

### div_rem_uint
执行两个uint256-ish元组之间的除法，并分别将商和余数作为uint256-ish元组返回。
```
x = (1, 100)
y = (0, 25)
z = div_rem_uint(x, y)
print(z)
# prints ((4, 0), (1, 0))
```

## Assertions
为了简化StarkNet事务的测试断言的冗长性，该项目包括以下辅助方法。

### assert_revert
一个异步的包装方法，用于执行应该失败的事务的try-except模式。请注意，此包装器不检查StarkNet的错误代码。这样可以更灵活地检查事务是否仅仅失败。如果您想检查确切的错误代码，可以使用StarkNet的[error_codes模块](https://github.com/starkware-libs/cairo-lang/blob/ed6cf8d6cec50a6ad95fa36d1eb4a7f48538019e/src/starkware/starknet/definitions/error_codes.py)，并实现额外的逻辑来assert_revert方法。

要成功使用此包装器，事务方法应该用assert_revert包装；然而，await应该在包装器之前，像这样
```
await assert_revert(signer.send_transaction(
    account, contract.contract_address, 'foo', [
        recipient,
        *token
    ])
)
```

这个包装器还包括检查错误消息是否包含在还原中的选项。要检查还原是否发送了正确的错误消息，请在实际事务之外（但仍在包装器内部）添加reverted_with关键字参数，如下所示。
```
await assert_revert(signer.send_transaction(
    account, contract.contract_address, 'foo', [
        recipient,
        *token
    ]),
    reverted_with="insert error message here"
)
```

### assert_revert_entry_point
assert_revert_entry_point是assert_revert的扩展，它断言给定的invalid_selector参数会导致入口点错误发生。这种断言在检查代理/实现合约时特别有用。要使用assert_revert_entry_point，可以按照以下步骤进行操作。
```
await assert_revert_entry_point(
    signer.send_transaction(
        account, contract.contract_address, 'nonexistent_selector', []
    ),
    invalid_selector='nonexistent_selector'
)
```

### assert_event_emitted
一个辅助方法，用于检查交易收据中的合约发出的事件（from_address），发出的事件本身（name）和发出的参数（data）。使用assert_event_emitted方法。
```
# capture the tx receipt
tx_exec_info = await signer.send_transaction(
    account, contract.contract_address, 'foo', [
        recipient,
        *token
    ])

# insert arguments to assert
assert_event_emitted(
    tx_exec_info,
    from_address=contract.contract_address,
    name='Foo_emitted',
    data=[
        account.contract_address,
        recipient,
        *token
    ]
)
```

## Memoization
记忆化函数可以实现更快速和计算成本更低的计算，这在测试智能合约时非常有益。

### get_contract_class
一个帮助方法，根据合同的名称返回合同类。要捕获合同类，只需将合同的名称作为参数添加，如下所示。
```
contract_class = get_contract_class('ContractName')
```
如果存在多个具有相同名称的合同，则必须使用is_path标志将合同的路径传递给它，而不是使用名称。要传递合同的路径。
```
contract_class = get_contract_class('path/to/Contract.cairo', is_path=True)
```

### cached_contract
一个帮助方法，返回给定合约的缓存状态。建议在缓存状态之前先部署所有相关的合约。在测试模块中，应该使用cached_contract在夹具中实例化每个必需的合约，此时状态已被复制。使用cached_contract的备忘录模式应该类似于以下示例。
```
# get contract classes
@pytest.fixture(scope='module')
def contract_classes():
  foo_cls = get_contract_class('Foo')
  return foo_cls

# deploy contracts
@pytest.fixture(scope='module')
async def foo_init(contract_classes):
    foo_cls = contract_classes
    starknet = await Starknet.empty()
    foo = await starknet.deploy(
        contract_class=foo_cls,
        constructor_calldata=[]
    )
    return starknet.state, foo  # return state and all deployed contracts

# memoization
@pytest.fixture(scope='module')
def foo_factory(contract_classes, foo_init):
    foo_cls = contract_classes                          # contract classes
    state, foo = foo_init                               # state and deployed contracts
    _state = state.copy()                               # copy the state
    cached_foo = cached_contract(_state, foo_cls, foo)  # cache contracts
    return cached_foo                                   # return cached contracts
```

## State
State类为初始化StarkNet状态提供了一个包装器，作为Account类的帮助程序。这个包装器允许Account部署共享相同的初始化状态，而无需显式地将实例化的StarkNet状态传递给Account。初始化状态应该如下所示。
```
from utils import State

starknet = await State.init()
```

## Account
Account类抽象了部署账户的大部分样板代码。使用这个类实例化账户需要先使用State.init()方法实例化StarkNet状态，像这样：
```
from utils import State, Account

starknet = await State.init()
account1 = await Account.deploy(public_key)
account2 = await Account.deploy(public_key)
```
Account类还提供了访问账户合约类的功能，这对于遵循Memoization模式非常有用。要获取账户合约类，可以使用以下方法。
```
fetch_class = Account.get_class
```

## MockSigner
MockSigner用于在给定的账户上使用[Nile的Signer](https://github.com/OpenZeppelin/nile/blob/main/src/nile/signer.py)实例执行交易，构建交易并管理nonce。Signer实例管理签名，并由MockSigner利用Account合约的__execute__方法进行操作。有关更多信息，请参阅*MockSigner实用程序*。
