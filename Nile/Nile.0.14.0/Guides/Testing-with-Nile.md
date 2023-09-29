# Testing with Nile
使用Nile时，通常会使用[pytest](https://docs.pytest.org/)和[StarkNet](https://github.com/starkware-libs/cairo-lang/tree/master/src/starkware/starknet/testing)测试框架进行测试，即使它是框架无关的，也不会干扰其他工具。

> NOTE
请记住，将来可以通过插件集成不同的测试模块。

## 概述
这是基本项目中提供的示例测试文件（由nile init创建）:
```
"""contract.cairo test file."""
import os

import pytest
from starkware.starknet.testing.starknet import Starknet

# The path to the contract source code.
CONTRACT_FILE = os.path.join("contracts", "contract.cairo")


# The testing library uses python's asyncio. So the following
# decorator and the ``async`` keyword are needed.
@pytest.mark.asyncio
async def test_increase_balance():
    """Test increase_balance method."""
    # Create a new StarkNet class that simulates the StarkNet
    # system.
    starknet = await Starknet.empty()

    # Deploy the contract.
    contract = await starknet.deploy(
        source=CONTRACT_FILE,
    )

    # Invoke increase_balance() twice.
    await contract.increase_balance(amount=10).execute()
    await contract.increase_balance(amount=20).execute()

    # Check the result of get_balance().
    execution_info = await contract.get_balance().call()
    assert execution_info.result == (30,)
```

根据pytest的要求，测试文件必须以test_开头或以_test结尾。你可以通过运行pytest来运行测试套件。
```
pytest
```

> TIP
要了解如何测试复杂项目的参考资料，你可以查看[cairo-contracts](https://github.com/OpenZeppelin/cairo-contracts)存储库。

## 处理慢速测试
我们非常清楚的一个问题是，当你有大量测试时，运行测试会变得非常缓慢。我们强烈建议使用[pytest-xdist](https://pytest-xdist.readthedocs.io/en/latest/)来加快这个过程。

### 使用xdist进行并行处理
1. 在你的项目中安装pytest-xdist。
```
python -m pip install pytest-xdist
```

2. 在项目的根目录中添加一个pytest.ini文件，并配置如下内容
```
# pytest.ini
[pytest]
addopts = -n auto
```

3. 完成！现在每次使用pytest运行测试时，它们将并行运行，大大加快了进程。

## Coverage Reports
默认情况下，Nile不支持coverage报告，，但是你可以使用[nile-coverage](https://github.com/ericnordelo/nile-coverage)插件轻松集成此功能。

1. 在你的项目中安装nile-coverage插件。
```
python -m pip install nile-coverage
```

2. 使用coverage命令（来自插件）生成coverage报告。
```
nile coverage

Filename                    Stmts    Miss  Cover    Missing
------------------------  -------  ------  -------  ---------
contracts/contract.cairo        8       0  100.00%
TOTAL                           8       0  100.00%
```

3. 完成！

> TIP
使用nile-coverage，你可以轻松地与[Codecov](https://codecov.io/)等工具集成。请查阅[插件文档](https://github.com/ericnordelo/nile-coverage/blob/main/README.md)以了解选项的参考。