# ERC20
ERC20代币合约用于*跟踪可互换*的代币：任何一个代币与其他代币完全相等；没有任何代币具有特殊的权利或行为。这使得ERC20代币非常适用于作为**交换媒介货币、投票权、质押**等用途。

OpenZeppelin Contracts提供了许多与ERC20相关的合约。在*API参考*中，您可以找到关于它们属性和使用方法的详细信息。

## 构建ERC20代币合约
使用Contracts，我们可以轻松创建自己的ERC20代币合约，用于跟踪一个虚拟项目中的内部货币Gold（GLD）。

以下是我们的GLD代币可能的样子。
```
// contracts/GLDToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract GLDToken is ERC20 {
    constructor(uint256 initialSupply) public ERC20("Gold", "GLD") {
        _mint(msg.sender, initialSupply);
    }
}
```
我们的合约经常通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)使用，并且在这里我们同时重用了*ERC20*的基本标准实现和可选的*名称*、*符号*和*小数位扩展*。此外，我们正在创建一个初始供应的代币，这些代币将被分配给部署合约的地址。

> TIP
要了解更完整的ERC20供应机制，请参阅*创建ERC20供*应。

就是这样！一旦部署，我们将能够查询部署者的余额。
```
> GLDToken.balanceOf(deployerAddress)
1000000000000000000000
```
我们还可以将这些代币转移到其他账户。

```
> GLDToken.transfer(otherAddress, 300000000000000000000)
> GLDToken.balanceOf(otherAddress)
300000000000000000000
> GLDToken.balanceOf(deployerAddress)
700000000000000000000
```

## 关于小数的说明

通常，您希望能够将您的代币分成任意数量：例如，如果您拥有5个GLD代币，您可能想要给朋友发送1.5个GLD代币，并保留3.5个GLD代币给自己。不幸的是，Solidity和EVM不支持这种行为：只能使用整数（完整的）数字，这带来了问题。您可以发送1或2个代币，但不能发送1.5个。

为了解决这个问题，*ERC20*提供了一个*小数字段*，用于指定一个代币拥有多少小数位。为了能够转移1.5个GLD代币，小数位必须至少为1，因为这个数字有一个小数位。

如何实现这一点？实际上非常简单：代币合约可以使用更大的整数值，这样50的余额将代表5个GLD代币，15的转移将对应于发送1.5个GLD代币，依此类推。

重要的是要理解，小数位仅用于显示目的。合约内的所有算术仍然是在整数上执行的，不同的用户界面（钱包、交易所等）必须根据小数位调整显示的值。总代币供应量和每个账户的余额不是以GLD指定的：您需要除以10^小数位数才能得到实际的GLD金额。

除非您有非常特殊的原因，否则您可能希望使用18个小数位的值，就像以太坊和大多数使用的ERC20代币合约一样。当铸造代币或进行转移时，您实际上将发送num GLD * 10^小数位数的数字。

> NOTE
默认情况下，ERC20使用18个小数位的值。要使用不同的值，您需要在构造函数中调用*_setupDecimals*方法。

因此，如果您想使用具有18个小数位的代币合约发送5个代币，则实际上要调用的方法将是：
```
transfer(recipient, 5 * 10^18);
```

## 预设ERC20合约
有一个预设的ERC20合约可用，名为*ERC20PresetMinterPauser*。它预设允许代币铸造（创建）、停止所有代币转移（暂停）以及允许持有人销毁（销毁）他们的代币。该合约使用*访问控制*来控制对铸造和暂停功能的访问。部署该合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色。

该合约可以直接部署，无需编写任何Solidity代码。它可用于快速原型设计和测试，同时也适用于生产环境。