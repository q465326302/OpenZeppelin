# Creating ERC20 Supply
在本指南中，您将学习如何创建具有自定义供应机制的ERC20令牌。我们将展示两种使用OpenZeppelin Contracts的惯用方法，您将能够将其应用于智能合约开发实践中。

建立在以太坊上的代币实现的标准接口称为ERC20，Contracts包括其广泛使用的实现：名为ERC20合约的合约。实际上，如果您尝试按原样部署*ERC20*的实例，它将变得完全无用...它将没有供应！没有供应的令牌有什么用？

供应的创建方式未在ERC20文档中定义。每个令牌都可以尝试使用自己的机制，从最分散的到最集中的，从最天真的到最研究的等等。

## 固定供应
假设我们想要一个固定供应量为1000的令牌，最初分配给部署合约的帐户。如果您使用的是Contracts v1，则可能编写了以下代码：
```
contract ERC20FixedSupply is ERC20 {
    constructor() {
        totalSupply += 1000;
        balances[msg.sender] += 1000;
    }
}
```

从Contracts v2开始，这种模式不仅被反对，而且被禁止。变量totalSupply和balances现在是ERC20的私有实现细节，您无法直接写入它们。相反，有一个内部的*_mint*函数会执行这个操作：
```
contract ERC20FixedSupply is ERC20 {
    constructor() ERC20("Fixed", "FIX") {
        _mint(msg.sender, 1000);
    }
}
```
像这样封装状态可以使扩展合约更安全。例如，在第一个示例中，我们必须手动将totalSupply与修改后的balances同步，这很容易忘记。实际上，我们忽略了另一个同样容易被忘记的东西：标准所要求的Transfer事件，一些客户端依赖于此。第二个示例没有这个错误，因为内部的_mint函数会处理它。

## 奖励矿工
内部的*_mint*函数是允许我们编写实现供应机制的ERC20扩展的关键构建块。

我们将实现的机制是给生产以太坊区块的矿工一个代币奖励。在Solidity中，我们可以通过全局变量block.coinbase访问当前区块矿工的地址。每当有人调用我们的代币上的函数mintMinerReward()时，我们将向此地址铸造代币奖励。这个机制听起来可能有些愚蠢，但你永远不知道这可能会产生什么样的动态，值得进行分析和实验！
```contract ERC20WithMinerReward is ERC20 {
    constructor() ERC20("Reward", "RWD") {}

    function mintMinerReward() public {
        _mint(block.coinbase, 1000);
    }
}
```

正如我们所看到的，_mint使得正确执行这个过程变得非常简单。

## 模块化机制
在合约中已经包含了一个供应机制：ERC20PresetMinterPauser。这是一个通用机制，其中一组帐户被分配了铸造者角色，授予他们调用铸造函数的权限，这是一个外部版本的_mint。

这可以用于中心化铸造，其中外部拥有的帐户（即具有一对加密密钥的人）决定创建多少供应量以及为谁创建。这种机制有非常合法的用例，例如[传统的资产支持的稳定币](https://medium.com/reserve-currency/why-another-stablecoin-866f774afede#3aea)。

具有铸造者角色的帐户不需要是外部拥有的，它们也可以是实现无信任机制的智能合约。实际上，我们可以实现与上一节相同的行为。
```
contract MinerRewardMinter {
    ERC20PresetMinterPauser _token;

    constructor(ERC20PresetMinterPauser token) {
        _token = token;
    }

    function mintMinerReward() public {
        _token.mint(block.coinbase, 1000);
    }
}
```
当使用ERC20PresetMinterPauser实例初始化此合约并授予该合约的铸造者角色时，将得到与上一节实现完全相同的行为。使用ERC20PresetMinterPauser有趣的是，我们可以通过将角色分配给多个合约轻松结合多个供应机制，而且我们可以动态地实现这一点。

> TIP
要了解有关角色和权限系统的更多信息，请转到我们的*访问控制指南*。

## 自动化奖励

到目前为止，我们的供应机制是手动触发的，但ERC20还允许我们通过*_beforeTokenTransfer* hooks （请参见*使用 hooks *）扩展令牌的核心功能。

在上一节的供应机制基础上，我们可以使用此 hooks 为包含在区块链中的每个令牌转移铸造矿工奖励。
```
contract ERC20WithAutoMinerReward is ERC20 {
    constructor() ERC20("Reward", "RWD") {}

    function _mintMinerReward() internal {
        _mint(block.coinbase, 1000);
    }

    function _beforeTokenTransfer(address from, address to, uint256 value) internal virtual override {
        if (!(from == address(0) && to == block.coinbase)) {
          _mintMinerReward();
        }
        super._beforeTokenTransfer(from, to, value);
    }
}
```

## 总结
我们已经看到了两种实现ERC20供应机制的方法：通过_mint内部实现和通过ERC20PresetMinterPauser外部实现。希望这有助于您了解如何使用OpenZeppelin Contracts以及其背后的设计原理，并可以将其应用于您自己的智能合约中。