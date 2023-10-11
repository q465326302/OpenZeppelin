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
internal#
给这个角色授予账户访问权限。

#### remove(struct Roles.Role role, address account)
internal#
移除该角色对账户的访问权限。

#### has(struct Roles.Role role, address account) → bool
internal#
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
modifier#

#### constructor()
internal#

#### isCapper(address account) → bool
public#

#### addCapper(address account)
public#

#### renounceCapper()
public#

#### _addCapper(address account)
internal#

#### _removeCapper(address account)
internal#

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

[MinterRemoved(account)](#minterremovedaddress-account)

#### onlyMinter()
modifier#

#### constructor()
internal#

#### isMinter(address account) → bool
public#

#### addMinter(address account)
public#

#### renounceMinter()
public#

#### _addMinter(address account)
internal#

#### _removeMinter(address account)
internal#

#### MinterAdded(address account)
事件#

#### MinterRemoved(address account)
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
modifier#

#### constructor()
internal#

#### isPauser(address account) → bool
public#

#### addPauser(address account)
public#

#### renouncePauser()
public#

#### _addPauser(address account)
internal#

#### _removePauser(address account)
internal#

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
modifier#

#### constructor()
internal#

#### isSigner(address account) → bool
public#

#### addSigner(address account)
public#

#### renounceSigner()
public#

#### _addSigner(address account)
internal#

#### _removeSigner(address account)
internal#

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
modifier#

#### constructor()
internal#

#### isWhitelistAdmin(address account) → bool
public#

#### addWhitelistAdmin(address account)
public#

#### renounceWhitelistAdmin()
public#

#### _addWhitelistAdmin(address account)
internal#

#### _removeWhitelistAdmin(address account)
internal#

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
modifier#

#### isWhitelisted(address account) → bool
public#

#### addWhitelisted(address account)
public#

#### removeWhitelisted(address account)
public#

#### renounceWhitelisted()
public#

#### _addWhitelisted(address account)
internal#

#### _removeWhitelisted(address account)
internal#

#### WhitelistedAdded(address account)
事件#

#### WhitelistedRemoved(address account)
事件#