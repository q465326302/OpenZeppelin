# Security
这些合同旨在涵盖常见的安全实践。
* *PullPayment*：这是一种可以用来避免重入攻击的模式。
* *ReentrancyGuard*：这是一个修饰符，可以在某些函数中防止重入。
* *Pausable*：这是一种常见的紧急响应机制，可以在进行修复期间暂停功能。

> TIP
有关重入以及可能的防范机制的概述，请阅读我们的文章*Reentrancy After Istanbul*.

## 合约

### PullPayment
```
import "@openzeppelin/contracts/security/PullPayment.sol";
```

这是一个简单的*拉取支付*策略的实现，支付合同不直接与接收方账户交互，接收方必须自行提取支付。

从安全角度来看，拉取支付通常被认为是最佳实践。它防止接收方阻塞执行，并消除了重入的担忧。

> TIP
如果您想了解更多关于重入和其他保护措施的信息，请查阅我们的博客文章*Reentrancy After Istanbul*.

使用时，继承PullPayment合同，并使用*_asyncTransfer*代替Solidity的transfer函数。支付方可以使用*payments*查询其应付款项，并使用*withdrawPayments*提取款项。

**FUNCTIONS**
constructor()
withdrawPayments(payee)
payments(dest)
_asyncTransfer(dest, amount)

#### constructor()
内部#

#### withdrawPayments(address payable payee)
公开#

提取累积的支付金额，并将所有的gas转给收款人。

请注意，任何账户都可以调用此函数，不仅仅是收款人。这意味着不了解PullPayment协议的合约仍然可以通过有一个单独的账户调用*withdrawPayments*来接收资金。

> WARNING
转发所有的gas会打开重入漏洞的大门。确保你信任收款人，或者遵循检查-效果-交互模式或使用*ReentrancyGuard*。

#### payments(address dest) → uint256
公开#
返回欠某个地址的付款。

#### _asyncTransfer(address dest, uint256 amount)
内部#
支付方调用该函数将发送的金额存储为可提取的信用额度。以这种方式发送的资金存储在一个*第三方*的托管合约中，因此在提取之前不会有被花费的风险。

### ReentrancyGuard
```
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
```
防止重入调用合约模块。

继承ReentrancyGuard将使*nonReentrant*修饰符可用，可以应用于函数，以确保没有嵌套（重入）调用它们。

请注意，由于只有一个nonReentrant保护，标记为nonReentrant的函数可能不会相互调用。可以通过将这些函数设置为私有，并向其添加外部的nonReentrant入口点来解决此问题。

> TIP
如果您想了解更多关于重入和其他保护措施的信息，请查看我们的博客文章[Reentrancy After Istanbul](https://blog.openzeppelin.com/reentrancy-after-istanbul/)。

**MODIFIERS**
nonReentrant()

**FUNCTIONS**
constructor()
_reentrancyGuardEntered()

#### nonReentrant()
修饰符#
防止合约直接或间接地调用自身。不支持从另一个非可重入函数调用非可重入函数。可以通过使非可重入函数为external，并使其调用执行实际工作的私有函数来防止发生这种情况。

#### constructor()
外部# 

#### _reentrancyGuardEntered() → bool
外部# 
如果重入保护当前设置为“进入”（entered），则返回true，这表示在调用堆栈中存在一个非重入函数。

### Pausable
```
import "@openzeppelin/contracts/security/Pausable.sol";
```
允许儿童实现由授权账户触发的紧急停止机制的合约模块。

通过继承使用此模块。它将提供 whenNotPaused 和 whenPaused 修饰器，可以应用于合约的函数。请注意，仅仅包含此模块不会使函数可暂停，只有在放置修饰器后才会生效。

**MODIFIERS**
whenNotPaused()
whenPaused()

**FUNCTIONS**
constructor()
paused()
_requireNotPaused()
_requirePaused()
_pause()
_unpause()

**EVENTS**
Paused(account)
Unpaused(account)

#### whenNotPaused()
修饰符#
当合约暂停时，使函数无法调用。

要求：
* 合约不能暂停。

#### whenPaused()
修饰符#
仅当合同暂停时才能调用的修饰符。

要求：

合同必须暂停。

#### constructor()
内部#
在未暂停状态下初始化合同。

#### paused() → bool
公开#
如果合同已暂停，则返回true，否则返回false。

#### _requireNotPaused()
内部#
如果合约被暂停，就会抛出异常。

#### _requirePaused()
内部#
如果合约没有暂停，则会抛出异常。

#### _pause()
内部#
触发停止状态。

要求：
* 合同不能暂停。

#### _unpause()
内部#
回到正常状态。

要求：
* 合同必须暂停。

#### Paused(address account)
事件#
当账户触发暂停时发出的信号。

#### Unpaused(address account)
事件#
当账户解除暂停时发出。
