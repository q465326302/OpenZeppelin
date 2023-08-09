# Presets
这些合约将不同的以太坊标准（ERCs）与自定义扩展和模块集成在一起，展示了一些常见的配置，可以直接部署而**无需编写任何Solidity代码**。

它们可以用于快速原型设计和测试，**也适用于生产环境**。

> TIP
中级和高级用户可以将这些合约作为起点，根据需要扩展它们的自定义功能。


## 代币

### ERC20PresetMinterPauser
包括以下功能的*ERC20*代币：

* 持有者能够销毁（销毁）他们的代币

* 允许代币铸造（创建）的铸造者角色

* 允许停止所有代币转账的暂停者角色

该合约使用*AccessControl*来锁定具有不同角色的权限函数-请参阅其文档了解详情。

部署合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色，这将使其能够将铸造者和暂停者角色授予其他账户。

**FUNCTIONS**
constructor(name, symbol)

mint(to, amount)

pause()

unpause()

_beforeTokenTransfer(from, to, amount)

PAUSABLE
paused()

_pause()

_unpause()

ERC20BURNABLE
burn(amount)

burnFrom(account, amount)

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

_transfer(sender, recipient, amount)

_mint(account, amount)

_burn(account, amount)

_approve(owner, spender, amount)

_setupDecimals(decimals_)

ACCESSCONTROL
hasRole(role, account)

getRoleMemberCount(role)

getRoleMember(role, index)

getRoleAdmin(role)

grantRole(role, account)

revokeRole(role, account)

renounceRole(role, account)

_setupRole(role, account)

_setRoleAdmin(role, adminRole)

**EVENTS**
PAUSABLE
Paused(account)

Unpaused(account)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

ACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

#### constructor(string name, string symbol)
公开#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授予部署合约的账户。

请参阅*ERC20.constructor*。

#### mint(address to, uint256 amount)
公开#
创建给定数量的新代币。

参考*ERC20._mint*。

要求：
* 调用者必须具有MINTER_ROLE角色。

#### pause()
公开#
暂停所有代币转移。

参见*ERC20Pausable*和*Pausable._pause*。

要求：
* 调用者必须具有PAUSER_ROLE。

#### unpause()
公开#
取消暂停所有代币转账。

参考 *ERC20Pausable* 和 *Pausable._unpause*。

要求：
* 调用者必须具有 PAUSER_ROLE。

#### _beforeTokenTransfer(address from, address to, uint256 amount)
内部#


### ERC721PresetMinterPauserAutoId
*ERC721*代币，包括：

* 持有者能够销毁（销毁）他们的代币的能力

* 一个铸造者角色，允许代币的铸造（创建）

* 一个暂停者角色，允许停止所有代币的转移

* 代币ID和URI的自动生成

该合约使用*AccessControl*来锁定使用不同角色的权限函数-请查看其文档以获取详细信息。

部署合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色，这将使其能够将铸造者和暂停者角色授予其他账户。

**FUNCTIONS**
constructor(name, symbol, baseURI)

mint(to)

pause()

unpause()

_beforeTokenTransfer(from, to, tokenId)

PAUSABLE
paused()

_pause()

_unpause()

ERC721BURNABLE
burn(tokenId)

ERC721
balanceOf(owner)

ownerOf(tokenId)

name()

symbol()

tokenURI(tokenId)

baseURI()

tokenOfOwnerByIndex(owner, index)

totalSupply()

tokenByIndex(index)

approve(to, tokenId)

getApproved(tokenId)

setApprovalForAll(operator, approved)

isApprovedForAll(owner, operator)

transferFrom(from, to, tokenId)

safeTransferFrom(from, to, tokenId)

safeTransferFrom(from, to, tokenId, _data)

_safeTransfer(from, to, tokenId, _data)

_exists(tokenId)

_isApprovedOrOwner(spender, tokenId)

_safeMint(to, tokenId)

_safeMint(to, tokenId, _data)

_mint(to, tokenId)

_burn(tokenId)

_transfer(from, to, tokenId)

_setTokenURI(tokenId, _tokenURI)

_setBaseURI(baseURI_)

_approve(to, tokenId)

ERC165
supportsInterface(interfaceId)

_registerInterface(interfaceId)

ACCESSCONTROL
hasRole(role, account)

getRoleMemberCount(role)

getRoleMember(role, index)

getRoleAdmin(role)

grantRole(role, account)

revokeRole(role, account)

renounceRole(role, account)

_setupRole(role, account)

_setRoleAdmin(role, adminRole)

**EVENTS**
PAUSABLE
Paused(account)

Unpaused(account)

IERC721
Transfer(from, to, tokenId)

Approval(owner, approved, tokenId)

ApprovalForAll(owner, operator, approved)

ACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

#### constructor(string name, string symbol, string baseURI)
公开#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授予部署合约的账户。

Token的URI将根据baseURI和它们的token ID自动生成。请参阅*ERC721.tokenURI*。

#### mint(address to)
公开#
为to创建一个新的代币。其代币ID将自动分配（并可在发出的*IERC721.Transfer*事件中获得），代币URI将根据在构造函数中传递的基本URI自动生成。

请参阅*ERC721._mint*。

要求：
* 调用者必须具有MINTER_ROLE。

#### pause()
公开#
暂停所有代币转移。

请参阅*ERC721Pausable*和*Pausable._pause*。

要求：
* 调用者必须具有PAUSER_ROLE。

#### _beforeTokenTransfer(address from, address to, uint256 tokenId)
内部#

### ERC1155PresetMinterPauser

*ERC1155*代币，包括以下功能：

* 持有者可以烧毁（销毁）他们的代币

* 有一个铸造者角色，可以创建代币

* 有一个暂停者角色，可以停止所有代币转移

这个合约使用*AccessControl*来锁定使用不同角色的权限功能-请查看其文档获取详细信息。

部署合约的账户将被授予铸造者和暂停者角色，以及默认的管理员角色，这将使其能够授予其他账户铸造者和暂停者角色的权限。

**FUNCTIONS**
constructor(uri)

mint(to, id, amount, data)

mintBatch(to, ids, amounts, data)

pause()

unpause()

_beforeTokenTransfer(operator, from, to, ids, amounts, data)

PAUSABLE
paused()

_pause()

_unpause()

ERC1155BURNABLE
burn(account, id, value)

burnBatch(account, ids, values)

ERC1155
uri(_)

balanceOf(account, id)

balanceOfBatch(accounts, ids)

setApprovalForAll(operator, approved)

isApprovedForAll(account, operator)

safeTransferFrom(from, to, id, amount, data)

safeBatchTransferFrom(from, to, ids, amounts, data)

_setURI(newuri)

_mint(account, id, amount, data)

_mintBatch(to, ids, amounts, data)

_burn(account, id, amount)

_burnBatch(account, ids, amounts)

ERC165
supportsInterface(interfaceId)

_registerInterface(interfaceId)

ACCESSCONTROL
hasRole(role, account)

getRoleMemberCount(role)

getRoleMember(role, index)

getRoleAdmin(role)

grantRole(role, account)

revokeRole(role, account)

renounceRole(role, account)

_setupRole(role, account)

_setRoleAdmin(role, adminRole)

**EVENTS**
PAUSABLE
Paused(account)

Unpaused(account)

IERC1155
TransferSingle(operator, from, to, id, value)

TransferBatch(operator, from, to, ids, values)

ApprovalForAll(account, operator, approved)

URI(value, id)

ACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

#### constructor(string uri)
公开#
将DEFAULT_ADMIN_ROLE、MINTER_ROLE和PAUSER_ROLE授予部署合约的账户。

#### mint(address to, uint256 id, uint256 amount, bytes data)
公开#
为特定的代币类型创建指定数量的新代币。

请参考*ERC1155._mint*。

要求：
* 调用者必须具有MINTER_ROLE角色。

#### mintBatch(address to, uint256[] ids, uint256[] amounts, bytes data)
公开#

批量版本的mint。

#### pause()
公开#
暂停所有代币转账。

请参考*ERC1155Pausable*和*Pausable._pause*。

要求：
* 调用者必须具有PAUSER_ROLE角色。

#### unpause()
公开#
取消暂停所有代币转账。

参考 *ERC1155Pausable* 和 *Pausable._unpause*。

要求：
* 调用者必须具有 PAUSER_ROLE。

#### _beforeTokenTransfer(address operator, address from, address to, uint256[] ids, uint256[] amounts, bytes data)
内部#

### ERC20PresetFixedSupply
*ERC20*代币，包括：

* 预挖的初始供应量

* 持有者可以销毁（销毁）他们的代币

* 没有访问控制机制（用于铸币/暂停），因此没有治理

该合约使用*ERC20Burnable*来包含销毁功能-请参阅其文档以了解详情。

*自v3.4起可用。*

**FUNCTIONS**
constructor(name, symbol, initialSupply, owner)

ERC20BURNABLE
burn(amount)

burnFrom(account, amount)

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

_transfer(sender, recipient, amount)

_mint(account, amount)

_burn(account, amount)

_approve(owner, spender, amount)

_setupDecimals(decimals_)

_beforeTokenTransfer(from, to, amount)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### constructor(string name, string symbol, address[] defaultOperators, uint256 initialSupply, address owner)
公开#
Mint函数初始化token的初始供应量，并将其转移到所有者账户中。

参见*ERC777.constructor*函数。