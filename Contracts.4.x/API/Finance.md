# Finance
该目录包括金融系统的基本组件：

* PaymentSplitter允许将以太币和ERC20支付拆分到一组账户中。发送者不需要知道资产将以这种方式拆分，因为合约会透明地处理。拆分可以平均分配或任意其他比例。

* VestingWallet处理给定受益人的以太币和ERC20代币的解锁。可以将多个代币的监管权交给该合约，该合约将根据给定的、可自定义的解锁时间表将代币释放给受益人。

## 合约

### PaymentSplitter
```
import "@openzeppelin/contracts/finance/PaymentSplitter.sol";
```
此合约允许将以太币支付分配给一组账户。发送者无需知道以太币将以这种方式分配，因为合约会透明地处理它。

分配可以是等份的，也可以是任意其他比例。指定方式是将每个账户分配给一定数量的股份。对于此合约收到的所有以太币，每个账户将能够按照其分配的总股份百分比索取相应数量的以太币。股份的分配在合约部署时设置，之后无法更新。

PaymentSplitter遵循拉取付款模型。这意味着付款不会自动转发到账户，而是保留在此合约中，并通过调用*释放*函数来触发实际转移的单独步骤。

> NOTE
此合约假定ERC20代币将类似于本机代币（以太币）的行为。重新调整代币和在转移期间应用费用的代币可能不会按预期受到支持。如有疑问，请在向此合约发送真正的价值之前进行测试。

**FUNCTIONS**
constructor(payees, shares_)
receive()
totalShares()
totalReleased()
totalReleased(token)
shares(account)
released(account)
released(token, account)
payee(index)
releasable(account)
releasable(token, account)
release(account)
release(token, account)

**EVENTS**
PayeeAdded(account, shares)
PaymentReleased(to, amount)
ERC20PaymentReleased(token, to, amount)
PaymentReceived(from, amount)

#### constructor(address[] payees, uint256[] shares_)
公开#
创建一个PaymentSplitter实例，其中payees中的每个帐户都被分配在shares数组中匹配位置的股份数。

payees中的所有地址必须是非零的。两个数组必须具有相同的非零长度，并且payees中不能有重复项。

#### receive()
外部#
接收到的以太币将被记录在*PaymentReceived*事件中。请注意，这些事件并不完全可靠：合约可以收到以太币而不触发此函数。这仅影响事件的可靠性，而不影响以太币的实际分割。

要了解更多信息，请参阅[Solidity文档](https://solidity.readthedocs.io/en/latest/contracts.html#fallback-function)中的回退函数。

#### totalShares() → uint256
公开#
获取支付受款人持有的总股数。

#### totalReleased() → uint256
公开#
获取已经释放的以太总量的方法。

#### totalReleased(contract IERC20 token) → uint256
公开#
获取已释放的代币总量的Getter函数。代币应该是一个IERC20合约的地址。

#### shares(address account) → uint256
公开#
获取账户持有的股票数量的方法。

#### released(address account) → uint256
公开#
获取已经释放给收款人的以太币数量。

#### released(contract IERC20 token, address account) → uint256
公开#
获取已经释放给收款人的代币令牌数量。令牌应该是一个IERC20合约的地址。

#### payee(uint256 index) → address
公开#
获取收款人号码索引的地址。

#### releasable(address account) → uint256
公开#
获取收款人可释放的以太币数量。

#### releasable(contract IERC20 token, address account) → uint256
公开#
获取收款人可释放代币的数量。代币应该是一个IERC20合约的地址。

#### release(address payable account)
公开#
触发将他们应得的以太币数量转移至账户，根据他们在总股份中的百分比和之前的提款。

#### release(contract IERC20 token, address account)
公开#
触发将他们应得的令牌数量转账到其账户，根据其在总份额中的百分比和之前的提取情况。令牌必须是一个IERC20合约的地址。

#### PayeeAdded(address account, uint256 shares)
事件#

#### PaymentReleased(address to, uint256 amount)
事件#

#### ERC20PaymentReleased(contract IERC20 indexed token, address to, uint256 amount)
事件#

#### PaymentReceived(address from, uint256 amount)
事件#

### VestingWallet
```
import "@openzeppelin/contracts/finance/VestingWallet.sol";
```

这个合同处理了给定受益人的Eth和ERC20代币的解锁。可以将多个代币的托管交给这个合同，它将根据给定的解锁计划释放代币给受益人。解锁计划可以通过*vestedAmount*函数进行自定义。

任何转移到此合同的代币都将按照从一开始就锁定的解锁计划进行解锁。因此，如果解锁已经开始，发送到此合同的任何代币数量都将（至少部分）立即可释放。

**FUNCTIONS**
constructor(beneficiaryAddress, startTimestamp, durationSeconds)
receive()
beneficiary()
start()
duration()
released()
released(token)
releasable()
releasable(token)
release()
release(token)
vestedAmount(timestamp)
vestedAmount(token, timestamp)
_vestingSchedule(totalAllocation, timestamp)

**EVENTS**
EtherReleased(amount)
ERC20Released(token, amount)

#### constructor(address beneficiaryAddress, uint64 startTimestamp, uint64 durationSeconds)
公开#
设定受益人、开始时间戳和锁定期的锁定钱包。

#### receive()
外部#
合约应该能够接收以太币。

#### beneficiary() → address
公开#
获取受益人地址的方法。

#### start() → uint256
公开#
获取开始时间戳。

#### duration() → uint256
公开#
获取归属期限的方法。

#### released() → uint256
公开#
已经释放的以太币数量

#### released(address token) → uint256
公开#
已发布的代币数量

#### releasable() → uint256
公开#
获取可释放以太币的数量的函数。

#### releasable(address token) → uint256
公开#
获取可释放代币的数量的函数。token应该是一个IERC20合约的地址。

#### release()
公开#
释放已经归属的本机代币（以太币）。

发出一个*EtherReleased*事件。

#### release(address token)
公开#
释放已经归属的代币。

发出*ERC20Released*事件。

#### vestedAmount(uint64 timestamp) → uint256
公开#
计算已经归属的以太币数量。默认实现是线性归属曲线。

#### vestedAmount(address token, uint64 timestamp) → uint256
公开#
计算已经归属的代币数量。默认实现是线性归属曲线。

#### _vestingSchedule(uint256 totalAllocation, uint64 timestamp) → uint256
内部#
实现分配公式的虚拟化。根据资产的总历史分配，这将返回随时间变化的已分配金额。

#### EtherReleased(uint256 amount)
事件#

#### ERC20Released(address indexed token, uint256 amount)
事件#
