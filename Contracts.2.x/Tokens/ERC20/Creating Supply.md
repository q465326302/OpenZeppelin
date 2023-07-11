# Creating ERC20 Supply
在本指南中，您将学习如何使用自定义供应机制创建一个ERC20代币。我们将展示两种使用OpenZeppelin Contracts的典型方式，您可以将其应用于您的智能合约开发实践中。

以太坊上构建的代币所实现的标准接口称为ERC20，Contracts包括了一个广泛使用的实现：名为ERC20的合约。这个合约与标准本身一样非常简单。实际上，如果您尝试原样部署*ERC20*的实例，它将是完全无用的...因为它没有供应！没有供应的代币有什么用呢？

供应的创建方式在ERC20文档中没有定义。每个代币都可以自由尝试使用自己的机制，从最分散到最集中，从最天真到最深入研究等等。

## 固定供应量
假设我们想要一个固定供应量为1000的代币，最初分配给部署合约的账户。如果您使用的是Contracts v1，您可能已经编写了如下代码：
```
contract ERC20FixedSupply is ERC20 {
    constructor() public {
        totalSupply += 1000;
        balances[msg.sender] += 1000;
    }
}
```
从合约v2开始，这种模式不仅被不鼓励，而且被禁止。totalSupply和balances变量现在是ERC20的私有实现细节，您不能直接写入它们。相反，有一个内部的*_mint*函数将会完成这个操作：

```
contract ERC20FixedSupply is ERC20 {
    constructor() public {
        _mint(msg.sender, 1000);
    }
}
```
像这样封装状态可以使扩展合约更加安全。例如，在第一个示例中，我们必须手动将totalSupply与修改后的余额保持同步，这很容易忘记。事实上，我们还省略了另一件容易忘记的事情：标准所要求的Transfer事件，一些客户端依赖于该事件。第二个示例没有这个错误，因为内部的_mint函数会处理这个问题。

## 奖励矿工
内部的*_mint*函数是允许我们编写实现供应机制的ERC20扩展的关键构建块。

我们将实现的机制是为生产以太坊区块的矿工提供代币奖励。在Solidity中，我们可以通过全局变量block.coinbase访问当前区块的矿工地址。每当有人在我们的代币上调用mintMinerReward()函数时，我们将向这个地址铸造一个代币奖励。这个机制可能听起来很愚蠢，但你永远不知道这可能导致什么样的动态，值得进行分析和实验！

```
contract ERC20WithMinerReward is ERC20 {
    function mintMinerReward() public {
        _mint(block.coinbase, 1000);
    }
}
```
正如我们所看到的，_mint使得正确实现这一点变得非常容易。

## 模块化机制
在合同中已经包含了一个供应机制：*ERC20Mintable*。这是一个通用的机制，其中一组账户被分配了*铸造者*角色，使他们有权限调用铸造函数，即_external的版本。

这可以用于集中铸币，其中一个外部拥有的账户（即拥有一对密码密钥的人）决定创建多少供应量以及分配给谁。这种机制有非常合理的用例，比如[传统资产支持的稳定币](https://medium.com/reserve-currency/why-another-stablecoin-866f774afede#3aea)。

拥有铸造者角色的账户不需要是外部拥有的，它们也可以是实现了无信任机制的智能合约。实际上，我们可以实现与前一节相同的行为。

```
contract MinerRewardMinter {
    ERC20Mintable _token;

    constructor(ERC20Mintable token) public {
        _token = token;
    }

    function mintMinerReward() public {
        _token.mint(block.coinbase, 1000);
    }
}
```

当使用ERC20Mintable实例初始化此合同时，将导致与前一节实现的完全相同的行为。使用ERC20Mintable的有趣之处在于，我们可以通过将该角色分配给多个合同来轻松组合多个供应机制，并且还可以动态地实现这一点。

## 自动化奖励
除了_mint之外，ERC20还提供了其他可以使用或扩展的内部函数，比如*_transfer*。该函数实现了代币转移，并且被ERC20使用，因此可以用于自动触发功能。这是使用ERC20Mintable方法无法实现的。

在我们之前的供应机制上添加，我们可以使用这个功能为包含在区块链中的每个代币转移铸造一个矿工奖励。

```
contract ERC20WithAutoMinerReward is ERC20 {
    function _mintMinerReward() internal {
        _mint(block.coinbase, 1000);
    }

    function _transfer(address from, address to, uint256 value) internal {
        _mintMinerReward();
        super._transfer(from, to, value);
    }
}
```
请注意，我们首先通过调用super._transfer来重写_transfer函数，以便先铸造矿工奖励，然后再运行原始的实现。这最后一步非常重要，以保留ERC20转账的原始语义。

##总结
我们已经看到了两种实现ERC20供应机制的方法：通过_mint内部实现和通过ERC20Mintable外部实现。希望这有助于您了解如何使用OpenZeppelin以及其背后的一些设计原则，并且您可以将它们应用到您自己的智能合约中。