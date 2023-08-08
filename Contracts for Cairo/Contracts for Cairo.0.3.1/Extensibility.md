# Extensibility
> NOTE
预计这种模式将会发展（正如它已经做到的那样），或者在Cairo中[实现适当的可扩展性功能](https://community.starknet.io/t/contract-extensibility-pattern/210/11?u=martriay)后可能会消失。

## 目录
* 可扩展性问题

* 该模式™️

    * 库

    * 合同

* 预设

* 函数名称和编码风格

* 模拟挂钩

## 可扩展性问题
智能合约开发是一项关键任务。与所有软件开发一样，它容易出错；但与大多数情况不同的是，一个错误可能导致组织和个人遭受重大损失。因此，编写复杂的智能合约是一项非常谨慎的任务。

减少引入错误的最佳方法之一是重用现有的、经过实战测试的代码，即使用库。但在StarkNet的智能合约中重用代码并不容易：

* Cairo没有明确的智能合约扩展机制，如继承或组合。

* 使用导入进行模块化可能会导致冲突（尤其是因为参数不是选择器的一部分），而缺乏重写或别名解决冲突的方法。

* 由导入模块定义的任何@external函数将自动由导入者（即智能合约）重新公开。

为了解决这些问题，该项目基于以下准则™️进行构建。

## 该模式
这个想法是有两种类型的Cairo模块：库和合约。库定义可重用的逻辑和存储变量，这些变量可以由合约进行扩展和公开。合约可以部署，库不能。

为了最小化风险、样板代码，并避免函数命名冲突，我们遵循以下规则：

### 库
考虑以下类型的函数：

* private：库内部私有函数，不应在模块外部或导入时使用。

* public：库的公共API的一部分。

* internal：公共的子集，可能不鼓励使用或潜在不安全（例如ERC20的_transfer）。

* external：公共的子集，准备好由合约直接导出（例如ERC20的transfer）。

* storage：存储变量函数。

然后：

* 必须在命名空间下实现public和external函数。

* 必须在命名空间外部实现private函数，以避免公开它们。

* 必须在内部函数前加下划线作为前缀（例如ERC20._mint）。

* 不能在外部函数前加下划线作为前缀（例如ERC20.transfer）。

* 必须在存储函数前加上命名空间的名称，以防止与其他库发生冲突（例如ERC20balances）。

* 不能实现任何@external、@view或@constructor函数。

* 可以实现初始化函数（永远不要使用@constructor或@external）。

* 不能在任何函数中调用初始化函数。

### 合同
* 可以从库中导入。

* 如果需要，应实现@external函数。

* 应实现调用初始化函数的@constructor函数。

* 除构造函数外，不能在任何函数中调用初始化函数。

请注意，由于初始化函数永远不会标记为@external，并且它们不会从任何地方调用，除了构造函数之外，因此在部署后不会重新初始化的风险。库开发人员需要避免使初始化函数相互依赖，以避免可能导致库的双重构造的奇怪依赖路径。

### 预设
预设是从我们的合同库中扩展的预先编写的合同。它们可以按原样部署，也可以用作自定义模板。

一些预设包括：

* [账户](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/account/presets/Account.cairo)

* [ERC165](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/tests/mocks/ERC165.cairo)

* [ERC20可铸造](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20Mintable.cairo)

* [ERC20可暂停](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20Pausable.cairo)

* [ERC20可升级](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20Upgradeable.cairo)

* [ERC20](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc20/presets/ERC20.cairo)

* [ERC721可铸造可销毁](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc721/presets/ERC721MintableBurnable.cairo)

* [ERC721可铸造可暂停](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc721/presets/ERC721MintablePausable.cairoo)

* [ERC721可枚举可销毁](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/token/erc721/enumerable/presets/ERC721EnumerableMintableBurnable.cairo)

## 函数名称和编码风格
* 根据Cairo的编程风格，我们在库API中使用snake_case（例如ERC20.transfer_from，ERC721.safe_transfer_from）。

* 但为了与标准的EVM生态系统兼容，我们使用camelCase在合约中实现外部函数（例如ERC20合约中的transferFrom）。

* 所谓的“only owner”之类的保护函数以assert_为前缀（例如Ownable.assert_only_owner）。

## 模拟钩子
与Solidity版本的[OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts)不同，该库不实现*钩子*。主要原因是Cairo不支持重写函数。

以下是Solidity中钩子的示例。
```
abstract contract ERC20Pausable is ERC20, Pausable {
    function _beforeTokenTransfer(address from, address to, uint256 amount) internal virtual override {
        super._beforeTokenTransfer(from, to, amount);

        require(!paused(), "ERC20Pausable: token transfer while paused");
    }
}
```

相反，可扩展性模式允许我们通过在调用函数（即transfer）之前或之后添加代码行来简单地扩展库的实现。这样，我们可以轻松地完成任务。
```
@external
func transfer{
        syscall_ptr : felt*,
        pedersen_ptr : HashBuiltin*,
        range_check_ptr
    }(recipient: felt, amount: Uint256) -> (success: felt):
    Pausable.assert_not_paused()
    ERC20.transfer(recipient, amount)
    return (TRUE)
end
```