# ERC20
ERC20代币合约跟踪*可替代的代币*：任何一个代币都与其他代币完全相等；没有代币具有与之相关的特殊权利或行为。这使得**ERC20代币对于货币交换媒介、投票权、质押等**非常有用。

OpenZeppelin合约提供了许多与ERC20相关的合约。在*API参考*中，你将找到关于它们属性和使用的详细信息。

## 构建一个ERC20代币合约
使用合约，我们可以轻松创建自己的ERC20代币合约，该合约将用于跟踪黄金（GLD），这是一个假想游戏中的内部货币。

以下是我们的GLD代币可能的样子:
```
// contracts/GLDToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract GLDToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("Gold", "GLD") {
        _mint(msg.sender, initialSupply);
    }
}
```

我们的合约通常通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)来使用，这里我们重用*ERC20*来实现基本的标准实现以及*名称*、*符号*和*小数*的可选扩展。此外，我们还创建了一个初始供应的代币，这些代币将分配给部署合约的地址。

> TIP
关于ERC20供应机制的更完整讨论，请参阅*创建ERC20供应*。

就是这样！一旦部署，我们将能够查询部署者的余额:
```
> GLDToken.balanceOf(deployerAddress)
1000000000000000000000
```

我们也可以将这些代币转移到其他账户:
```
> GLDToken.transfer(otherAddress, 300000000000000000000)
> GLDToken.balanceOf(otherAddress)
300000000000000000000
> GLDToken.balanceOf(deployerAddress)
700000000000000000000
```

## 关于小数的说明
通常，你可能希望能够将你的代币分割成任意数量：比如，如果你拥有5个GLD，你可能想要给朋友发送1.5个GLD，并保留3.5个GLD给自己。不幸的是，Solidity和EVM并不支持这种行为：只能使用整数（整数）数字，这就带来了问题。你可以发送1或2个代币，但不能发送1.5个。

为了解决这个问题，*ERC20*提供了一个*小数字段*，用于指定一个代币有多少个小数位。为了能够转移1.5个GLD，小数必须至少为1，因为该数字有一个小数位。

这如何实现呢？实际上非常简单：一个代币合约可以使用更大的整数值，所以一个余额为50的代表5个GLD，一个转账为15的对应1.5个GLD被发送，等等。

重要的是要理解，小数只是用于显示目的。合约内部的所有算术运算仍然是在整数上进行的，而不同的用户界面（钱包，交易所等）必须根据小数调整显示的值。每个账户的总代币供应量和余额并未以GLD指定：你需要除以10 ** 小数来得到实际的GLD数量。

你可能希望使用18作为小数值，就像以太坊和大多数正在使用的ERC20代币合约一样，除非你有特别的理由不这样做。在铸造代币或转移它们时，你实际上将发送的数量是 num GLD * (10 ** 小数)。

> NOTE
默认情况下，ERC20对小数使用18的值。要使用不同的值，你需要在你的合约中重写decimals()函数。
```
function decimals() public view virtual override returns (uint8) {
  return 16;
}
```

所以，如果你想要通过带有18位小数的代币合约发送5个代币，你实际上需要调用的方法将是:
```
transfer(recipient, 5 * (10 ** 18));
```