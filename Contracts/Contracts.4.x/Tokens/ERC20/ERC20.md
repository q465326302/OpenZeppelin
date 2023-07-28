# ERC20
一个ERC20代币合约用于跟踪*可交换的代币*：任何一个代币都完全等同于任何其他代币；没有任何代币具有特殊的权利或行为。这使得ERC20代币适用于**像交换货币、投票权、抵押**等方面。

OpenZeppelin Contracts提供了许多与ERC20相关的合约。在*API参考*中，您将找到有关它们属性和用法的详细信息。

## 构建ERC20代币合约

通过使用合约，我们可以轻松创建自己的ERC20代币合约，用于跟踪Gold（GLD），这是一个虚构游戏中的内部货币。

以下是我们的GLD代币的样子。
```
// contracts/GLDToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract GLDToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("Gold", "GLD") {
        _mint(msg.sender, initialSupply);
    }
}
```

我们的合约通常通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)来使用，这里我们重用*ERC20*作为基本标准实现和*名称*、*符号*和*小数点*可选扩展。此外，我们创建了一个初始供应量的代币，该代币将分配给部署合约的地址。

>TIP
有关ERC20供应机制的更完整讨论，请参见*创建ERC20供应量*。

就是这样！一旦部署，我们将能够查询部署者的余额：
```
> GLDToken.balanceOf(deployerAddress)
1000000000000000000000
```

我们也可以将这些*代币*转移到其他账户。
```
> GLDToken.transfer(otherAddress, 300000000000000000000)
> GLDToken.balanceOf(otherAddress)
300000000000000000000
> GLDToken.balanceOf(deployerAddress)
700000000000000000000
```

## 关于小数点

通常，您可能希望将代币分成任意数量：例如，如果您拥有5个GLD，您可能希望将1.5个GLD发送给朋友，并将3.5个GLD保留给自己。不幸的是，Solidity和EVM不支持此行为：只能使用整数（整数）数字，这会带来问题。您可以发送1或2个代币，但不能发送1.5个代币。

为了解决这个问题，*ERC20*提供了一个*小数位字段*，用于指定代币具有多少小数位。要能够转移1.5个GLD，小数位必须至少为1，因为该数字具有一个小数位。

如何实现呢？这实际上非常简单：代币合约可以使用更大的整数值，以便余额为50表示5个GLD，转移15将对应于发送1.5个GLD，依此类推。

重要的是要理解，小数位仅用于显示目的。合约内的所有算术仍然是在整数上执行的，而不同的用户界面（钱包，交易所等）必须根据小数位调整显示的值。每个帐户的总代币供应量和余额都不是以GLD指定的：您需要除以10 **小数位以获得实际的GLD金额。

您可能希望使用18个小数位的值，就像以太和大多数正在使用的ERC20代币合约一样，除非您有非常特殊的原因。在铸造代币或将其转移时，您实际上将发送数字num GLD *（10 **小数位）。

>NOTE
默认情况下，ERC20使用18个小数位的值。要使用不同的值，您需要在合约中覆盖decimals（）函数。
```
function decimals() public view virtual override returns (uint8) {
  return 16;
}
```

如果您想使用具有18个小数位的代币合约发送5个代币，则需要调用的方法实际上是：
```
transfer(recipient, 5 * (10 ** 18));
```

## 预设的ERC20合约
已经有一个预设的ERC20合约可用，即[ERC20PresetMinterPauser](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.7/contracts/token/ERC20/presets/ERC20PresetMinterPauser.sol)。它预设了允许代币铸造（创建）、停止所有代币转移（暂停）以及允许持有人销毁（销毁）其代币的功能。该合约使用*访问控制*来控制对铸造和暂停功能的访问。部署合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色。

这个合约已经准备好部署，无需编写任何Solidity代码。它可用于快速原型设计和测试，但也适用于生产环境。

>NOTE
合约预设现已停用，更强大的替代方案是[Contracts Wizard](https://wizard.openzeppelin.com/)。