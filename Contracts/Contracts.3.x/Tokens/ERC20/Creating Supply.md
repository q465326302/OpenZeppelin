# Creating ERC20 Supply
在本指南中，您将学习如何创建具有自定义供应机制的ERC20代币。我们将展示两种使用OpenZeppelin Contracts实现此目的的典型方式，您可以将其应用于智能合约开发实践中。

以太坊上构建的代币实现的标准接口称为ERC20，Contracts包括一个广泛使用的实现：名为*ERC20*的合约。这个合约，就像标准本身一样，非常简单。事实上，如果您尝试部署一个原始的ERC20实例，它将是完全无用的...它没有供应量！没有供应量的代币有什么用呢？

供应量的创建方式在ERC20文件中没有定义。每个代币都可以自由地尝试使用自己的机制，从最分散到最集中，从最天真到最研究过的等等。

## 固定供应量
假设我们想要一个供应量为1000的代币，最初分配给部署合约的账户。如果您使用过Contracts v1，您可能已经编写了类似以下代码的代码：
```
contract ERC20FixedSupply is ERC20 {
    constructor() public {
        totalSupply += 1000;
        balances[msg.sender] += 1000;
    }
}
```

从合同v2开始，这种模式不仅被反对，而且被禁止。totalSupply和balances变量现在是ERC20的私有实现细节，您不能直接写入它们。相反，有一个内部的_mint函数将完成这个任务。
```
contract ERC20FixedSupply is ERC20 {
    constructor() public ERC20("Fixed", "FIX") {
        _mint(msg.sender, 1000);
    }
}
```
像这样封装状态使得扩展合约更加安全。例如，在第一个示例中，我们必须手动保持totalSupply与修改后的balances同步，这很容易忘记。实际上，我们还省略了另一件容易忘记的事情：标准所需的Transfer事件，某些客户端依赖此事件。第二个示例没有这个错误，因为内部的_mint函数会处理这个问题。

## 奖励矿工
内部的*_mint*函数是我们可以使用的关键构建块，可以实现一个供应机制的ERC20扩展。

我们将实现的机制是为生产以太坊区块的矿工提供令牌奖励。在Solidity中，我们可以通过全局变量block.coinbase访问当前区块的矿工地址。每当有人在我们的令牌上调用mintMinerReward()函数时，我们将向该地址铸造一个令牌奖励。这种机制可能听起来很愚蠢，但你永远不知道这可能导致什么样的动态，值得分析和实验！

```
contract ERC20WithMinerReward is ERC20 {
    constructor() public ERC20("Reward", "RWD") {}

    function mintMinerReward() public {
        _mint(block.coinbase, 1000);
    }
}
```
正如我们所看到的，_mint使得正确执行这一过程非常简单。

## 模块化机制
在Contracts中已经包含了一个供应机制：ERC20PresetMinterPauser。这是一个通用的机制，其中一组账户被分配了铸币者角色，使它们有权限调用一个铸币函数，即 _mint 的外部版本。

这可以用于集中铸币，其中外部拥有的账户（即拥有一对密码密钥的人）决定创建多少供应量以及为谁创建。这个机制有非常合理的使用场景，比如[传统资产支持的稳定币](https://medium.com/reserve-currency/why-another-stablecoin-866f774afede#3aea)。

具有铸币者角色的账户不需要是外部拥有的，同样也可以是实现了无信任机制的智能合约。实际上，我们可以实现与上一节相同的行为。

```
contract MinerRewardMinter {
    ERC20PresetMinterPauser _token;

    constructor(ERC20PresetMinterPauser token) public {
        _token = token;
    }

    function mintMinerReward() public {
        _token.mint(block.coinbase, 1000);
    }
}
```

当使用ERC20PresetMinterPauser实例进行初始化并授予该合约的铸造者角色时，将得到与前一节实现的完全相同的行为。使用ERC20PresetMinterPauser的有趣之处在于我们可以通过将该角色分配给多个合约来轻松组合多个供应机制，而且我们还可以动态地进行这样的操作。

> TIP
要了解有关角色和权限系统的更多信息，请参阅我们的*访问控制指南*。

## 自动化奖励
到目前为止，我们的供应机制是手动触发的，但是ERC20还允许我们通过*_beforeTokenTransfer*钩子（参见*使用钩子*）扩展代币的核心功能。

在前几节中添加到供应机制中，我们可以使用此钩子来为包含在区块链中的每次代币转移铸造一个矿工奖励。
```
contract ERC20WithAutoMinerReward is ERC20 {
    constructor() public ERC20("Reward", "RWD") {}

    function _mintMinerReward() internal {
        _mint(block.coinbase, 1000);
    }

    function _beforeTokenTransfer(address from, address to, uint256 value) internal virtual override {
        _mintMinerReward();
        super._beforeTokenTransfer(from, to, value);
    }
}
```

## 总结
我们已经看到了两种实现ERC20供应机制的方式：通过_mint内部实现和通过ERC20PresetMinterPauser外部实现。希望这能帮助您了解如何使用OpenZeppelin以及其中的一些设计原则，并将其应用到您自己的智能合约中。