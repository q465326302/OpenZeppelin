# Presets
这些合约将不同的以太坊标准（ERCs）与自定义扩展和模块集成在一起，展示了一些常见的配置，可以直接部署而**无需编写任何Solidity代码**。

它们可以用于快速原型设计和测试，**也适用于生产环境**。

> TIP
中级和高级用户可以将这些合约作为起点，根据需要扩展它们的自定义功能。


## Tokens

### ERC20PresetMinterPauser
包括以下功能的[ERC20](./ERC20.md#erc20)代币：

* 持有者能够销毁（销毁）他们的代币

* 允许代币铸造（创建）的铸造者角色

* 允许停止所有代币转账的暂停者角色

该合约使用[AccessControl](./Access.md#accesscontrol)来锁定具有不同角色的权限函数-请参阅其文档了解详情。

部署合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色，这将使其能够将铸造者和暂停者角色授予其他账户。

**FUNCTIONS**
[constructor(name, symbol)](#constructorstring-name-string-symbol)

[mint(to, amount)](#mintaddress-to-uint256-amount)

[pause()](#pause)

[unpause()](#unpause)

[_beforeTokenTransfer(from, to, amount)](#_beforetokentransferaddress-from-address-to-uint256-amount)

PAUSABLE
[paused()](./Utils.md#paused-→-bool)

[_pause()](./Utils.md#_pause)

[_unpause()](./Utils.md#_unpause)

ERC20BURNABLE
[burn(amount)](./ERC20.md#burnuint256-amount)

[burnFrom(account, amount)](./ERC20.md#burnfromaddress-account-uint256-amount)

ERC20
[name()](./ERC20.md#name-→-string)

[symbol()](./ERC20.md#symbol-→-string)

[decimals()](./ERC20.md#decimals-→-uint8)

[totalSupply()](./ERC20.md#totalsupply-e28692-uint256-1)

[balanceOf(account)](./ERC20.md#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](./ERC20.md#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](./ERC20.md#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](./ERC20.md#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](./ERC20.md#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](./ERC20.md#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](./ERC20.md#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](./ERC20.md#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](./ERC20.md#_mintaddress-account-uint256-amount)

[_burn(account, amount)](./ERC20.md#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](./ERC20.md#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](./ERC20.md#_setupdecimalsuint8-decimals)

ACCESSCONTROL
[hasRole(role, account)](./Access.md#hasrolebytes32-role-address-account-→-bool)

[getRoleMemberCount(role)](./Access.md#getrolemembercountbytes32-role-→-uint256)

[getRoleMember(role, index)](./Access.md#getrolememberbytes32-role-uint256-index-→-address)

[getRoleAdmin(role)](./Access.md#getroleadminbytes32-role-→-bytes32)

[grantRole(role, account)](./Access.md#grantrolebytes32-role-address-account)

[revokeRole(role, account)](./Access.md#revokerolebytes32-role-address-account)

[renounceRole(role, account)](./Access.md#renouncerolebytes32-role-address-account)

[_setupRole(role, account)](./Access.md#_setuprolebytes32-role-address-account)

[_setRoleAdmin(role, adminRole)](./Access.md#_setroleadminbytes32-role-bytes32-adminrole)

**EVENTS**
PAUSABLE
[Paused(account)](./Utils.md#pausedaddress-account)

[Unpaused(account)](./Utils.md#unpausedaddress-account)

IERC20
[Transfer(from, to, value)](./ERC20.md#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](./ERC20.md#approvaladdress-owner-address-spender-uint256-value)

ACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](./Access.md#roleadminchangedbytes32-role-bytes32-previousadminrole-bytes32-newadminrole)

[RoleGranted(role, account, sender)](./Access.md#rolegrantedbytes32-role-address-account-address-sender)

[RoleRevoked(role, account, sender)](./Access.md#rolerevokedbytes32-role-address-account-address-sender)

#### constructor(string name, string symbol)
public#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授予部署合约的账户。

请参阅 [ERC20.constructor](./ERC20.md#constructorstring-name_-string-symbol_)。

#### mint(address to, uint256 amount)
public#
创建给定数量的新代币。

参考 [ERC20._mint](./ERC20.md#_mintaddress-account-uint256-amount)。

要求：
* 调用者必须具有MINTER_ROLE角色。

#### pause()
public#
暂停所有代币转移。

参见 [ERC20Pausable](./ERC20.md#erc20pausable)和[Pausable._pause](./Utils.md#_pause)。

要求：
* 调用者必须具有PAUSER_ROLE。

#### unpause()
public#
取消暂停所有代币转账。

参考 [ERC20Pausable](./ERC20.md#erc20pausable) 和 [Pausable._unpause](./Utils.md#_unpause)。

要求：
* 调用者必须具有 PAUSER_ROLE。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
internal#


### ERC721PresetMinterPauserAutoId
[ERC721](./ERC721.md#erc721)代币，包括：

* 持有者能够销毁（销毁）他们的代币的能力

* 一个铸造者角色，允许代币的铸造（创建）

* 一个暂停者角色，允许停止所有代币的转移

* 代币ID和URI的自动生成

该合约使用[AccessControl](./Access.md#accesscontrol)来锁定使用不同角色的权限函数-请查看其文档以获取详细信息。

部署合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色，这将使其能够将铸造者和暂停者角色授予其他账户。

**FUNCTIONS**
[constructor(name, symbol, baseURI)](#constructorstring-name-string-symbol-string-baseuri)

[mint(to)](#mintaddress-to)

[pause()](#pause-1)

[unpause()](#unpause-1)

[_beforeTokenTransfer(from, to, tokenId)](#_beforetokentransferaddress-from-address-to-uint256-tokenid)

PAUSABLE
[paused()](./Utils.md#paused-→-bool)

[_pause()](./Utils.md#_pause)

[_unpause()](./Utils.md#_unpause)

ERC721BURNABLE
[burn(tokenId)](./ERC721.md#burnuint256-tokenid)

ERC721
[balanceOf(owner)](./ERC721.md#balanceofaddress-owner-→-uint256)

[ownerOf(tokenId)](./ERC721.md#ownerofuint256-tokenid-→-address)

[name()](./ERC721.md#name-e28692-string-1)

[symbol()](./ERC721.md#symbol-e28692-string-1)

[tokenURI(tokenId)](./ERC721.md#totalsupply-e28692-uint256-1)

[baseURI()](./ERC721.md#baseuri-→-string)

[tokenOfOwnerByIndex(owner, index)](./ERC721.md#tokenofownerbyindexaddress-owner-uint256-index-→-uint256-tokenid)

[totalSupply()](./ERC721.md#totalsupply-e28692-uint256-1)

[tokenByIndex(index)](./ERC721.md#tokenbyindexuint256-index-e28692-uint256-1)

[approve(to, tokenId)](./ERC721.md#approveaddress-to-uint256-tokenid-1)

[getApproved(tokenId)](./ERC721.md#getapproveduint256-tokenid-→-address)

[setApprovalForAll(operator, approved)](./ERC721.md#setapprovalforalladdress-operator-bool-approved)

[isApprovedForAll(owner, operator)](./ERC721.md#isapprovedforalladdress-owner-address-operator-e28692-bool-1)

[transferFrom(from, to, tokenId)](./ERC721.md#transferfromaddress-from-address-to-uint256-tokenid)

[safeTransferFrom(from, to, tokenId)](./ERC721.md#safetransferfromaddress-from-address-to-uint256-tokenid-1)

[safeTransferFrom(from, to, tokenId, _data)](./ERC721.md#safetransferfromaddress-from-address-to-uint256-tokenid-bytes-_data)

[_safeTransfer(from, to, tokenId, _data)](./ERC721.md#_safetransferaddress-from-address-to-uint256-tokenid-bytes-_data)

[_exists(tokenId)](./ERC721.md#_existsuint256-tokenid-→-bool)

[_isApprovedOrOwner(spender, tokenId)](./ERC721.md#_isapprovedorowneraddress-spender-uint256-tokenid-→-bool)

[_safeMint(to, tokenId)](./ERC721.md#_safemintaddress-to-uint256-tokenid)

[_safeMint(to, tokenId, _data)](./ERC721.md#_safemintaddress-to-uint256-tokenid-bytes-_data)

[_mint(to, tokenId)](./ERC721.md#_mintaddress-to-uint256-tokenid)

[_burn(tokenId)](./ERC721.md#_burnuint256-tokenid)

[_transfer(from, to, tokenId)](./ERC721.md#_transferaddress-from-address-to-uint256-tokenid)

[_setTokenURI(tokenId, _tokenURI)](./ERC721.md#_settokenuriuint256-tokenid-string-_tokenuri)

[_setBaseURI(baseURI_)](./ERC721.md#_setbaseuristring-baseuri)

[_approve(to, tokenId)](./ERC721.md#_approveaddress-to-uint256-tokenid)

ERC165
[supportsInterface(interfaceId)](./Introspection.md#supportsinterfacebytes4-interfaceid-e28692-bool-1)

[_registerInterface(interfaceId)](./Introspection.md#_registerinterfacebytes4-interfaceid)

ACCESSCONTROL
[hasRole(role, account)](./Access.md#hasrolebytes32-role-address-account-→-bool)

[getRoleMemberCount(role)](./Access.md#getrolemembercountbytes32-role-→-uint256)

[getRoleMember(role, index)](./Access.md#getrolememberbytes32-role-uint256-index-→-address)

[getRoleAdmin(role)](./Access.md#getroleadminbytes32-role-→-bytes32)

[grantRole(role, account)](./Access.md#grantrolebytes32-role-address-account)

[revokeRole(role, account)](./Access.md#revokerolebytes32-role-address-account)

[renounceRole(role, account)](./Access.md#renouncerolebytes32-role-address-account)

[_setupRole(role, account)](./Access.md#_setuprolebytes32-role-address-account)

[_setRoleAdmin(role, adminRole)](./Access.md#_setroleadminbytes32-role-bytes32-adminrole)

**EVENTS**
PAUSABLE
[Paused(account)](./Utils.md#pausedaddress-account)

[Unpaused(account)](./Utils.md#unpausedaddress-account)

IERC721
[Transfer(from, to, tokenId)](./ERC721.md#transferaddress-from-address-to-uint256-tokenid)

[Approval(owner, approved, tokenId)](./ERC721.md#approvaladdress-owner-address-approved-uint256-tokenid)

[ApprovalForAll(owner, operator, approved)](./ERC721.md#approvalforalladdress-owner-address-operator-bool-approved)

ACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](./Access.md#roleadminchangedbytes32-role-bytes32-previousadminrole-bytes32-newadminrole)

[RoleGranted(role, account, sender)](./Access.md#rolegrantedbytes32-role-address-account-address-sender)

[RoleRevoked(role, account, sender)](./Access.md#rolegrantedbytes32-role-address-account-address-sender)

#### constructor(string name, string symbol, string baseURI)
public#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授予部署合约的账户。

Token的URI将根据baseURI和它们的token ID自动生成。请参阅[ERC721.tokenURI](./ERC721.md#tokenuriuint256-tokenid-e28692-string-1)。

#### mint(address to)
public#
为to创建一个新的代币。其代币ID将自动分配（并可在发出的[IERC721.Transfer](./ERC721.md#transferaddress-from-address-to-uint256-tokenid)事件中获得），代币URI将根据在构造函数中传递的基本URI自动生成。

请参阅[ERC721._mint](./ERC721.md#_mintaddress-to-uint256-tokenid)。

要求：
* 调用者必须具有MINTER_ROLE。

#### pause()
public#
暂停所有代币转移。

请参阅[ERC721Pausable](./ERC721.md#erc721pausable)和[Pausable._pause](./Utils.md#_pause)。

要求：
* 调用者必须具有PAUSER_ROLE。

#### unpause()
public#
解除所有代币转移的暂停。

参见[ERC721Pausable](./ERC721.md#erc721pausable)和[Pausable._unpause](./Utils.md#_unpause)。

要求：

调用者必须具有PAUSER_ROLE。

#### _beforeTokenTransfer(address from, address to, uint256 tokenId)
internal#

### ERC1155PresetMinterPauser

[ERC1155](./ERC1155.md)代币，包括以下功能：

* 持有者可以烧毁（销毁）他们的代币

* 有一个铸造者角色，可以创建代币

* 有一个暂停者角色，可以停止所有代币转移

这个合约使用[AccessControl](./Access.md#accesscontrol)来锁定使用不同角色的权限功能-请查看其文档获取详细信息。

部署合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色，这将使其能够授予其他账户铸造者和暂停者角色的权限。

**FUNCTIONS**
[constructor(uri)](#constructorstring-uri)

[mint(to, id, amount, data)](#mintaddress-to-uint256-id-uint256-amount-bytes-data)

[mintBatch(to, ids, amounts, data)](#mintbatchaddress-to-uint256-ids-uint256-amounts-bytes-data)

[pause()](#pause-2)

[unpause()](#unpause-2)

[_beforeTokenTransfer(operator, from, to, ids, amounts, data)](#_beforetokentransferaddress-operator-address-from-address-to-uint256-ids-uint256-amounts-bytes-data)

PAUSABLE
[paused()](./Utils.md#paused-→-bool)

[_pause()](./Utils.md#_pause)

[_unpause()](./Utils.md#_unpause)

ERC1155BURNABLE
[burn(account, id, value)](./ERC1155.md#burnaddress-account-uint256-id-uint256-value)

[burnBatch(account, ids, values)](./ERC1155.md#burnbatchaddress-account-uint256-ids-uint256-values)

ERC1155
[uri(_)](./ERC1155.md#uriuint256-→-string)

[balanceOf(account, id)](./ERC1155.md#balanceofaddress-account-uint256-id-e28692-uint256-1)

[balanceOfBatch(accounts, ids)](./ERC1155.md#balanceofbatchaddress-accounts-uint256-ids-e28692-uint256-1)

[setApprovalForAll(operator, approved)](./ERC1155.md#setapprovalforalladdress-operator-bool-approved-1)

[isApprovedForAll(account, operator)](./ERC1155.md#isapprovedforalladdress-account-address-operator-e28692-bool-1)

[safeTransferFrom(from, to, id, amount, data)](./ERC1155.md#safetransferfromaddress-from-address-to-uint256-id-uint256-amount-bytes-data-1)

[safeBatchTransferFrom(from, to, ids, amounts, data)](./ERC1155.md#safebatchtransferfromaddress-from-address-to-uint256-ids-uint256-amounts-bytes-data-1)

[_setURI(newuri)](./ERC1155.md#_seturistring-newuri)

[_mint(account, id, amount, data)](./ERC1155.md#_mintaddress-account-uint256-id-uint256-amount-bytes-data)

[_mintBatch(to, ids, amounts, data)](./ERC1155.md#_mintbatchaddress-to-uint256-ids-uint256-amounts-bytes-data)

[_burn(account, id, amount)](./ERC1155.md#_burnaddress-account-uint256-id-uint256-amount)

[_burnBatch(account, ids, amounts)](./ERC1155.md#_burnbatchaddress-account-uint256-ids-uint256-amounts)

ERC165
[supportsInterface(interfaceId)](./Introspection.md#supportsinterfacebytes4-interfaceid-e28692-bool-1)

[_registerInterface(interfaceId)](./Introspection.md#_registerinterfacebytes4-interfaceid)

ACCESSCONTROL
[hasRole(role, account)](./Access.md#hasrolebytes32-role-address-account-→-bool)

[getRoleMemberCount(role)](./Access.md#getrolemembercountbytes32-role-→-uint256)

[getRoleMember(role, index)](./Access.md#getrolememberbytes32-role-uint256-index-→-address)

[getRoleAdmin(role)](./Access.md#getroleadminbytes32-role-→-bytes32)

[grantRole(role, account)](./Access.md#grantrolebytes32-role-address-account)

[revokeRole(role, account)](./Access.md#revokerolebytes32-role-address-account)

[renounceRole(role, account)](./Access.md#renouncerolebytes32-role-address-account)

[_setupRole(role, account)](./Access.md#_setuprolebytes32-role-address-account)

[_setRoleAdmin(role, adminRole)](./Access.md#_setroleadminbytes32-role-bytes32-adminrole)

**EVENTS**
PAUSABLE
[Paused(account)](./Utils.md#pausedaddress-account)

[Unpaused(account)](./Utils.md#unpausedaddress-account)

IERC1155
[TransferSingle(operator, from, to, id, value)](./ERC1155.md#transfersingleaddress-operator-address-from-address-to-uint256-id-uint256-value)

[TransferBatch(operator, from, to, ids, values)](./ERC1155.md#transferbatchaddress-operator-address-from-address-to-uint256-ids-uint256-values)

[ApprovalForAll(account, operator, approved)](./ERC1155.md#approvalforalladdress-account-address-operator-bool-approved)

[URI(value, id)](./ERC1155.md#uristring-value-uint256-id)

ACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](./Access.md#roleadminchangedbytes32-role-bytes32-previousadminrole-bytes32-newadminrole)

[RoleGranted(role, account, sender)](./Access.md#rolegrantedbytes32-role-address-account-address-sender)

[RoleRevoked(role, account, sender)](./Access.md#rolegrantedbytes32-role-address-account-address-sender)

#### constructor(string uri)
public#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授予部署合约的账户。

#### mint(address to, uint256 id, uint256 amount, bytes data)
public#
为特定的代币类型创建指定数量的新代币。

请参考[ERC1155._mint](./ERC1155.md#_mintaddress-account-uint256-id-uint256-amount-bytes-data)。

要求：
* 调用者必须具有MINTER_ROLE角色。

#### mintBatch(address to, uint256[] ids, uint256[] amounts, bytes data)
public#

批量版本的mint。

#### pause()
public#
暂停所有代币转账。

请参考 [ERC1155Pausable](./ERC1155.md#erc1155pausable)和[Pausable._pause](./Utils.md#_pause)。

要求：
* 调用者必须具有PAUSER_ROLE角色。

#### unpause()
public#
取消暂停所有代币转账。

参考 [ERC1155Pausable](./ERC1155.md#erc1155pausable) 和 [Pausable._unpause](./Utils.md#_unpause)。

要求：
* 调用者必须具有 PAUSER_ROLE。

#### _beforeTokenTransfer(address operator, address from, address to, uint256[] ids, uint256[] amounts, bytes data)
internal#

### ERC20PresetFixedSupply
[ERC20](./ERC20.md)代币，包括：

* 预挖的初始供应量

* 持有者可以销毁（销毁）他们的代币

* 没有访问控制机制（用于铸币/暂停），因此没有治理

该合约使用[ERC20Burnable](./ERC20.md#erc20burnable)来包含销毁功能-请参阅其文档以了解详情。

*自v3.4起可用。*

**FUNCTIONS**
[constructor(name, symbol, initialSupply, owner)]()

ERC20BURNABLE
[burn(amount)](./ERC20.md#burnuint256-amount)

[burnFrom(account, amount)](./ERC20.md#burnfromaddress-account-uint256-amount)

ERC20
[name()](./ERC20.md#name-→-string)

[symbol()](./ERC20.md#symbol-→-string)

[decimals()](./ERC20.md#decimals-→-uint8)

[totalSupply()](./ERC20.md#totalsupply-e28692-uint256-1)

[balanceOf(account)](./ERC20.md#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](./ERC20.md#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](./ERC20.md#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](./ERC20.md#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](./ERC20.md#transferfromaddress-sender-address-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](./ERC20.md#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](./ERC20.md#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](./ERC20.md#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](./ERC20.md#_mintaddress-account-uint256-amount)

[_burn(account, amount)](./ERC20.md#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](./ERC20.md#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](./ERC20.md#_setupdecimalsuint8-decimals)

[_beforeTokenTransfer(from, to, amount)](./ERC20.md#_beforetokentransferaddress-from-address-to-uint256-amount-2)

EVENTS
IERC20
[Transfer(from, to, value)](./ERC20.md#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](./ERC20.md#approvaladdress-owner-address-spender-uint256-value)

#### constructor(string name, string symbol, uint256 initialSupply, address owner)
public#
初始化供应量的代币并将它们转移到所有者。

参见[ERC20.constructor](./ERC20.md#constructorstring-name_-string-symbol_)。

### ERC777PresetFixedSupply

[ERC777](./ERC777.md#erc777)代币，包括：

预先铸造的初始供应量

没有访问控制机制（用于铸币/暂停），因此没有治理

自v3.4版本以来可用。

FUNCTIONS
[constructor(name, symbol, defaultOperators, initialSupply, owner)](#constructorstring-name-string-symbol-uint256-initialsupply-address-owner)

ERC777
[name()](./ERC777.md#name-e28692-string-1)

[symbol()](./ERC777.md#symbol-e28692-string-1)

[decimals()](./ERC777.md#decimals-→-uint8)

[granularity()](./ERC777.md#granularity-e28692-uint256-1)

[totalSupply()](./ERC777.md#totalsupply-e28692-uint256-1)

[balanceOf(tokenHolder)](./ERC777.md#balanceofaddress-tokenholder-→-uint256)

[send(recipient, amount, data)](./ERC777.md#sendaddress-recipient-uint256-amount-bytes-data)

[transfer(recipient, amount)](./ERC777.md#transferaddress-recipient-uint256-amount-→-bool)

[burn(amount, data)](./ERC777.md#burnuint256-amount-bytes-data-1)

[isOperatorFor(operator, tokenHolder)](./ERC777.md#isoperatorforaddress-operator-address-tokenholder-e28692-bool-1)

[authorizeOperator(operator)](./ERC777.md#authorizeoperatoraddress-operator-1)

[revokeOperator(operator)](./ERC777.md#revokeoperatoraddress-operator-1)

[defaultOperators()](./ERC777.md#defaultoperators-e28692-address-1)

[operatorSend(sender, recipient, amount, data, operatorData)](./ERC777.md#operatorsendaddress-sender-address-recipient-uint256-amount-bytes-data-bytes-operatordata-1)

[operatorBurn(account, amount, data, operatorData)](./ERC777.md#operatorburnaddress-account-uint256-amount-bytes-data-bytes-operatordata-1)

[allowance(holder, spender)](./ERC777.md#allowanceaddress-holder-address-spender-→-uint256)

[approve(spender, value)](./ERC777.md#approveaddress-spender-uint256-value-→-bool)

[transferFrom(holder, recipient, amount)](./ERC777.md#transferfromaddress-holder-address-recipient-uint256-amount-→-bool)

[_mint(account, amount, userData, operatorData)](./ERC777.md#_mintaddress-account-uint256-amount-bytes-userdata-bytes-operatordata)

[_send(from, to, amount, userData, operatorData, requireReceptionAck)](./ERC777.md#_sendaddress-from-address-to-uint256-amount-bytes-userdata-bytes-operatordata-bool-requirereceptionack)

[_burn(from, amount, data, operatorData)](./ERC777.md#_burnaddress-from-uint256-amount-bytes-data-bytes-operatordata)

[_approve(holder, spender, value)](./ERC777.md#_approveaddress-holder-address-spender-uint256-value)

[_beforeTokenTransfer(operator, from, to, amount)](./ERC777.md#_beforetokentransferaddress-operator-address-from-address-to-uint256-amount)

EVENTS
IERC20
[Transfer(from, to, value)](./ERC20.md#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](./ERC20.md#approvaladdress-owner-address-spender-uint256-value)

IERC777
[Sent(operator, from, to, amount, data, operatorData)](./ERC777.md#sentaddress-operator-address-from-address-to-uint256-amount-bytes-data-bytes-operatordata)

[Minted(operator, to, amount, data, operatorData)](./ERC777.md#mintedaddress-operator-address-to-uint256-amount-bytes-data-bytes-operatordata)

[Burned(operator, from, amount, data, operatorData)](./ERC777.md#burnedaddress-operator-address-from-uint256-amount-bytes-data-bytes-operatordata)

[AuthorizedOperator(operator, tokenHolder)](./ERC777.md#authorizedoperatoraddress-operator-address-tokenholder)

[RevokedOperator(operator, tokenHolder)](./ERC777.md#revokedoperatoraddress-operator-address-tokenholder)

#### constructor(string name, string symbol, address[] defaultOperators, uint256 initialSupply, address owner)
public#
初始化供应一定数量的代币并将它们转移到所有者。

参见 [ERC777.constructor](./ERC777.md#constructorstring-name_-string-symbol_-address-defaultoperators_)。