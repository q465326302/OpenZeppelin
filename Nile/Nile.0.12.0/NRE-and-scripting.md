# NRE and scripting
Nile提供了一个API，用于构建利用Nile运行时环境（NRE）的脚本，这是一个对象，从编译到账户和部署管理，都使用了Nile的功能。Nile脚本实现了一个异步的def run(nre)函数，该函数接收NileRuntimeEnvironment对象。

## 运行脚本
1. 在项目的根目录下的scripts/目录中创建一个名为test_script.py的文件，并将以下内容添加到文件中。请记住，scripts/目录是一个建议，不是必需的。
```
async def run(nre):
    accounts = await nre.get_accounts(predeployed=True)
    print("First Account:", accounts[0])
```

> NOTE
predeployed参数利用了本地节点已经部署的账户。请阅读[get_accounts](./API-Reference/NRE-Reference.md#get_accounts)的API参考文档以了解更多信息。

2. 使用nile node运行一个开放网络节点。
```
nile node
```

3. 通过使用nile run运行脚本
```
nile run scripts/test_script.py
```

> NOTE
你可以使用--network选项更改脚本的目标网络。
有关NRE公开成员的完整参考，请参阅[NRE参考部分](./API-Reference/NRE-Reference.md)。

## 有用的脚本示例
在本节中，你可以找到一些可能有用的脚本示例。

### 声明OZ账户
如果你需要在本地devnet节点中部署账户，可以使用此选项进行先前声明。
```
# script.py
async def run(nre):
  accounts = await nre.get_accounts(predeployed=True)
  declarer_account = accounts[0]

  # nile_account flag tells Nile to use its pre-packed artifact
  output = await declarer_account.declare("Account", nile_account=True)
  print(output)
```

### 从预部署的开发网络账户转移资金
用于在开发网络中给地址提供资金:
```
from nile.common import ETH_TOKEN_ADDRESS

async def run(nre):
    accounts = await nre.get_accounts(predeployed=True)
    account = accounts[0]

    # define the recipient address
    recipient = "0x053edde5384e39bad137d3c4130c708fb96ee28a4c80bf28049c486d3f369"

    # define the amount to transfer (uint256 format)
    amount = [2 * 10 ** 18, 0]

    print(f"Transferring {amount} to {recipient} from {hex(account.address)}")

    # send the transfer transaction
    output = await account.send(ETH_TOKEN_ADDRESS, "transfer", [recipient, *amount])
    print(output)
```