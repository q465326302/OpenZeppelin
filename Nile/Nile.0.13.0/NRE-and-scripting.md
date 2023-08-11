# NRE and scripting
Nile提供了一个API，用于构建利用Nile运行时环境（NRE）的脚本，该环境是一个对象，从编译到账户和部署管理都暴露了Nile的功能。Nile脚本实现了一个异步的def run(nre)函数，该函数接收NileRuntimeEnvironment对象。

## 运行脚本
1. 在项目的根目录下的scripts/目录中创建一个名为test_script.py的文件，并将以下内容添加到文件中。请记住，scripts/目录是一个建议的约定，但不是必需的。
```
async def run(nre):
    accounts = await nre.get_accounts(predeployed=True)
    print("First Account:", accounts[0])
```
> NOTE
predeployed参数利用了本地节点已经部署的账户。请阅读*get_accounts的API参考文档*以了解更多信息。

2. 使用nile node运行一个开发网络节点。
```
nile node
```
3. 通过使用nile run运行脚本
```
nile run scripts/test_script.py
```
> NOTE
您可以使用--network选项更改脚本的目标网络。
有关NRE公开成员的完整参考，请参阅*NRE参考部分*。

## 有用的脚本示例
在本节中，您可以找到一些可能有用的脚本示例。

### 声明OZ账户
如果您需要在本地devnet节点中部署账户，可以使用此选项进行先前声明。
```
# scripts/declare_oz_account.py
async def run(nre):
  accounts = await nre.get_accounts(predeployed=True)
  declarer_account = accounts[0]

  # nile_account flag tells Nile to use its pre-packed artifact
  #
  # If we don't pass a max_fee, nile will estimate the transaction
  # fee by default. This line is equivalent to:
  #
  # tx = await declarer_account.declare("Account", max_fee=0, nile_account=True)
  # max_fee = await tx.estimate_fee()
  # tx.update_fee(max_fee)
  #
  # Note that tx.update_fee will update tx.hash and tx.max_fee members
  tx = await declarer_account.declare("Account", nile_account=True)

  tx_status, *_ = await tx.execute(watch_mode="track")

  print(tx_status.status, tx_status.error_message or "")
```

### 从预部署的开发网络账户转移资金
用于在开发网络中给地址提供资金:
```
# scripts/transfer_funds.py
from nile.common import ETH_TOKEN_ADDRESS
from nile.utils import to_uint, hex_address

async def run(nre):
  accounts = await nre.get_accounts(predeployed=True)
  account = accounts[0]

  # define the recipient address
  recipient = "0x05a0ca51cbc03e5ec8d9fad116f8737a6afe2613b3128ebd515643a1a5e5c52d"

  # define the amount to transfer
  amount = 2 * 10 ** 18

  print(
    f"Transferring {amount} WEI\n"
    f"from {hex_address(account.address)}\n"
    f"to   {recipient}\n"
  )

  # If we don't pass a max_fee, nile will estimate the transaction fee by default
  tx = await account.send(ETH_TOKEN_ADDRESS, "transfer", [recipient, *to_uint(amount)])

  tx_status, *_ = await tx.execute(watch_mode="track")

  print(tx_status.status, tx_status.error_message or "")
```