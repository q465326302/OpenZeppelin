# Payment
与发送和接收付款相关的实用程序。例如，[PullPayment](#pullpayment)在向第三方发送资金时实施了最佳安全实践，而[PaymentSplitter](#paymentsplitter)则用于将收到的付款分配给多个受益人。

> TIP
在与不受信任的第三方进行资金转移时，总会存在重入的安全风险。如果你想了解更多关于此问题及其防范措施的信息，请查看我们的博客文章[Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

## 实用程序

### PaymentSplitter
该合约允许将以太付款分割给一组帐户。发送者无需知道以太将以这种方式分割，因为合约会透明地处理。

分割可以是均等的部分，也可以是任意的比例。指定方式是将每个帐户分配给一定数量的股份。对于该合约收到的所有以太，每个帐户将能够按照其分配的总股份的百分比来索取相应的金额。

PaymentSplitter遵循拉取付款模式。这意味着付款不会自动转发到帐户，而是保留在该合约中，并通过调用 [release](#releaseaddress-payable-account)函数来触发实际的转账步骤。

**FUNCTIONS**
[constructor(payees, shares_)](#constructoraddress-payees-uint256-shares_)

[receive()](#receive)

[totalShares()](#totalshares-→-uint256)

[totalReleased()](#totalreleased-→-uint256)

[shares(account)](#sharesaddress-account-→-uint256)

[released(account)](#releasedaddress-account-→-uint256)

[payee(index)](#payeeuint256-index-→-address)

[release(account)](#releaseaddress-payable-account)

**EVENTS**
[PayeeAdded(account, shares)](#payeeaddedaddress-account-uint256-shares)

[PaymentReleased(to, amount)](#paymentreleasedaddress-to-uint256-amount)

[PaymentReceived(from, amount)](#paymentreceivedaddress-from-uint256-amount)

#### constructor(address[] payees, uint256[] shares_)
public#
创建一个PaymentSplitter的实例，其中payees中的每个账户都被分配到shares数组中相应位置的份额。

payees中的所有地址必须非零。两个数组的非零长度必须相同，并且payees中不能有重复的地址。

#### receive()
外部#
接收到的以太将被记录在[PaymentReceived](#paymentreceivedaddress-from-uint256-amount)事件中。请注意，这些事件并不完全可靠：合约可能会收到以太而不触发此函数。这只影响事件的可靠性，而不影响以太的实际分割。

要了解更多信息，请参阅Solidity文档中的[回退函数部分](https://solidity.readthedocs.io/en/latest/contracts.html#fallback-function)。

#### totalShares() → uint256
public#
获取支付人持有的股份总数。

#### totalReleased() → uint256
public# 
以太已经释放的总量的获取器。

#### shares(address account) → uint256
public#
获取账户持有的股份数量。

#### released(address account) → uint256
public#
获取已释放给收款人的以太数量。

#### payee(uint256 index) → address
public#
获取收款人号码索引的地址。

#### release(address payable account)
public#
根据他们在总股份中所占的比例和之前的提款情况，触发将他们所欠的以太转账到他们的账户。

#### PayeeAdded(address account, uint256 shares)
event#

#### PaymentReleased(address to, uint256 amount)
event#

#### PaymentReceived(address from, uint256 amount)
event#

### PullPayment
简单实现了一种[拉取支付](https://consensys.github.io/smart-contract-best-practices/recommendations/#favor-pull-over-push-for-external-calls)策略，其中支付合约不直接与接收方账户进行交互，接收方必须自行提取支付。

从安全角度考虑，拉取支付通常被认为是最佳实践。它可以防止接收方阻止执行，并消除了可重入性的担忧。

> TIP
如果你想了解更多关于可重入性的信息以及防范可重入性的替代方法，请查看我们的博客文章[《Reentrancy After Istanbul》](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

要使用该功能，请从PullPayment合约派生，并使用[_asyncTransfer](#_asynctransferaddress-dest-uint256-amount)代替Solidity的transfer函数。收款方可以使用[payments](#paymentsaddress-dest-→-uint256)查询到他们应收的支付款项，并使用[withdrawPayments](#withdrawpaymentsaddress-payable-payee)提取这些款项。

**FUNCTIONS**
[constructor()](#constructor)

[withdrawPayments(payee)](#withdrawpaymentsaddress-payable-payee)

[payments(dest)](#paymentsaddress-dest-→-uint256)

[_asyncTransfer(dest, amount)](#_asynctransferaddress-dest-uint256-amount)

#### constructor()
internal#

#### withdrawPayments(address payable payee)
public#
撤回累积的支付，将所有的gas转发给收款人。

请注意，任何账户都可以调用此函数，不仅限于收款人。这意味着即使不了解PullPayment协议的合约也可以通过让一个单独的账户调用[withdrawPayments](#withdrawpaymentsaddress-payable-payee)来接收资金。

> WARNING
转发所有的gas会打开可重入性漏洞的大门。请确保信任收款人，或者遵循检查-效果-交互模式或使用[ReentrancyGuard](./Utils.md#reentrancyguard)。

#### payments(address dest) → uint256
public#
返回到某个地址所欠款项的支付。

#### _asyncTransfer(address dest, uint256 amount)
internal#
支付方呼叫将已发送的金额存储为待提取的信用额度。以这种方式发送的资金将存储在一个[Escrow](#escrow)合约中，因此不存在在提取之前被使用的危险。

## Escrow

### Escrow
基本托管合约，将资金保留给收款人，直到他们提取为止。

预期使用方式：该合约（以及派生的托管合约）应该是一个独立的合约，只与实例化它的合约交互。这样，可以保证所有以太都按照托管规则处理，无需检查继承树中的可支付函数或转账。使用托管作为支付方式的合约应该是其所有者，并提供公共方法重定向到托管的存款和提取。

**FUNCTIONS**
[depositsOf(payee)](#depositsofaddress-payee-→-uint256)

[deposit(payee)](#depositaddress-payee)

[withdraw(payee)](#withdrawaddress-payable-payee)

OWNABLE
[constructor()](./Access.md#constructor)

[owner()](./Access.md#ownable)

[renounceOwnership()](./Access.md#renounceownership)

[transferOwnership(newOwner)](./Access.md#transferownershipaddress-newowner)

**EVENTS**
[Deposited(payee, weiAmount)](#depositedaddress-payee-uint256-weiamount)

[Withdrawn(payee, weiAmount)](#withdrawnaddress-payee-uint256-weiamount)

OWNABLE
[OwnershipTransferred(previousOwner, newOwner)](./Access.md#ownershiptransferredaddress-previousowner-address-newowner)

#### depositsOf(address payee) → uint256
public#

#### deposit(address payee)
public#
将发送的金额存储为可提取的信用。

#### withdraw(address payable payee)
public#
提取收款人的累积余额，将所有燃料转发给收款人。

> WARNING
将所有燃料转发给收款人会导致可重新进入漏洞。请确保你信任收款人，或者遵循检查-效果-交互模式或使用[ReentrancyGuard](./Utils.md#reentrancyguard)。

#### Deposited(address payee, uint256 weiAmount)
event#

#### Withdrawn(address payee, uint256 weiAmount)
event#

### ConditionalEscrow
基本抽象托管只允许在满足条件时进行提款。预期用途：参见[Escrow](#escrow)。同样的使用准则适用于这里。

**FUNCTIONS**
[withdrawalAllowed(payee)](#withdrawalallowedaddress-payee-→-bool)

[withdraw(payee)](#withdrawaddress-payable-payee)

ESCROW
[depositsOf(payee)](#depositsofaddress-payee-→-uint256)

[deposit(payee)](#depositaddress-payee)

OWNABLE
[constructor()](./Access.md#constructor)

[owner()](./Access.md#ownable)

[renounceOwnership()](./Access.md#renounceownership)

[transferOwnership(newOwner)](./Access.md#transferownershipaddress-newowner)

**EVENTS**
ESCROW
[Deposited(payee, weiAmount)](#depositedaddress-payee-uint256-weiamount)

[Withdrawn(payee, weiAmount)](#withdrawnaddress-payee-uint256-weiamount)

OWNABLE
[OwnershipTransferred(previousOwner, newOwner)](./Access.md#ownershiptransferredaddress-previousowner-address-newowner)

#### withdrawalAllowed(address payee) → bool
public#
返回一个地址是否被允许提取其资金。由派生合约实施。

#### withdraw(address payable payee)
public#

### RefundEscrow
托管款项给受益人的托管账户，款项由多个参与方存入。预期用途：参见 [Escrow](#escrow)。同样的使用指南适用于此处。所有与退款托管交互都将通过拥有者合约进行。拥有者账户（即实例化此合约的合约）可以存入款项，关闭存款期，并允许受益人提款或向存款人退款。

**FUNCTIONS**
[constructor(beneficiary_)](#constructoraddress-payable-beneficiary_)

[state()](#state-→-enum-refundescrowstate)

[beneficiary()](#beneficiary-→-address-payable)

[deposit(refundee)](#depositaddress-refundee)

[close()](#close)

[enableRefunds()](#enablerefunds)

[beneficiaryWithdraw()](#beneficiarywithdraw)

[withdrawalAllowed(_)](#withdrawalallowedaddress-→-bool)

CONDITIONALESCROW
[withdraw(payee)](#withdrawaddress-payable-payee)

ESCROW
[depositsOf(payee)](#depositsofaddress-payee-→-uint256)

OWNABLE
[owner()](./Access.md#owner-→-address)

[renounceOwnership()](./Access.md#renounceownership)

[transferOwnership(newOwner)](./Access.md#transferownershipaddress-newowner)

**EVENTS**
[RefundsClosed()](#refundsclosed)

[RefundsEnabled()](#refundsenabled)

ESCROW
[Deposited(payee, weiAmount)](#depositedaddress-payee-uint256-weiamount)

[Withdrawn(payee, weiAmount)](#withdrawnaddress-payee-uint256-weiamount)

OWNABLE
[OwnershipTransferred(previousOwner, newOwner)](./Access.md#ownershiptransferredaddress-previousowner-address-newowner)

#### constructor(address payable beneficiary_)
public#
构造函数。

#### state() → enum RefundEscrow.State
public#

#### beneficiary() → address payable
public#

#### deposit(address refundee)
public#
存储可能在以后退还的资金。

#### close()
public#
允许受益人提取他们的资金，并拒绝进一步的存款。

#### enableRefunds()
public#
允许退款，并拒绝进一步的存款。

#### beneficiaryWithdraw()
public#
提取受益人的资金。

#### withdrawalAllowed(address) → bool
public#
返回退款人是否可以取回他们的存款（被退款）。重写的函数接收一个'payee'参数，但在这里我们忽略它，因为条件是全局的，而不是每个收款人的。

#### RefundsClosed()
event#

#### RefundsEnabled()
event#