# Payment

> NOTE
这个页面尚未完成。我们正在努力改进它，以便在下一次发布中完善。请继续关注！

## Utilities

### PaymentSplitter
这份合同允许将以太币支付分成一组账户。发送者不需要知道以太币将以这种方式分割，因为合同会透明地处理。

分割可以是等份的，也可以是任意其他比例。指定方式是为每个账户分配一定数量的股份。当这个合同收到的以太币时，每个账户将能够按照其被分配的总股份的百分比来索取相应金额的以太币。

PaymentSplitter遵循拉取支付模型。这意味着支付不会自动转发到账户，而是保留在合同中，并通过调用*释放*函数来触发实际的转账步骤。

**FUNCTIONS**
constructor(payees, shares)

fallback()

totalShares()

totalReleased()

shares(account)

released(account)

payee(index)

release(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
PayeeAdded(account, shares)

PaymentReleased(to, amount)

PaymentReceived(from, amount)

#### constructor([.var-type]#address[# payees, uint256[] shares)]
公开#
创建一个PaymentSplitter的实例，其中payees中的每个账户被分配到shares数组中相应位置的股份数量。

payees中的所有地址必须非零。两个数组的长度必须相同且非零，并且payees中不能有重复的地址。

#### fallback()
外部#
接收到的以太币将被记录在*PaymentReceived*事件中。请注意，这些事件并不完全可靠：合约可能会收到以太币而不触发此函数。这只影响事件的可靠性，而不影响以太币的实际分割。

要了解更多信息，请参阅Solidity文档中的[回退函数](https://solidity.readthedocs.io/en/latest/contracts.html#fallback-function)部分。

#### totalShares() → uint256
公开#
获取支付人持有的总股份的方法。

#### totalReleased() → uint256
公开#
以太币的总发行量。

#### shares(address account) → uint256
公开#

#### released(address account) → uint256
公开#
获取已经释放给收款人的以太币数量。

#### payee(uint256 index) → address
公开#
获取收款人编号索引的地址。

#### release(address payable account)
公开#

#### PayeeAdded(address account, uint256 shares)
事件#

#### PaymentReleased(address to, uint256 amount)
事件#

#### PaymentReceived(address from, uint256 amount)
事件#

### PullPayment
简单实现了一种[拉取支付](https://consensys.github.io/smart-contract-best-practices/recommendations/#favor-pull-over-push-for-external-calls)策略，支付合约不直接与接收方账户进行交互，接收方必须自行提取支付。

从安全角度考虑，拉取支付通常被认为是最佳实践。它可以防止接收方阻塞执行，并消除了重入的担忧。

> TIP
如果您想了解更多关于重入和其他保护措施的信息，请查看我们的博客文章[ Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

要使用该功能，请从PullPayment合约派生，并使用*_asyncTransfer*代替Solidity的transfer函数。收款人可以使用*payments*查询其应付款项，并使用*withdrawPayments*提取款项。

**FUNCTIONS**
constructor()

withdrawPayments(payee)

withdrawPaymentsWithGas(payee)

payments(dest)

_asyncTransfer(dest, amount)

#### constructor()
内部#

#### withdrawPayments(address payable payee)
公开#
提取累积支付。

请注意，任何账户都可以调用此函数，不仅限于收款人。这意味着不了解PullPayment协议的合约仍然可以通过让一个单独的账户调用*withdrawPayments*来接收资金。

> NOTE
此函数已被弃用，请改用*withdrawPaymentsWithGas*。使用固定的gas限制调用合约是一种反模式，并可能在网络升级（硬分叉）中破坏合约交互。了解[更多信息](https://diligence.consensys.net/blog/2019/09/stop-using-soliditys-transfer-now/)。

#### withdrawPaymentsWithGas(address payable payee)
外部#
与*withdrawPayments*相同，但将所有gas转发给收件人。

> WARNING
转发所有gas会打开重入漏洞的大门。确保您信任收件人，或者遵循检查-效果-交互模式或使用ReentrancyGuard。

*自v2.4.0起可用。*

#### payments(address dest) → uint256
公开#
返回欠付给某个地址的付款。

#### _asyncTransfer(address dest, uint256 amount)
内部#
支付方通过调用此函数将发送的金额存储为待提取的信用额度。以这种方式发送的资金将存储在一个*中间的托管*合约中，因此在提取之前不会有被使用的风险。

## Escrow

### Escrow
基本托管合约，将资金保留给收款人，直到他们提取。

预期使用：该合约（以及派生的托管合约）应该是一个独立的合约，只与实例化它的合约交互。这样，可以确保所有以太币都按照托管规则处理，无需检查继承树中的可支付函数或转账。使用托管作为付款方式的合约应该是主要的，并提供公共方法重定向到托管的存款和提取。

**MODIFIERS**
SECONDARY
onlyPrimary()

**FUNCTIONS**
depositsOf(payee)

deposit(payee)

withdraw(payee)

withdrawWithGas(payee)

SECONDARY
constructor()

primary()

transferPrimary(recipient)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
Deposited(payee, weiAmount)

Withdrawn(payee, weiAmount)

SECONDARY
PrimaryTransferred(recipient)

#### depositsOf(address payee) → uint256
公开#

#### deposit(address payee)
公开#
将发送的金额存储为可提取的信用。

#### withdraw(address payable payee)
公开#
提取收款人的累积余额，并转发2300个gas（一个Solidity转账）。

> NOTE
此函数已被弃用，请改用*withdrawWithGas*。使用固定gas限制调用合约是一种反模式，并可能在网络升级（硬分叉）中破坏合约交互。了解[更多信息](https://diligence.consensys.net/blog/2019/09/stop-using-soliditys-transfer-now/)。

#### withdrawWithGas(address payable payee)
公开#
与*withdraw*相同，但将所有gas转发给收件人。

> WARNING
转发所有gas会打开重入漏洞的大门。确保您信任收件人，或者遵循检查-效果-交互模式或使用*ReentrancyGuard*。

*自v2.4.0起可用。*

#### Deposited(address payee, uint256 weiAmount)
事件#

#### Withdrawn(address payee, uint256 weiAmount)
事件#

### ConditionalEscrow
基本抽象的托管只允许在满足条件的情况下提取。预期用途：参见*托管*。同样适用于此处的使用准则。

**MODIFIERS**
SECONDARY
onlyPrimary()

**FUNCTIONS**
withdrawalAllowed(payee)

withdraw(payee)

ESCROW
depositsOf(payee)

deposit(payee)

withdrawWithGas(payee)

SECONDARY
constructor()

primary()

transferPrimary(recipient)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
ESCROW
Deposited(payee, weiAmount)

Withdrawn(payee, weiAmount)

SECONDARY
PrimaryTransferred(recipient)

#### withdrawalAllowed(address payee) → bool
公开#
返回一个地址是否被允许提取资金。由派生合约实施。

#### withdraw(address payable payee)
公开#

### RefundEscrow
托管款项以供受益人使用，这些款项来自多方存款。预期用途：参见托管。同样的使用准则适用于此处。主要账户（即实例化此合约的合约）可以存款、关闭存款期，并允许受益人提款或向存款人退款。与RefundEscrow的所有交互都将通过主合约进行。参见RefundableCrowdsale合约，了解RefundEscrow的使用示例。

**MODIFIERS**
SECONDARY
onlyPrimary()

**FUNCTIONS**
constructor(beneficiary)

state()

beneficiary()

deposit(refundee)

close()

enableRefunds()

beneficiaryWithdraw()

withdrawalAllowed(_)

CONDITIONALESCROW
withdraw(payee)

ESCROW
depositsOf(payee)

withdrawWithGas(payee)

SECONDARY
primary()

transferPrimary(recipient)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
RefundsClosed()

RefundsEnabled()

ESCROW
Deposited(payee, weiAmount)

Withdrawn(payee, weiAmount)

SECONDARY
PrimaryTransferred(recipient)

#### constructor(address payable beneficiary)
公开#
构造函数。

#### state() → enum RefundEscrow.State
公开#

#### beneficiary() → address
公开#

#### deposit(address refundee)
公开#
存储可能在以后退还的资金。

#### close()
公开#
允许受益人提取他们的资金，并拒绝进一步的存款。

#### enableRefunds()
公开#
允许退款，并拒绝进一步的存款。

#### beneficiaryWithdraw()
公开#
取出受益人的资金。

#### withdrawalAllowed(address) → bool
公开#
返回退款人是否可以提取他们的存款（被退款）。重写的函数接收一个“收款人”参数，但我们在这里忽略它，因为条件是全局的，而不是每个收款人。

#### RefundsClosed()
事件#

#### RefundsEnabled()
事件#
