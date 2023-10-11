# Security
这些合约旨在涵盖常见的安全实践。
* [PullPayment](#pullpayment)：这是一种可以用来避免重入攻击的模式。
* [ReentrancyGuard](#reentrancyguard)：这是一个修饰符，可以在某些函数中防止重入。
* [Pausable](#pausable)：这是一种常见的紧急响应机制，可以在进行修复期间暂停功能。

> TIP
有关重入以及可能的防范机制的概述，请阅读我们的文章[Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/).

## 合约

### PullPayment
```
import "@openzeppelin/contracts/security/PullPayment.sol";
```

这是一个简单的[拉取支付策略](https://consensys.github.io/smart-contract-best-practices/development-recommendations/general/external-calls/#favor-pull-over-push-for-external-calls)的实现，支付合约不直接与接收方账户交互，接收方必须自行提取支付。

从安全角度来看，拉取支付通常被认为是最佳实践。它防止接收方阻止执行，并消除了重入的担忧。

> TIP
如果你想了解更多关于重入和其他保护措施的信息，请查阅我们的博客文章[Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/).

使用时，继承PullPayment合约，并使用[_asyncTransfer](#_asynctransferaddress-dest-uint256-amount)代替Solidity的transfer函数。支付方可以使用[payments](#paymentsaddress-dest-→-uint256)查询其应付款项，并使用[withdrawPayments](#withdrawpaymentsaddress-payable-payee)提取款项。

**FUNCTIONS**
[constructor()](#constructor)
[withdrawPayments(payee)](#withdrawpaymentsaddress-payable-payee)
[payments(dest)](#paymentsaddress-dest-→-uint256)
[_asyncTransfer(dest, amount)](#_asynctransferaddress-dest-uint256-amount)

#### constructor()
internal#

#### withdrawPayments(address payable payee)
public#

提取累积的支付金额，并将所有的gas转给收款人。

请注意，任何账户都可以调用此函数，不仅仅是收款人。这意味着不了解PullPayment协议的合约仍然可以通过有一个单独的账户调用*withdrawPayments*来接收资金。

> WARNING
转发所有的gas会打开重入漏洞的大门。确保你信任收款人，或者遵循检查-效果-交互模式或使用[ReentrancyGuard](#reentrancyguard)。

#### payments(address dest) → uint256
public#
返回欠某个地址的付款。

#### _asyncTransfer(address dest, uint256 amount)
internal#
支付方调用该函数将发送的金额存储为可提取的信用额度。以这种方式发送的资金存储在一个[第三方](./Utils.md#escrow)的托管合约中，因此在提取之前不会有被花费的风险。

### ReentrancyGuard
```
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
```

防止重入调用合约模块。

继承ReentrancyGuard将使[nonReentrant](#nonreentrant)修饰符可用，可以应用于函数，以确保没有嵌套（重入）调用它们。

请注意，由于只有一个nonReentrant保护，标记为nonReentrant的函数可能不会相互调用。可以通过将这些函数设置为私有，并向其添加外部的nonReentrant入口点来解决此问题。

> TIP
如果你想了解更多关于重入和其他保护措施的信息，请查看我们的博客文章[Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

**MODIFIERS**
[nonReentrant()](#nonreentrant)

**FUNCTIONS**
[constructor()](#constructor)
[_reentrancyGuardEntered()](#_reentrancyguardentered-→-bool)

#### nonReentrant()
modifier#
防止合约直接或间接地调用自身。不支持从另一个非可重入函数调用非可重入函数。可以通过使非可重入函数为external，并使其调用执行实际工作的私有函数来防止发生这种情况。

#### constructor()
external#

#### _reentrancyGuardEntered() → bool
external#
如果重入保护当前设置为“进入”（entered），则返回true，这表示在调用堆栈中存在一个非重入函数。

### Pausable
```
import "@openzeppelin/contracts/security/Pausable.sol";
```

允许儿童实现由授权账户触发的紧急停止机制的合约模块。

通过继承使用此模块。它将提供 whenNotPaused 和 whenPaused 修饰器，可以应用于合约的函数。请注意，仅仅包含此模块不会使函数可暂停，只有在放置修饰器后才会生效。

**MODIFIERS**
[whenNotPaused()](#whennotpaused)
[whenPaused()](#whenpaused)

**FUNCTIONS**
[constructor()](#constructor)
[paused()](#paused-→-bool)
[_requireNotPaused()](#_requirenotpaused)
[_requirePaused()](#_requirepaused)
[_pause()](#_pause)
[_unpause()](#_unpause)

**EVENTS**
[Paused(account)](#pausedaddress-account)
[Unpaused(account)](#unpausedaddress-account)

#### whenNotPaused()
modifier#
当合约暂停时，使函数无法调用。

要求：
* 合约不能暂停。

#### whenPaused()
modifier#
仅当合约暂停时才能调用的修饰符。

要求：

合约必须暂停。

#### constructor()
internal#
在未暂停状态下初始化合约。

#### paused() → bool
public#
如果合约已暂停，则返回true，否则返回false。

#### _requireNotPaused()
internal#
如果合约被暂停，就会抛出异常。

#### _requirePaused()
internal#
如果合约没有暂停，则会抛出异常。

#### _pause()
internal#
触发停止状态。

要求：
* 合约不能暂停。

#### _unpause()
internal#
回到正常状态。

要求：
* 合约必须暂停。

#### Paused(address account)
event#
当账户触发暂停时发出的信号。

#### Unpaused(address account)
event#
当账户解除暂停时发出。
