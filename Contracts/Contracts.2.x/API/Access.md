# Access
> TIP
这个页面尚未完成。我们正在努力改进它，以便在下一个版本发布时能够完善。请保持关注！

## Library

### Roles
用于管理分配给角色的地址的库。

**FUNCTIONS**
[add(role, account)](#addstruct-rolesrole-role-address-account)

[remove(role, account)](#removestruct-rolesrole-role-address-account)

[has(role, account)](#hasstruct-rolesrole-role-address-account-→-bool)

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
[onlyCapper()](#onlycapper)

**FUNCTIONS**
[constructor()](#constructor)

[isCapper(account)](#iscapperaddress-account-→-bool)

[addCapper(account)](#addcapperaddress-account)

[renounceCapper()](#renouncecapper)

[_addCapper(account)](#_addcapperaddress-account)

[_removeCapper(account)](#_removecapperaddress-account)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[CapperAdded(account)](#capperaddedaddress-account)

[CapperRemoved(account)](#capperremovedaddress-account)

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
[onlyMinter()](#onlyminter)

**FUNCTIONS**
[constructor()](#constructor-1)

[isMinter(account)](#isminteraddress-account-→-bool)

[addMinter(account)](#addminteraddress-account)

[renounceMinter()](#renounceminter)

[_addMinter(account)](#_addminteraddress-account)

[_removeMinter(account)](#_removeminteraddress-account)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[MinterAdded(account)](#minteraddedaddress-account)

[MinterRemoved(account)](#minteraddedaddress-account)

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
[onlyPauser()](#onlypauser)

**FUNCTIONS**
[constructor()](#constructor-2)

[isPauser(account)](#ispauseraddress-account-→-bool)

[addPauser(account)](#addpauseraddress-account)

[renouncePauser()](#renouncepauser)

[_addPauser(account)](#_addpauseraddress-account)

[_removePauser(account)](#_removepauseraddress-account)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[PauserAdded(account)](#pauseraddedaddress-account)

[PauserRemoved(account)](#pauserremovedaddress-account)

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
[onlySigner()](#onlysigner)

**FUNCTIONS**
[constructor()](#constructor-3)

[isSigner(account)](#issigneraddress-account-→-bool)

[addSigner(account)](#addsigneraddress-account)

[renounceSigner()](#renouncesigner)

[_addSigner(account)](#_addsigneraddress-account)

[_removeSigner(account)](#_removesigneraddress-account)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[SignerAdded(account)](#signeraddedaddress-account)

[SignerRemoved(account)](#signerremovedaddress-account)

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
[onlyWhitelistAdmin()](#onlywhitelistadmin)

**FUNCTIONS**
[constructor()](#constructor-4)

[isWhitelistAdmin(account)](#iswhitelistadminaddress-account-→-bool)

[addWhitelistAdmin(account)](#addwhitelistadminaddress-account)

[renounceWhitelistAdmin()](#renouncewhitelistadmin)

[_addWhitelistAdmin(account)](#_addwhitelistadminaddress-account)

[_removeWhitelistAdmin(account)](#_removewhitelistadminaddress-account)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[WhitelistAdminAdded(account)](#whitelistadminaddedaddress-account)

[WhitelistAdminRemoved(account)](#whitelistadminremovedaddress-account)

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
[onlyWhitelisted()](#onlywhitelisted)

WHITELISTADMINROLE
[onlyWhitelistAdmin()](#onlywhitelistadmin)

**FUNCTIONS**
[isWhitelisted(account)](#iswhitelistedaddress-account-→-bool)

[addWhitelisted(account)](#addwhitelistedaddress-account)

[removeWhitelisted(account)](#removewhitelistedaddress-account)

[renounceWhitelisted()](#renouncewhitelisted)

[_addWhitelisted(account)](#_addwhitelistedaddress-account)

[_removeWhitelisted(account)](#_removewhitelistedaddress-account)

WHITELISTADMINROLE
[constructor()](#constructor-4)

[isWhitelistAdmin(account)](#iswhitelistadminaddress-account-→-bool)

[addWhitelistAdmin(account)](#addwhitelistadminaddress-account)

[renounceWhitelistAdmin()](#renouncewhitelistadmin)

[_addWhitelistAdmin(account)](#_addwhitelistadminaddress-account)

[_removeWhitelistAdmin(account)](#_removewhitelistadminaddress-account)

CONTEXT
[_msgSender()](./GSN.md#_msgsender-→-address-payable)

[_msgData()](./GSN.md#_msgdata-→-bytes)

**EVENTS**
[WhitelistedAdded(account)](#whitelistadminaddedaddress-account)

[WhitelistedRemoved(account)](#whitelistadminremovedaddress-account)

WHITELISTADMINROLE
[WhitelistAdminAdded(account)](#whitelistadminaddedaddress-account)

[WhitelistAdminRemoved(account)](#whitelistadminremovedaddress-account)

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