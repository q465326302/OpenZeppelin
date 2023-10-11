# Access Control
这个目录提供了限制谁可以访问合约功能或何时访问的方法。

* [AccessControl](../Access-Control.md) 提供了一个基于角色的访问控制机制。可以创建多个层次的角色，并将每个角色分配给多个帐户。

* [Ownable](#ownable) 是一个更简单的机制，具有单个所有者“角色”，可以分配给单个帐户。这种较简单的机制对于快速测试很有用，但具有生产方面的问题的项目可能会超出它的范围。

## 授权

### [Ownable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/Ownable.sol)

import "@openzeppelin/contracts/access/Ownable.sol";
合约模块提供了一个基本的访问控制机制，其中有一个帐户（所有者）可以被授予对特定功能的独占访问权。

初始所有者被设置为部署者提供的地址。这可以在后期通过*transferOwnership*进行更改。

这个模块通过继承使用。它将提供修饰符 onlyOwner，可以应用于你的函数，以限制它们的使用仅限于所有者。

**MODIFIERS**
[onlyOwner()](#onlyowner)

**FUNCTIONS**
[constructor()](#constructor)
[owner()](#owner-→-address)
[_checkOwner()](#_checkowner)
[renounceOwnership()](#renounceownership)
[transferOwnership(newOwner)](#transferownershipaddress-newowner)
[_transferOwnership(newOwner)](#_transferownershipaddress-newowner)

**EVENTS**
[OwnershipTransferred(previousOwner, newOwner)](#ownershiptransferredaddress-indexed-previousowner-address-indexed-newowner)

#### onlyOwner()
modifier#
如果被除了所有者以外的任何账户调用，就会抛出异常。

#### constructor()
internal#
初始化合约，将部署者设置为初始所有者。

#### owner() → address
公共函数
返回当前所有者的地址。

#### _checkOwner()
内部函数#
如果发送方不是所有者，则抛出异常。

#### renounceOwnership()
函数公开可用#
该函数使合约失去所有者身份。此后将无法调用仅限所有者使用的函数。只能由当前所有者调用。
> NOTE
放弃所有权将使合约没有所有者，从而禁用仅对所有者可用的任何功能。

#### transferOwnership(address newOwner)
public#
将合约的所有权转移给一个新的账户（newOwner）。只有当前所有者才能调用。

#### _transferOwnership(address newOwner)
internal#
将合约的所有权转移给一个新账户（newOwner）。内部函数，没有访问限制。

#### OwnershipTransferred(address indexed previousOwner, address indexed newOwner)
事件#

### Ownable2Step
```
import "@openzeppelin/contracts/access/Ownable2Step.sol";
```
该合约模块提供了访问控制机制，其中有一个账户（所有者）可以被授予对特定函数的独占访问权限。

默认情况下，所有者账户将是部署合约的账户。这可以通过[transferOwnership](#transferownershipaddress-newowner-1)和[acceptOwnership](#acceptownership)进行更改。

该模块通过继承使用。它将使父级（Ownable）的所有函数可用。
**FUNCTIONS**
[pendingOwner()](#pendingowner-→-address)
[transferOwnership(newOwner)](#transferownershipaddress-newowner-1)
[_transferOwnership(newOwner)](#_transferownershipaddress-newowner-1)
[acceptOwnership()](#acceptownership)

OWNABLE
[owner()](#owner-→-address)
[_checkOwner()](#_checkowner)
[renounceOwnership()](#renounceownership)

**EVENTS**
[OwnershipTransferStarted(previousOwner, newOwner)](#ownershiptransferstartedaddress-indexed-previousowner-address-indexed-newowner)

OWNABLE
OwnershipTransferred(previousOwner, newOwner)

#### pendingOwner() → address
public#
返回待定所有者的地址。

#### transferOwnership(address newOwner)
public#
开始将合约的所有权转移给新账户。如果有待处理的转移，则替换它。只能由当前所有者调用。

#### _transferOwnership(address newOwner)
internal#
将合约的所有权转移到一个新账户（newOwner）并删除任何待定的所有者。内部函数，没有访问限制。

#### acceptOwnership()
public#
新业主接受所有权转移。

#### OwnershipTransferStarted(address indexed previousOwner, address indexed newOwner)
事件#

## IAccessControl
```
import "@openzeppelin/contracts/access/IAccessControl.sol";
```
AccessControl的外部接口声明支持ERC165检测。

**FUNCTIONS**
[hasRole(role, account)](#hasrolebytes32-role-address-account-→-bool)
[getRoleAdmin(role)](#getroleadminbytes32-role-→-bytes32)
[grantRole(role, account)](#grantrolebytes32-role-address-account)
[revokeRole(role, account)](#revokerolebytes32-role-address-account)
[renounceRole(role, account)](#renouncerolebytes32-role-address-account)

**EVENTS**
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### hasRole(bytes32 role, address account) → bool
外部#
如果账户已被授予角色，则返回true。

#### getRoleAdmin(bytes32 role) → bytes32
外部#
返回控制角色的管理员角色。请参见*grantRole*和*revokeRole*。

要更改角色的管理员，请使用*AccessControl._setRoleAdmin*。

#### grantRole(bytes32 role, address account)
外部#
授予账户角色。
如果账户之前没有被授予角色，则发出一个RoleGranted事件。
要求：
* 调用者必须具有角色管理员角色。

#### revokeRole(bytes32 role, address account)
外部#
从账户中撤销角色。
如果账户已被授予角色，则发出一个*RoleRevoked*事件。
要求：
* 调用者必须具有角色的管理员角色。

#### renounceRole(bytes32 role, address account)
外部#
从调用账户中撤销角色。
角色通常通过grantRole和revokeRole进行管理：此函数的目的是为了提供一种机制，以便在账户受到攻击时失去其特权（例如，当信任的设备遗失时）。
如果调用账户已被授予角色，则发出*RoleRevoked*事件。
要求：
* 调用者必须是账户。

#### RoleAdminChanged(bytes32 indexed role, bytes32 indexed previousAdminRole, bytes32 indexed newAdminRole)
事件#
当将newAdminRole设置为角色的管理员角色时，替换了previousAdminRole时，会发出此事件。
尽管没有发出RoleAdminChanged信号，DEFAULT_ADMIN_ROLE仍是所有角色的起始管理员。
自v3.1起可用。

#### RoleGranted(bytes32 indexed role, address indexed account, address indexed sender)
事件#
当账户被授予角色时发出。
发送者是发起合约调用的账户，是管理员角色持有者，除非使用*AccessControl._setupRole*。

#### RoleRevoked(bytes32 indexed role, address indexed account, address indexed sender)
事件#
当账户被撤销角色时发出。
发送者是发起合约调用的账户：- 如果使用revokeRole，则是管理员角色持有人- 如果使用renounceRole，则是角色持有人（即账户）。

### AccessControl
```
import "@openzeppelin/contracts/access/AccessControl.sol";
```

合约模块，允许儿童实现基于角色的访问控制机制。这是一个轻量级版本，不允许通过访问合约事件日志以外的方式枚举角色成员。一些应用程序可能会从链上可枚举性中受益，对于这些情况，请参阅[AccessControlEnumerable](#accesscontrolenumerable)。

角色由它们的bytes32标识符引用。这些应该在外部API中公开，并且是唯一的。实现这一点的最好方法是使用公共常量哈希摘要：
```
bytes32 public constant MY_ROLE = keccak256("MY_ROLE");
```

角色可以用来表示一组权限。为了限制对函数调用的访问，使用[hasRole](#hasrolebytes32-role-address-account-→-bool)：
```
function foo() public {
    require(hasRole(MY_ROLE, msg.sender));
    ...
}
```

角色可以通过[grantRole](#grantrolebytes32-role-address-account-1)和[revokeRole](#revokerolebytes32-role-address-account-1)函数动态地授予和撤销。每个角色都有一个关联的管理员角色，只有拥有角色管理员角色的帐户才能调用[grantRole](#grantrolebytes32-role-address-account-1)和[revokeRole](#revokerolebytes32-role-address-account-1)。

默认情况下，所有角色的管理员角色都是DEFAULT_ADMIN_ROLE，这意味着只有拥有此角色的帐户才能授予或撤销其他角色。可以使用[_setRoleAdmin](#_setroleadminbytes32-role-bytes32-adminrole)创建更复杂的角色关系。


> WARNING
DEFAULT_ADMIN_ROLE也是其自己的管理员：它有权限授予和撤销此角色。应该采取额外的预防措施来保护已被授予该角色的帐户。我们建议使用[AccessControlDefaultAdminRules](#accesscontroldefaultadminrules)为该角色执行额外的安全措施。

**MODIFIERS**
[onlyRole(role)](#onlyrolebytes32-role)

**FUNCTIONS**
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[hasRole(role, account)](#hasrolebytes32-role-address-account-→-bool)
[_checkRole(role)](#_checkrolebytes32-role)
[_checkRole(role, account)](#_checkrolebytes32-role)
[getRoleAdmin(role)](#getroleadminbytes32-role-→-bytes32)
[grantRole(role, account)](#grantrolebytes32-role-address-account)
[revokeRole(role, account)](#revokerolebytes32-role-address-account)
[renounceRole(role, account)](#renouncerolebytes32-role-address-account)
[_setupRole(role, account)](#_setuprolebytes32-role-address-account)
[_setRoleAdmin(role, adminRole)](#_setroleadminbytes32-role-bytes32-adminrole)
[_grantRole(role, account)](#_grantrolebytes32-role-address-account)
[_revokeRole(role, account)](#_revokerolebytes32-role-address-account)

**EVENTS**
IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### onlyRole(bytes32 role)
修饰词#
检查账户是否具有特定角色的修饰符。如果没有，将返回包含所需角色的标准化消息。
回滚原因的格式由以下正则表达式给出：

```
/^AccessControl: account (0x[0-9a-f]{40}) is missing role (0x[0-9a-f]{64})$/
```

*自v4.1版本以来可用。*

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查看[IERC165.supportsInterface](../API/Utils.md#supportsinterfacebytes4-interfaceid-→-bool)。

#### hasRole(bytes32 role, address account) → bool
public#
如果账户被授予了角色，则返回 true。

#### _checkRole(bytes32 role)
internal#
如果 _msgSender() 没有角色，则使用标准消息进行回退。重写此函数会更改 [onlyRole](#onlyrolebytes32-role) 修饰符的行为。
回退消息的格式在 [_checkRole](#_checkrolebytes32-role-address-account) 中描述。
*自 v4.6 起可用。*

#### _checkRole(bytes32 role, address account)
internal#
如果账户缺少角色，请使用标准消息进行回退。
回退原因的格式由以下正则表达式给出：
```
/^AccessControl: account (0x[0-9a-f]{40}) is missing role (0x[0-9a-f]{64})$/
```

#### getRoleAdmin(bytes32 role) → bytes32
public#
返回控制角色的管理员角色。请参阅[grantRole](#grantrolebytes32-role-address-account-1)和[revokeRole](#revokerolebytes32-role-address-account-1)。
要更改角色的管理员，请使用[_setRoleAdmin](#_setroleadminbytes32-role-bytes32-adminrole-1)。

#### grantRole(bytes32 role, address account)
public#
授予账户角色。
如果账户尚未被授予角色，则发出[RoleGranted](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。
要求：
* 调用者必须具有角色的管理员角色。
可能会发出[RoleGranted](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。

#### revokeRole(bytes32 role, address account)
public#
从帐户中撤销角色。
如果该帐户已被授予角色，则会发出一个[RoleRevoked](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。
要求：
* 调用者必须具有该角色的管理员角色。
可能会发出一个[RoleRevoked](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。

#### renounceRole(bytes32 role, address account)
public#
从调用账户中撤销角色。
角色通常通过[grantRole](#grantrolebytes32-role-address-account-1)和[revokeRole](#revokerolebytes32-role-address-account-1)进行管理：此函数的目的是为了提供一种机制，以便在账户被攻击时失去其特权（例如当信任的设备丢失时）。
如果调用账户已被撤销角色，则发出[RoleRevoked](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。
要求：
* 调用者必须是账户。
可能会发出[RoleRevoked](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。

#### _setupRole(bytes32 role, address account)
internal#
授予账户角色。
如果账户尚未被授予角色，则会发出[RoleGranted](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。请注意，与[grantRole](#grantrolebytes32-role-address-account-1)不同，此函数不对调用账户进行任何检查。
可能会发出[RoleGranted](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。

> WARNING
只应该在设置系统的初始角色时从构造函数中调用此函数。
在任何其他方式中使用此函数实际上是绕过[AccessControl](#accesscontrol)强制实施的管理系统。

> NOTE
此函数已过时，建议使用[_grantRole](#_grantrolebytes32-role-address-account)。

#### _setRoleAdmin(bytes32 role, bytes32 adminRole)
internal#
将adminRole设置为角色的管理员角色。

触发一个[RoleAdminChanged](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)事件。

#### _grantRole(bytes32 role, address account)
internal#
授予账户角色。
内部功能没有访问限制。
可能会发出[RoleGranted](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。

#### _revokeRole(bytes32 role, address account)
internal#
从帐户中撤销角色。
内部函数，没有访问限制。
可能会发出[RoleRevoked](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)事件。

### AccessControlCrossChain
```
import "@openzeppelin/contracts/access/AccessControlCrossChain.sol";
```

[AccessControl](#accesscontrol)的扩展，支持跨链访问管理。对于每个角色，该扩展实现了一个等效的“别名”角色，用于限制来自其他链的调用。

例如，如果一个函数myFunction受到onlyRole(SOME_ROLE)的保护，如果一个地址x具有角色SOME_ROLE，它将能够直接调用myFunction。然而，来自另一条链上相同地址的钱包或合约将无法调用此函数。为了这样做，它需要拥有角色_crossChainRoleAlias(SOME_ROLE)。

这种别名是必要的，以保护不同链上控制冲突实体的相同地址上存在的多个合约。

*自v4.6以来可用。*

**FUNCTIONS**
[_checkRole(role)](#_checkrolebytes32-role-1)
[_crossChainRoleAlias(role)](#_crosschainrolealiasbytes32-role-→-bytes32)

CROSSCHAINENABLED
[_isCrossChain()](./Crosschain.md#_iscrosschain-→-bool)
[_crossChainSender()](./Crosschain.md#_crosschainsender-→-address)

ACCESSCONTROL
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[hasRole(role, account)](#hasrolebytes32-role-address-account-→-bool)
[_checkRole(role, account)](#_checkrolebytes32-role)
[getRoleAdmin(role)](#getroleadminbytes32-role-→-bytes32)
[grantRole(role, account)](#grantrolebytes32-role-address-account-1)
[revokeRole(role, account)](#revokerolebytes32-role-address-account-1)
[renounceRole(role, account)](#renouncerolebytes32-role-address-account-1)
[_setupRole(role, account)](#_setuprolebytes32-role-address-account)
[_setRoleAdmin(role, adminRole)](#_setroleadminbytes32-role-bytes32-adminrole)
[_grantRole(role, account)](#_grantrolebytes32-role-address-account)
[_revokeRole(role, account)](#_revokerolebytes32-role-address-account)

**EVENTS**
IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### _checkRole(bytes32 role)
internal#
请查看[AccessControl._checkRole](#_checkrolebytes32-role-address-account)。

#### _crossChainRoleAlias(bytes32 role) → bytes32
internal#
返回与角色对应的别名角色。

### IAccessControlEnumerable
```
import "@openzeppelin/contracts/access/IAccessControlEnumerable.sol";
```
AccessControlEnumerable的外部接口声明为支持ERC165检测。

**FUNCTIONS**
[getRoleMember(role, index)](#getrolememberbytes32-role-uint256-index-→-address)
[getRoleMemberCount(role)](#getrolemembercountbytes32-role-→-uint256)

IACCESSCONTROL
[hasRole(role, account)](#hasrolebytes32-role-address-account-→-bool)
[getRoleAdmin(role)](#getroleadminbytes32-role-→-bytes32)
[grantRole(role, account)](#grantrolebytes32-role-address-account)
[revokeRole(role, account)](#revokerolebytes32-role-address-account)
[renounceRole(role, account)](#renouncerolebytes32-role-address-account)

EVENTS
IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### getRoleMember(bytes32 role, uint256 index) → address
外部#
返回具有角色的帐户之一。索引必须是0和[getRoleMemberCount](#getrolemembercountbytes32-role-→-uint256)之间的值，不包括这两个值。

角色承载者没有特定的排序方式，其排序可能随时更改。

> WARNING
使用[getRoleMember](#getrolememberbytes32-role-uint256-index-→-address)和[getRoleMemberCount](#getrolemembercountbytes32-role-→-uint256)时，请确保在同一块上执行所有查询。有关更多信息，请参见以下[论坛帖子](https://forum.openzeppelin.com/t/iterating-over-elements-on-enumerableset-in-openzeppelin-contracts/2296)。

#### getRoleMemberCount(bytes32 role) → uint256
外部#
返回具有角色的帐户数量。可与[getRoleMember](#getrolememberbytes32-role-uint256-index-→-address)一起使用，枚举角色的所有承担者。

### AccessControlEnumerable
```
import "@openzeppelin/contracts/access/AccessControlEnumerable.sol";
```

[AccessControl](#accesscontrol)的扩展，允许枚举每个角色的成员。

**FUNCTIONS**
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[getRoleMember(role, index)](#getrolememberbytes32-role-uint256-index-→-address)
[getRoleMemberCount(role)](#getrolemembercountbytes32-role-→-uint256)
[_grantRole(role, account)](#_grantrolebytes32-role-address-account-1)
[_revokeRole(role, account)](#_revokerolebytes32-role-address-account-1)

ACCESSCONTROL
[hasRole(role, account)](#hasrolebytes32-role-address-account-→-bool)
[_checkRole(role)](#_checkrolebytes32-role)
[_checkRole(role, account)](#_checkrolebytes32-role-address-account)
[getRoleAdmin(role)](#getroleadminbytes32-role-→-bytes32)
[grantRole(role, account)](#grantrolebytes32-role-address-account-1)
[revokeRole(role, account)](#revokerolebytes32-role-address-account-1)
[renounceRole(role, account)](#renouncerolebytes32-role-address-account-1)
[_setupRole(role, account)](#_setuprolebytes32-role-address-account)
[_setRoleAdmin(role, adminRole)](#_setroleadminbytes32-role-bytes32-adminrole)

**EVENTS**
IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### supportsInterface(bytes4 interfaceId) → bool
public#
请参见[IERC165.supportsInterface](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool)。

#### getRoleMember(bytes32 role, uint256 index) → address
public#
返回具有该角色的帐户之一。索引必须是介于0和[getRoleMemberCount](#getrolemembercountbytes32-role-→-uint256)之间的值，不包括这两个值。

角色持有人没有特定的排序方式，并且它们的顺序可能随时更改。

> WARNING
在使用[getRoleMember](#getrolememberbytes32-role-uint256-index-→-address)和[getRoleMemberCount](#getrolemembercountbytes32-role-→-uint256)时，请确保在同一块上执行所有查询。有关更多信息，请参见以下[论坛帖子](https://forum.openzeppelin.com/t/iterating-over-elements-on-enumerableset-in-openzeppelin-contracts/2296)。

#### getRoleMemberCount(bytes32 role) → uint256
public#
返回拥有某个角色的账户数量。可以与[getRoleMember](#getrolememberbytes32-role-uint256-index-→-address)一起使用，枚举角色的所有持有者。

#### _grantRole(bytes32 role, address account)
internal#
重载[_grantRole](#_grantrolebytes32-role-address-account-1)以跟踪可枚举成员资格

#### _revokeRole(bytes32 role, address account)
internal#
重载 [_revokeRole](#_revokerolebytes32-role-address-account-1) 以跟踪可枚举的成员资格

### AccessControlDefaultAdminRules
```
import "@openzeppelin/contracts/access/AccessControlDefaultAdminRules.sol";
```

[AccessControl](#accesscontrol)的扩展，允许指定特殊规则来管理DEFAULT_ADMIN_ROLE持有者，这是一个关键角色，具有对其他可能拥有特权权限的角色的特殊权限。

如果没有为特定角色分配管理员角色，则DEFAULT_ADMIN_ROLE的持有者将能够授予和撤销管理员角色。

该合约在[AccessControl](#accesscontrol)的基础上实现了以下风险缓解措施：
* 自部署以来，只有一个帐户持有DEFAULT_ADMIN_ROLE，直到可能被放弃。
* 强制执行将DEFAULT_ADMIN_ROLE转移到另一个帐户的2步过程。
* 强制执行可配置的两个步骤之间的延迟，在接受转移之前可以取消。
* 延迟可以通过调度进行更改，请参见[changeDefaultAdminDelay](#changedefaultadmindelayuint48-newdelay)。
* 不可能使用另一个角色来管理DEFAULT_ADMIN_ROLE。

示例用法：
```
contract MyToken is AccessControlDefaultAdminRules {
  constructor() AccessControlDefaultAdminRules(
    3 days,
    msg.sender // Explicit initial `DEFAULT_ADMIN_ROLE` holder
   ) {}
}
```

*自v4.9版本起可用。*

**FUNCTIONS**
[constructor(initialDelay, initialDefaultAdmin)](#constructoruint48-initialdelay-address-initialdefaultadmin)
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-e28692-bool-2)
[owner()](#owner-e28692-address-1)
[grantRole(role, account)](#grantrolebytes32-role-address-account-2)
[revokeRole(role, account)](#revokerolebytes32-role-address-account-2)
[renounceRole(role, account)](#renouncerolebytes32-role-address-account-2)
[_grantRole(role, account)](#_grantrolebytes32-role-address-account-2)
[_revokeRole(role, account)](#_revokerolebytes32-role-address-account-2)
[_setRoleAdmin(role, adminRole)](#_setroleadminbytes32-role-bytes32-adminrole-1)
[defaultAdmin()](#defaultadmin-→-address)
[pendingDefaultAdmin()](#pendingdefaultadmin-→-address-newadmin-uint48-schedule)
[defaultAdminDelay()](#defaultadmindelay-→-uint48)
[pendingDefaultAdminDelay()](#pendingdefaultadmindelay-→-uint48-newdelay-uint48-schedule)
[defaultAdminDelayIncreaseWait()](#defaultadmindelayincreasewait-→-uint48)
[beginDefaultAdminTransfer(newAdmin)](#begindefaultadmintransferaddress-newadmin)
[_beginDefaultAdminTransfer(newAdmin)](#_begindefaultadmintransferaddress-newadmin)
[cancelDefaultAdminTransfer()](#canceldefaultadmintransfer)
[_cancelDefaultAdminTransfer()](#_canceldefaultadmintransfer)
[acceptDefaultAdminTransfer()](#acceptdefaultadmintransfer)
[_acceptDefaultAdminTransfer()](#_acceptdefaultadmintransfer)
[changeDefaultAdminDelay(newDelay)](#changedefaultadmindelayuint48-newdelay)
[_changeDefaultAdminDelay(newDelay)](#_changedefaultadmindelayuint48-newdelay)
[rollbackDefaultAdminDelay()](#rollbackdefaultadmindelay)
[_rollbackDefaultAdminDelay()](#_rollbackdefaultadmindelay)
[_delayChangeWait(newDelay)](#_delaychangewaituint48-newdelay-→-uint48)

ACCESSCONTROL
[hasRole(role, account)](#hasrolebytes32-role-address-account-e28692-bool-1)
[_checkRole(role)](#_checkrolebytes32-role)
[_checkRole(role, account)](#_checkrolebytes32-role-address-account)
[getRoleAdmin(role)](#getroleadminbytes32-role-e28692-bytes32-1)
[_setupRole(role, account)](#_setuprolebytes32-role-address-account)

**EVENTS**
IACCESSCONTROLDEFAULTADMINRULES
DefaultAdminTransferScheduled(newAdmin, acceptSchedule)
DefaultAdminTransferCanceled()
DefaultAdminDelayChangeScheduled(newDelay, effectSchedule)
DefaultAdminDelayChangeCanceled()

IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

#### constructor(uint48 initialDelay, address initialDefaultAdmin)
internal#
设置默认[管理员延迟](#defaultadmindelay-→-uint48)和默认管理员地址的[初始值](#defaultadmin-→-address)。

#### supportsInterface(bytes4 interfaceId) → bool
public#
请参见[IERC165.supportsInterface](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool)。

#### owner() → address
public#
请参见[IERC5313.owner](./Interfaces.md#owner-→-address) .

#### grantRole(bytes32 role, address account)
public#
查看[AccessControl.grantRole](#grantrolebytes32-role-address-account-1)。针对DEFAULT_ADMIN_ROLE进行还原。

#### revokeRole(bytes32 role, address account)
public#
请参阅[AccessControl.revokeRole](#revokerolebytes32-role-address-account-1)。默认情况下撤销DEFAULT_ADMIN_ROLE角色。

#### renounceRole(bytes32 role, address account)
public#
请参阅[AccessControl.renounceRole](#renouncerolebytes32-role-address-account-1)。

对于DEFAULT_ADMIN_ROLE，它只允许通过首先调用[beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin) 到address（0）来进行两步操作，因此在调用此函数时需要确认[pendingDefaultAdmin](#pendingdefaultadmin-→-address-newadmin-uint48-schedule)计划也已经通过。

执行后，将无法调用onlyRole(DEFAULT_ADMIN_ROLE)函数。

> NOTE
放弃DEFAULT_ADMIN_ROLE将使合约失去 [defaultAdmin](#defaultadmin-→-address)，从而禁用仅适用于其的任何功能，并且无法重新分配非受管理角色。

#### _grantRole(bytes32 role, address account)
internal#
请参见[AccessControl._grantRole](#_grantrolebytes32-role-address-account-1)。

对于DEFAULT_ADMIN_ROLE，只有在没有[defaultAdmin](#defaultadmin-→-address)或者该角色已被先前放弃的情况下才允许授予。

> NOTE
通过另一种机制公开此函数可能会使DEFAULT_ADMIN_ROLE再次可分配。请确保在你的实现中保证这是预期的行为。

#### _revokeRole(bytes32 role, address account)
internal#
请查看[AccessControl._revokeRole](#_revokerolebytes32-role-address-account-1)函数。

#### _setRoleAdmin(bytes32 role, bytes32 adminRole)
internal#
参见 [AccessControl._setRoleAdmin](#_setroleadminbytes32-role-bytes32-adminrole)。对于DEFAULT_ADMIN_ROLE进行还原。

#### defaultAdmin() → address
public#
返回当前 DEFAULT_ADMIN_ROLE 持有者的地址。

#### pendingDefaultAdmin() → address newAdmin, uint48 schedule
public#
返回一个新管理员和接受日程的元组。

日程表过期后，新管理员将能够通过调用 [acceptDefaultAdminTransfer](#acceptdefaultadmintransfer) 接受 [defaultAdmin](#defaultadmin-→-address) 角色，完成角色转移。

acceptSchedule 中只有零值表示没有待处理的管理员转移。

> NOTE
一个零地址的newAdmin意味着 [defaultAdmin](#defaultadmin-→-address)正在放弃。

#### defaultAdminDelay() → uint48
public#
返回启动[defaultAdmin](#defaultadmin-→-address) 转移的接受所需的延迟时间。

在调用[beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin)设置接受计划时，此延迟将添加到当前时间戳。

> NOTE
如果已经安排了延迟更改，则该函数将在安排过程中立即生效，返回新的延迟。请参见[changeDefaultAdminDelay](#changedefaultadmindelayuint48-newdelay)。

#### pendingDefaultAdminDelay() → uint48 newDelay, uint48 schedule
public#
返回一个新延迟和一个效果计划的元组。

计划过期后，对于每个使用[beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin)开始的新[defaultAdmin](#defaultadmin-→-address)转移，新延迟将立即生效。

仅在effectSchedule中为零表示没有挂起的延迟更改。

> NOTE
一个仅为newDelay的零值意味着在效果计划之后，下一个[defaultAdminDelay](#defaultadmindelay-→-uint48)将为零。

#### defaultAdminDelayIncreaseWait() → uint48
public#
将默认[defaultAdminDelay](#defaultadmindelay-→-uint48) 增加的最长时间（使用[changeDefaultAdminDelay](#changedefaultadmindelayuint48-newdelay)计划）生效的秒数。默认为5天。

当计划增加[默认管理员延迟](#defaultadmindelay-→-uint48)时，它会在新的延迟时间过去后生效，目的是为了给予足够的时间来撤销任何意外更改（例如使用毫秒而不是秒），以避免合约被锁定。但是，为了避免过度计划，此函数将被限制等待时间，并且可以被重写以进行自定义的[默认管理员延迟](#defaultadmindelay-→-uint48)增加计划。

> IMPORTANT
在重写此值时，请确保添加合理的时间，否则存在设置新延迟的风险，该延迟几乎立即生效，而在输入错误的情况下无法进行人为干预（例如，设置毫秒而不是秒）。

#### beginDefaultAdminTransfer(address newAdmin)
public#
通过设置一个待确认的[pendingDefaultAdmin](#pendingdefaultadmin-→-address-newadmin-uint48-schedule)，以当前时间戳加上[defaultAdminDelay](#defaultadmindelay-→-uint48)为限，启动[defaultAdmin](#defaultadmin-→-address)转移。

要求：
* 只能由当前[defaultAdmin](#defaultadmin-→-address)调用。

触发DefaultAdminRoleChangeStarted事件。

#### _beginDefaultAdminTransfer(address newAdmin)
internal#
查看 [beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin)。

内部函数，没有访问限制。

#### cancelDefaultAdminTransfer()
public#
取消先前使用[beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin)启动的[ defaultAdmin](#defaultadmin-→-address)转移。

尚未接受的[pendingDefaultAdmin](#pendingdefaultadmin-→-address-newadmin-uint48-schedule)也可以使用此函数取消。

要求：
* 只能由当前[defaultAdmin](#defaultadmin-→-address)调用。

可能会发出DefaultAdminTransferCanceled事件。

#### _cancelDefaultAdminTransfer()
internal#
查看[cancelDefaultAdminTransfer](#canceldefaultadmintransfer)。

这是一个没有访问限制的内部函数。

#### acceptDefaultAdminTransfer()
public#
完成之前使用[beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin)开始的[defaultAdmin](#defaultadmin-→-address)转移。

调用该函数后：
* DEFAULT_ADMIN_ROLE应授予调用者。
* DEFAULT_ADMIN_ROLE应从以前的持有者中撤销。
* [pendingDefaultAdmin](#pendingdefaultadmin-→-address-newadmin-uint48-schedule)应重置为零值。

要求：
* 只能由[pendingDefaultAdmin](#pendingdefaultadmin-→-address-newadmin-uint48-schedule)的newAdmin调用。
* [pendingDefaultAdmin](#pendingdefaultadmin-→-address-newadmin-uint48-schedule)的acceptSchedule应已通过。

#### _acceptDefaultAdminTransfer()
internal#
查看[acceptDefaultAdminTransfer](#acceptdefaultadmintransfer)。
这是一个没有访问限制的内部函数。

#### changeDefaultAdminDelay(uint48 newDelay)
public#
通过设置一个[pendingDefaultAdminDelay](#pendingdefaultadmindelay-→-uint48-newdelay-uint48-schedule)来启动一个[defaultAdminDelay](#defaultadmindelay-→-uint48)更新，该延迟计划在当前时间戳加上[defaultAdminDelay](#defaultadmindelay-→-uint48)后生效。

该函数保证在调用此方法时和[pendingDefaultAdminDelay](#pendingdefaultadmindelay-→-uint48-newdelay-uint48-schedule)生效计划之间的任何对[beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin)的调用都将使用在调用之前设置的当前[defaultAdminDelay](#defaultadmindelay-→-uint48)。

[pendingDefaultAdminDelay](#pendingdefaultadmindelay-→-uint48-newdelay-uint48-schedule)的生效计划设计为等待计划并使用新延迟调用[beginDefaultAdminTransfer](#begindefaultadmintransferaddress-newadmin)将至少需要与另一个[defaultAdmin](#defaultadmin-→-address)完整转移（包括接受）相同的时间。

该计划适用于两种情况：
* 当延迟更改为较大值时，计划是block.timestamp + newDelay，上限为[defaultAdminDelayIncreaseWait](#defaultadmindelayincreasewait-→-uint48)。
* 当延迟更改为较短的时间时，计划是block.timestamp +（当前延迟 - 新延迟）。

如果一个未生效的[pendingDefaultAdminDelay](#pendingdefaultadmindelay-→-uint48-newdelay-uint48-schedule)将被取消，以支持新的计划更改。
要求：
* 只能由当前的[defaultAdmin](#defaultadmin-→-address)调用。

发出DefaultAdminDelayChangeScheduled事件，可能还会发出DefaultAdminDelayChangeCanceled事件。

#### _changeDefaultAdminDelay(uint48 newDelay)
internal#
查看[changeDefaultAdminDelay](#changedefaultadmindelayuint48-newdelay)。
内部函数，没有访问限制。

#### rollbackDefaultAdminDelay()
public#
取消预定的[defaultAdminDelay](#defaultadmindelay-→-uint48)更改。
要求：
* 只能由当前的[defaultAdmin](#defaultadmin-→-address)调用。

可能会发出DefaultAdminDelayChangeCanceled事件。

#### _rollbackDefaultAdminDelay()
internal#
查看[rollbackDefaultAdminDelay](#rollbackdefaultadmindelay)。
这是一个没有访问限制的内部函数。

#### _delayChangeWait(uint48 newDelay) → uint48
internal#
返回在newDelay成为新的[defaultAdminDelay](#defaultadmindelay-→-uint48)之后需要等待的秒数。
返回的值保证了如果延迟被减少，它将在遵守先前设置的延迟的等待后生效。
请参阅[defaultAdminDelayIncreaseWait](#defaultadmindelayincreasewait-→-uint48)。
