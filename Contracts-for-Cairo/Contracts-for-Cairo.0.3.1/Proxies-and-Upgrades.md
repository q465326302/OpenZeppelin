# Proxies
> NOTE
随着这种模式的成熟和可能出现更多的模式，预计会进行快速迭代。

## 目录
- [Proxies](#proxies)
  - [目录](#目录)
  - [快速入门](#快速入门)
  - [Solidity/Cairo升级比较](#soliditycairo升级比较)
    - [构造函数](#构造函数)
    - [存储](#存储)
  - [代理](#代理)
    - [代理合约](#代理合约)
    - [实现合约](#实现合约)
  - [升级库API](#升级库api)
    - [方法](#方法)
      - [初始化器](#初始化器)
      - [assert\_only\_admin](#assert_only_admin)
      - [get\_implementation](#get_implementation)
      - [get\_admin](#get_admin)
      - [\_set\_admin](#_set_admin)
      - [\_set\_implementation\_hash](#_set_implementation_hash)
    - [Events](#events)
      - [升级](#升级)
      - [AdminChanged](#adminchanged)
  - [使用代理](#使用代理)
    - [合约升级](#合约升级)
    - [声明合约](#声明合约)
    - [测试方法调用](#测试方法调用)
  - [预设](#预设)

## 快速入门
一般的工作流程如下：

1. 声明一个实现[合约类](https://starknet.io/docs/hello_starknet/intro.html#declare-the-contract-on-the-starknet-testnet)

2. 部署代理合约，将实现合约的类哈希设置在代理的构造函数调用数据中

3. 通过向代理合约发送调用来初始化实现合约。这将重定向调用到实现合约类，并表现得像实现合约的构造函数一样

在Python中，这将如下所示：
```
    # declare implementation contract
    IMPLEMENTATION = await starknet.declare(
        "path/to/implementation.cairo",
    )

    # deploy proxy
    PROXY = await starknet.deploy(
        "path/to/proxy.cairo",
        constructor_calldata=[
            IMPLEMENTATION.class_hash,  # set implementation contract class hash
        ]
    )

    # users should only interact with the proxy contract
    await signer.send_transaction(
        account, PROXY.contract_address, 'initializer', [
            proxy_admin
        ]
    )
```

## Solidity/Cairo升级比较

### 构造函数
Solidity的OpenZeppelin合约需要使用另一个库来实现可升级合约。需要注意的是，在Solidity中，构造函数不是部署合约的运行时字节码的一部分；相反，构造函数的逻辑仅在合约实例部署时执行一次，然后被丢弃。这就是为什么代理无法模拟其实现的构造过程，因此需要一个不同的初始化机制。

可升级合约中的构造函数问题通过使用初始化方法来解决。初始化方法本质上是常规方法，执行本应在构造函数中的逻辑。需要小心处理初始化方法，以确保它们只能被调用一次。因此，OpenZeppelin提供了一个[可升级合约库](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable)，其中大部分过程都被抽象化了。详细信息请参阅[OpenZeppelin的编写可升级合约](/Upgrades-Plugins/Writing-Upgradeable-Contracts.md)。

Cairo编程语言不支持继承。相反，Cairo合约遵循[可扩展性模式](./Extensibility.md)，该模式已经使用初始化方法来模拟构造函数。可升级合约因此不需要一个单独的库来重构构造函数逻辑。

### 存储
OpenZeppelin的替代升级库还实现了[非结构化存储](/Upgrades-Plugins/Proxy-Upgrade-Pattern.md#非结构化存储代理)。非结构化存储的基本思想是将可升级合约的存储结构伪随机化，使其基于变量名而不是声明顺序，从而在升级过程中几乎不会发生存储冲突。

与此同时，StarkNet编译器默认情况下通过对存储变量名（以及映射中的键）进行哈希运算来创建伪随机存储地址。换句话说，StarkNet已经使用了非结构化存储，不需要第二个库来修改存储设置。有关更多信息，请参阅[StarkNet的合约存储文档](https://starknet.io/documentation/contracts/#contracts_storage)。

## 代理
代理合约是将函数调用委托给另一个合约的合约。这种模式将状态和逻辑解耦。代理合约存储状态并将函数调用重定向到处理逻辑的实现合约。这允许不同的模式，例如升级，其中实现合约可以更改，但代理合约（因此状态）不会更改；以及部署多个指向相同实现的代理实例。这对于部署具有相同逻辑但具有唯一初始化数据的多个合约非常有用。

在合约升级的情况下，只需将代理的引用更改为声明的实现类的类哈希即可。这允许开发人员添加功能，更新逻辑和修复错误，而无需触及状态或与应用程序交互的合约地址。

### 代理合约
[代理合约](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/upgrades/presets/Proxy.cairo)包括两个核心方法：

1. __default__方法是一个回退方法，将函数调用和关联的calldata重定向到实现合约。

2. __l1_default__方法也是一个回退方法；但是，它将函数调用和关联的calldata重定向到第一层合约。为了调用__l1_default__，原始函数调用必须包括库函数send_message_to_l1。有关更多信息，请参阅[Cairo与L1合约的交互](https://www.cairo-lang.org/docs/hello_starknet/l1l2.html)。

由于此代理旨在同时作为[UUPS风格的升级代理](https://eips.ethereum.org/EIPS/eip-1822)和非可升级代理工作，因此它不知道如何处理自己的状态。因此，它要求在构造时先声明实现合约类，以便将其类哈希传递给代理。

在与合约交互时，用户应将函数调用发送到代理。代理的回退函数将函数调用重定向到实现合约以执行。

### 实现合约
实现合约，也称为逻辑合约，接收来自代理合约的重定向函数调用。实现合约应遵循[可扩展性模式](./Extensibility.md#可扩展性问题)，并直接从[代理库](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/upgrades/library.cairo)导入。

实现合约应：

* 导入代理命名空间。

* 提供一个外部初始化函数（调用Proxy.initializer）以在部署后立即初始化代理。

如果实现是可升级的，则应：

* 包括升级实现的方法（例如升级）。

* 使用访问控制保护合约的可升级性。

实现合约不应：

* 像常规合约一样部署。相反，应声明实现合约（这将创建一个包含其哈希和abi的DeclaredClass）。

* 使用传统构造函数设置其初始状态（用@constructor修饰）。而是使用调用代理构造函数的初始化方法。

> NOTE
代理构造函数包括一个检查，确保初始化方法只能被调用一次；但是_set_implementation不包括此检查。开发人员需要使用访问控制（如[assert_only_admin](#assert_only_admin)）保护其实现合约的可升级性。

有关完整的实现合约示例，请参见：

[可代理实现](https://github.com/OpenZeppelin/cairo-contracts/blob/main/tests/mocks/ProxiableImplementation.cairo)

## 升级库API

### 方法
```
func initializer(proxy_admin: felt):
end

func assert_only_admin():
end

func get_implementation_hash() -> (implementation: felt):
end

func get_admin() -> (admin: felt):
end

func _set_admin(new_admin: felt):
end

func _set_implementation_hash(new_implementation: felt):
end
```

#### 初始化器
使用初始实现初始化代理合约。

参数：
```
proxy_admin: felt
```

返回：无。

#### assert_only_admin
如果被非管理员账户调用，则回滚。

参数：无。

返回：无。

#### get_implementation
返回当前实现的哈希值。

参数：无。

返回：哈希值。
```
implementation: felt
```

#### get_admin
获取当前管理员。

参数：无。

返回值：
```
admin: felt
```

#### _set_admin
将new_admin设置为代理合约的管理员。

参数：
```
new_admin: felt
```
返回：无。

#### _set_implementation_hash
将new_implementation设置为实现的合约类。此方法包含在代理合约的构造函数中，可用于升级合约。

参数：
```
new_implementation: felt
```
返回：无。

### Events
```
func Upgraded(implementation: felt):
end

func AdminChanged(previousAdmin: felt, newAdmin: felt):
end
```

#### 升级
当代理合约设置新的实现类哈希时发出。

参数：
```
implementation: felt
```

#### AdminChanged
当管理员从previousAdmin更改为newAdmin时触发。

参数: 
```
previousAdmin: felt
newAdmin: felt
```

## 使用代理

### 合约升级
要升级合约，实现合约应该包含一个升级方法，当调用时，将引用更改为一个新部署的合约，如下所示。
```
    # declare first implementation
    IMPLEMENTATION = await starknet.declare(
        "path/to/implementation.cairo",
    )

    # deploy proxy
    PROXY = await starknet.deploy(
        "path/to/proxy.cairo",
        constructor_calldata=[
            IMPLEMENTATION.class_hash,  # set implementation hash
        ]
    )

    # declare implementation v2
    IMPLEMENTATION_V2 = await starknet.declare(
        "path/to/implementation_v2.cairo",
    )

    # call upgrade with the new implementation contract class hash
    await signer.send_transaction(
        account, PROXY.contract_address, 'upgrade', [
            IMPLEMENTATION_V2.class_hash
        ]
    )
```
完整的部署和升级实施请参见：

* [升级 V1](https://github.com/OpenZeppelin/cairo-contracts/blob/main/tests/mocks/UpgradesMockV1.cairo)


* [升级 V2](https://github.com/OpenZeppelin/cairo-contracts/blob/main/tests/mocks/UpgradesMockV2.cairo)

### 声明合约
StarkNet 合约有两种形式：合约类和合约实例。合约类表示未实例化的无状态代码；而合约实例被实例化并包含状态。由于代理合约通过其类哈希引用实现合约，因此仅声明一个实现合约即足够（而不是进行完整的部署）。有关声明类的更多信息，请参阅 [StarkNet 的文档](https://starknet.io/docs/hello_starknet/intro.html#declare-contract)。

### 测试方法调用
与大多数 StarkNet 合约一样，与代理合约交互需要一个[账户抽象](./Accounts.md#快速入门)。然而，由于 StarkNet 测试框架的限制，@view 方法也需要一个账户抽象。这仅在测试时是必需的。例如，使用 Python 编写的 getter 方法的差异如下：

```
# standard ERC20 call
result = await erc20.totalSupply().call()

# upgradeable ERC20 call
result = await signer.send_transaction(
        account, PROXY.contract_address, 'totalSupply', []
    )
```

## 预设
预设是从我们的合约库中延伸出来的预先编写的合约。它们可以直接部署，也可以用作自定义模板。

一些预设包括：

* [ERC20可升级版](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20Upgradeable.cairo)

* 更多即将推出！有想法吗？请[提出问题](https://github.com/OpenZeppelin/cairo-contracts/issues/new/choose)！