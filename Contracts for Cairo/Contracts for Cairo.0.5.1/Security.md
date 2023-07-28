# Security
以下文档提供了openzeppelin/security/中的方法和常量的上下文、原因和示例。

> CAUTION
预计该模块会不断发展。

# 目录
* Initializable

* Pausable

* Reentrancy Guard

* SafeMath

  * SafeUint256

## Initializable
Initializable库提供了一个简单的机制，模拟构造函数的功能。更具体地说，它使得逻辑可以只执行一次，这在无法使用构造函数设置合约的初始状态时非常有用。

使用Initializable的推荐模式是在目标函数中包含一个检查Initializable状态为False的检查，并像这样调用initialize函数。
```
from openzeppelin.security.initializable.library import Initializable

@external
func foo{syscall_ptr : felt*, pedersen_ptr : HashBuiltin*, range_check_ptr}() {
    let (initialized) = Initializable.initialized();
    assert initialized = FALSE;

    Initializable.initialize();
    return ();
}
```
> CAUTION
这个Initializable模式只应该用在一个函数上。


## Pausable
可暂停库允许合约实现紧急停止机制。这对于防止交易直到评估期结束或在出现严重错误时冻结所有交易的情况非常有用。

要使用可暂停库，合约应包含暂停和取消暂停函数（应受保护）。对于只在未暂停时可用的方法，插入assert_not_paused。对于只在暂停时可用的方法，插入assert_paused。例如：
```
from openzeppelin.security.pausable.library import Pausable

@external
func whenNotPaused{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    Pausable.assert_not_paused();

    // function body

    return ();
}

@external
func whenPaused{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    Pausable.assert_paused();

    // function body

    return ();
}
```
> NOTE
暂停和取消暂停已经包含了这些断言。换句话说，当已经暂停时，无法调用暂停操作，反之亦然。

以下是使用Pausable库的完整实现列表：

* [ERC20Pausable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/src/openzeppelin/token/erc20/presets/ERC20Pausable.cairo)
* [ERC721MintablePausable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/src/openzeppelin/token/erc721/presets/ERC721MintablePausable.cairo)

## 重入保护
[重入攻击](https://gus-tavo-guim.medium.com/reentrancy-attack-on-smart-contracts-how-to-identify-the-exploitable-and-an-example-of-an-attack-4470a2d8dfe4)是指调用者通过递归调用目标函数，能够获得比允许的更多资源的情况。

由于Cairo不支持像Solidity那样的修饰符，[重入保护库](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/src/openzeppelin/security/reentrancyguard/library.cairo)提供了两个方法start和end来保护函数免受重入攻击。受保护的函数必须在第一个函数语句之前调用ReentrancyGuard.start，并在return语句之前调用ReentrancyGuard.end，如下所示：
```
from openzeppelin.security.reentrancyguard.library import ReentrancyGuard

func test_function{syscall_ptr : felt*, pedersen_ptr : HashBuiltin*, range_check_ptr}() {
   ReentrancyGuard.start();

   // function body

   ReentrancyGuard.end();
   return ();
}
```

## SafeMath

### SafeUint256
SafeUint256命名空间在[SafeMath库](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/src/openzeppelin/security/safemath/library.cairo)中通过利用Cairo的Uint256库并集成溢出检查，为无符号256位整数（uint256）提供算术运算。Cairo的Uint256函数中有一些在溢出时不会回滚。例如，当总和超过256位时，uint256_add将返回一个位进位。该库包括了额外的断言，确保值不会溢出。

使用SafeUint256方法非常简单。只需导入SafeUint256并插入算术方法，如下所示。
```
from openzeppelin.security.safemath.library import SafeUint256

func add_two_uints{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    a: Uint256, b: Uint256) -> (c: Uint256) {
    let (c: Uint256) = SafeUint256.add(a, b);
    return (c);
}
```