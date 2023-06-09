# Lifecycle

## Pausable

### Pausable
合约模块，允许子合约实现由授权账户触发的紧急停止机制。

通过继承使用该模块。它将提供 whenNotPaused 和 whenPaused 修饰符，可以应用于合约的函数。请注意，仅仅包括此模块并不能使函数可暂停，只有一旦修饰符被放置，函数才会变得可暂停。

**MODIFIERS**
whenNotPaused()

whenPaused()

PAUSERROLE
onlyPauser()

**FUNCTIONS**
constructor()

paused()

pause()

unpause()

PAUSERROLE
isPauser(account)

addPauser(account)

renouncePauser()

_addPauser(account)

_removePauser(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
Paused(account)

Unpaused(account)

PAUSERROLE
PauserAdded(account)

PauserRemoved(account)

#### whenNotPaused()
修饰符#
当合约未暂停时，对函数进行调用的修改器。

#### whenPaused()
修饰符#
当合同暂停时才能调用的修饰符。

#### constructor()
内部#
在未暂停状态下初始化合同。将Pauser角色分配给部署者。

#### paused() → bool
公开#
如果合同暂停，则返回true，否则返回false。

#### pause()
公开#
被暂停器调用暂停，触发停止状态。

#### unpause()、
公开#
被暂停者呼叫以解除暂停，恢复正常状态。

#### Paused(address account)
事件#
当暂停是由暂停者（账户）触发时发出。

#### Unpaused(address account)
事件#
当暂停由暂停者（账户）解除时发出。