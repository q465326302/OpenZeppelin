# Access
> TIP
这个页面尚未完成。我们正在努力改进它，以便在下一个版本发布时能够完善。请保持关注！

## Library

### Roles
用于管理分配给角色的地址的图书馆。

**FUNCTIONS**
add(role, account)

remove(role, account)

has(role, account)

#### add(struct Roles.Role role, address account)
内部#
给这个角色授予账户访问权限。

#### remove(struct Roles.Role role, address account)
内部#
移除该角色对账户的访问权限。

#### has(struct Roles.Role role, address account) → bool
内部#
检查账户是否具有此角色。

## Roles

### CapperRole

**MODIFIERS**
onlyCapper()

**FUNCTIONS**
constructor()

isCapper(account)

addCapper(account)

renounceCapper()

_addCapper(account)

_removeCapper(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
CapperAdded(account)

CapperRemoved(account)

#### onlyCapper()
修饰符#

#### constructor()
内部#

#### isCapper(address account) → bool
公开#

#### addCapper(address account)
公开#

#### renounceCapper()
公开#

#### _addCapper(address account)
内部#

#### _removeCapper(address account)
内部#

#### CapperAdded(address account)
事件#

#### CapperRemoved(address account)
事件# 

### MinterRole

**MODIFIERS**
onlyMinter()

**FUNCTIONS**
constructor()

isMinter(account)

addMinter(account)

renounceMinter()

_addMinter(account)

_removeMinter(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
MinterAdded(account)

MinterRemoved(account)

#### onlyMinter()
修饰符#

#### constructor()
内部#

#### isMinter(address account) → bool
公开#

#### addMinter(address account)
公开#

#### renounceMinter()
公开#

#### _addMinter(address account)
内部#

#### _removeMinter(address account)
内部#

#### MinterAdded(address account)
事件#

#### MinterAdded(address account)
事件#

### PauserRole

**MODIFIERS**
onlyPauser()

**FUNCTIONS**
constructor()

isPauser(account)

addPauser(account)

renouncePauser()

_addPauser(account)

_removePauser(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
PauserAdded(account)

PauserRemoved(account)

#### onlyPauser()
修饰符#

#### constructor()
内部#

#### isPauser(address account) → bool
公开#

#### addPauser(address account)
公开#

#### renouncePauser()
公开#

#### _addPauser(address account)
内部#

#### _removePauser(address account)
内部#

#### PauserAdded(address account)
事件#

#### PauserRemoved(address account)
事件#

### SignerRole

**MODIFIERS**
onlySigner()

**FUNCTIONS**
constructor()

isSigner(account)

addSigner(account)

renounceSigner()

_addSigner(account)

_removeSigner(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
SignerAdded(account)

SignerRemoved(account)

#### onlySigner()
修饰符#

#### constructor()
内部#

#### isSigner(address account) → bool
公开#

#### addSigner(address account)
公开#

#### renounceSigner()
公开#

#### _addSigner(address account)
内部#

#### _removeSigner(address account)
内部#

#### SignerAdded(address account)
事件#

#### SignerRemoved(address account)
事件#

### WhitelistAdminRole
WhitelistAdmins负责分配和删除白名单账户。

**MODIFIERS**
onlyWhitelistAdmin()

**FUNCTIONS**
constructor()

isWhitelistAdmin(account)

addWhitelistAdmin(account)

renounceWhitelistAdmin()

_addWhitelistAdmin(account)

_removeWhitelistAdmin(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
WhitelistAdminAdded(account)

WhitelistAdminRemoved(account)

#### onlyWhitelistAdmin()
修饰符#

#### constructor()
内部#

#### isWhitelistAdmin(address account) → bool
公开#

#### addWhitelistAdmin(address account)
公开#

#### renounceWhitelistAdmin()
公开#

#### _addWhitelistAdmin(address account)
内部#

#### _removeWhitelistAdmin(address account)
内部#

#### WhitelistAdminAdded(address account)
事件#

#### WhitelistAdminRemoved(address account)
事件#

### WhitelistedRole
白名单帐户已由白名单管理员批准，可以执行特定操作（例如参与众筹）。这个角色很特殊，只有白名单管理员才能添加它（也可以删除它），而不是白名单本身的帐户。

**MODIFIERS**
onlyWhitelisted()

WHITELISTADMINROLE
onlyWhitelistAdmin()

**FUNCTIONS**
isWhitelisted(account)

addWhitelisted(account)

removeWhitelisted(account)

renounceWhitelisted()

_addWhitelisted(account)

_removeWhitelisted(account)

WHITELISTADMINROLE
constructor()

isWhitelistAdmin(account)

addWhitelistAdmin(account)

renounceWhitelistAdmin()

_addWhitelistAdmin(account)

_removeWhitelistAdmin(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
WhitelistedAdded(account)

WhitelistedRemoved(account)

WHITELISTADMINROLE
WhitelistAdminAdded(account)

WhitelistAdminRemoved(account)

#### onlyWhitelisted()
修饰符#

#### isWhitelisted(address account) → bool
公开#

#### addWhitelisted(address account)
公开#

#### removeWhitelisted(address account)
公开#

#### renounceWhitelisted()
公开#

#### _addWhitelisted(address account)
内部#

#### _removeWhitelisted(address account)
内部#

#### WhitelistedAdded(address account)
事件#

#### WhitelistedRemoved(address account)
事件#