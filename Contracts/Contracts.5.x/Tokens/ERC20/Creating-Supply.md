# Creating ERC20 Supply
在本指南中，您将学习如何创建一个具有自定义供应机制的ERC20代币。我们将展示两种使用OpenZeppelin合约的典型方式，您将能够将其应用于您的智能合约开发实践。

在以太坊上构建的代币实现的标准接口被称为*ERC20*，合约包括了一个广泛使用的实现：名为ERC20的合约。这个合约，就像标准本身一样，非常简单和基础。事实上，如果你试图部署一个原样的ERC20实例，它将完全无用……它没有供应！一个没有供应的代币有什么用呢？

供应的创建方式在ERC20文档中并未定义。每个代币都可以自由地尝试其自己的机制，从最去中心化到最中心化，从最天真到最研究，等等。

## Fixed Supply
假设我们想要一个固定供应量为1000的代币，最初分配给部署合约的账户。如果你使用过合约v1，你可能写过如下的代码:
```
contract ERC20FixedSupply is ERC20 {
    constructor() {
        totalSupply += 1000;
        balances[msg.sender] += 1000;
    }
}
```

从Contracts v2开始，这种模式不仅被不推荐，而且被禁止。现在，totalSupply和balances变量是ERC20的私有实现细节，你不能直接写入它们。相反，有一个内部的*_mint*函数会做完全相同的事情:
```
contract ERC20FixedSupply is ERC20 {
    constructor() ERC20("Fixed", "FIX") {
        _mint(msg.sender, 1000);
    }
}
```

将状态封装起来使得扩展合约更安全。例如，在第一个例子中，我们必须手动保持totalSupply与修改后的余额同步，这很容易忘记。实际上，我们还遗漏了一些也很容易被忘记的东西：标准所要求的，一些客户端依赖的Transfer事件。第二个例子没有这个bug，因为内部的_mint函数会处理它。

## 奖励矿工
内部的*_mint*函数是我们编写ERC20扩展实现供应机制的关键构建块。

我们将实现的机制是对产生以太坊区块的矿工的代币奖励。在Solidity中，我们可以在全局变量block.coinbase中访问当前区块的矿工的地址。每当有人在我们的代币上调用函数mintMinerReward()时，我们都会向这个地址铸造一个代币奖励。这个机制可能听起来很傻，但你永远不知道这可能会产生什么样的动态，这是值得分析和实验的！
```
contract ERC20WithMinerReward is ERC20 {
    constructor() ERC20("Reward", "RWD") {}

    function mintMinerReward() public {
        _mint(block.coinbase, 1000);
    }
}
```

正如我们所看到的，_mint使得正确执行此操作变得非常容易。

## 自动化奖励
到目前为止，我们的供应机制是手动触发的，但ERC20也允许我们通过*_update*函数扩展代币的核心功能。

在前一部分的供应机制中，我们可以使用这个函数为每一个被包含在区块链中的代币转移铸造一个矿工奖励。
```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract ERC20WithAutoMinerReward is ERC20 {
    constructor() ERC20("Reward", "RWD") {
        _mintMinerReward();
    }

    function _mintMinerReward() internal {
        _mint(block.coinbase, 1000);
    }

    function _update(address from, address to, uint256 value) internal virtual override {
        if (!(from == address(0) && to == block.coinbase)) {
            _mintMinerReward();
        }
        super._update(from, to, value);
    }
}
```

## 总结
我们已经看到如何通过_mint实现ERC20供应机制。希望这能帮助你理解如何使用OpenZeppelin合约以及其背后的一些设计原则，你可以将这些应用到你自己的智能合约中。