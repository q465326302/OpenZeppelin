# ERC20
一个ERC20代币合约用于*跟踪可互换的代币*：任何一个代币与其他代币完全相等；没有任何代币具有特殊的权利或行为。这使得ERC20代币在**作为交换货币、投票权、质押等**方面非常有用。

OpenZeppelin Contracts提供了许多与ERC20相关的合约。在*API参考*中，您将找到关于它们属性和用法的详细信息。

## 构建一个ERC20代币合约
使用Contracts，我们可以轻松地创建自己的ERC20代币合约，用于跟踪Gold (GLD)，这是一个虚构游戏中的内部货币。

下面是我们的GLD代币可能的样子。
```
pragma solidity ^0.5.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20Detailed.sol";

contract GLDToken is ERC20, ERC20Detailed {
    constructor(uint256 initialSupply) ERC20Detailed("Gold", "GLD", 18) public {
        _mint(msg.sender, initialSupply);
    }
}
```
我们的合约经常通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)来使用，这里我们重用了*ERC20*来进行基本标准实现，并使用*ERC20Detailed*来获取*名称*、*符号*和*小数位*属性。此外，我们创建了一个初始的代币供应量，将分配给部署合约的地址。

> TIP
有关 ERC20 供应机制的更完整讨论，请参阅*创建 ERC20 供应*。

就是这样！一旦部署完成，我们就能查询部署者的余额。
```
> GLDToken.balanceOf(deployerAddress)
1000
```

我们还可以将这些代币*转移*到其他账户。
```
> GLDToken.transfer(otherAddress, 300)
> GLDToken.balanceOf(otherAddress)
300
> GLDToken.balanceOf(deployerAddress)
700
```

## 关于小数的说明
通常，您可能希望将您的代币分成任意数量：例如，如果您拥有5个GLD，您可能希望向朋友发送1.5个GLD，并保留3.5个GLD给自己。不幸的是，Solidity和EVM不支持这种行为：只能使用整数（整数）数字，这是一个问题。您可以发送1或2个代币，但不能发送1.5个。

为了解决这个问题，*ERC20Detailed*提供了一个*小数字段*，用于指定代币的小数位数。要能够传输1.5个GLD，小数位数必须至少为1，因为该数字有一个小数位。

如何实现这一点？实际上非常简单：代币合约可以使用较大的整数值，以便50的余额表示5个GLD，15的转账表示发送了1.5个GLD，等等。

重要的是要理解小数只用于显示目的。合约内部的所有算术仍然是基于整数进行的，而各种用户界面（钱包、交易所等）必须根据小数调整显示的值。总代币供应量和每个账户的余额没有以GLD为单位指定：您需要除以10^小数位数才能获得实际的GLD金额。

您可能希望使用18个小数位的小数值，就像以太坊和大多数使用的ERC20代币合约一样，除非您有非常特殊的原因。当铸造代币或在代币之间进行转移时，您实际上将发送数字num GLD * 10^小数。

因此，如果您想使用具有18个小数位的代币合约发送5个代币，那么要调用的方法实际上将是：
```
transfer(recipient, 5 * 10^18);
```