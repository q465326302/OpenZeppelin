# Contracts for Cairo
一个用于StarkNet的安全智能合约开发的Cairo库，StarkNet是一个去中心化的零知识Rollup解决方案。

## Usage
这个存储库包含高度实验性的代码。预计会进行快速迭代。**使用时请自行承担风险**。

### First time?
在安装Cairo之前，您需要安装gmp。
```
sudo apt install -y libgmp3-dev # linux
brew install gmp # mac
```

> TIP
如果您在Apple M1计算机上安装gmp时遇到任何问题，[这里是一些可能的解决方案列表](https://github.com/OpenZeppelin/nile/issues/22)。

### 设置你的项目
创建一个项目目录，然后进入该目录并创建一个Python虚拟环境。
```
mkdir my-project
cd my-project
python3 -m venv env
source env/bin/activate
```

安装[Nile](https://github.com/OpenZeppelin/nile)开发环境，然后运行init来启动一个新项目。Nile将创建项目目录结构并安装[Cairo语言](https://www.cairo-lang.org/docs/quickstart.html)、[本地网络](https://github.com/Shard-Labs/starknet-devnet/)和[测试框架](https://docs.pytest.org/en/6.2.x/)。
```
pip install cairo-nile
nile init
```

### 安装这个库
```
pip install openzeppelin-cairo-contracts
```

> WARNING直接通过GitHub安装可能包含不完整或破坏性的实现。虽然我们努力不引入这样的更改，但我们仍强烈建议通过官方发布进行安装。

### 使用基本预设
预设是可以立即部署的即用合约。它们还可以作为使用库模块的示例。了解更多关于*预设的信息*。
```
// contracts/MyToken.cairo

%lang starknet

from openzeppelin.token.erc20.presets.ERC20 import (
    constructor,
    name,
    symbol,
    totalSupply,
    decimals,
    balanceOf,
    allowance,
    transfer,
    transferFrom,
    approve,
    increaseAllowance,
    decreaseAllowance
)
```
立即编译和部署它。
```
nile compile

nile deploy MyToken <name> <symbol> <decimals> <initial_supply> <recipient> --alias my_token
```

> NOTE
<initial_supply> 预计是两个整数，例如 1 0。有关更多信息，请参阅 uint256。

### 
使用库模块编写自定义合同
*阅读更多关于库的内容。*
```
%lang starknet

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.cairo.common.uint256 import Uint256
from openzeppelin.security.pausable.library import Pausable
from openzeppelin.token.erc20.library import ERC20

(...)

@external
func transfer{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    recipient: felt, amount: Uint256
) -> (success: felt) {
    Pausable.assert_not_paused();
    return ERC20.transfer(recipient, amount);
}
```