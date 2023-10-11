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
constructor(initialOwner)
[owner()](#owner-→-address)
[_checkOwner()](#_checkowner)
[renounceOwnership()](#renounceownership)
[transferOwnership(newOwner)](#transferownershipaddress-newowner)
[_transferOwnership(newOwner)](#_transferownershipaddress-newowner)

**EVENTS**
[OwnershipTransferred(previousOwner, newOwner)](#ownershiptransferredaddress-indexed-previousowner-address-indexed-newowner)

**ERRORS**
OwnableUnauthorizedAccount(account)

OwnableInvalidOwner(owner)

#### onlyOwner()
modifier#
如果被除了所有者以外的任何账户调用，就会抛出异常。

#### constructor(address initialOwner)
internal#
初始化合约，将部署者提供的地址设置为初始所有者。

#### owner() → address
public#
返回当前所有者的地址。

#### _checkOwner()
internal#
如果发送方不是所有者，则抛出异常。

#### renounceOwnership()
public#
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
event#

#### OwnableUnauthorizedAccount(address account)
error#
呼叫者账户无权进行此操作。

#### OwnableInvalidOwner(address owner)
error#
所有者不是一个有效的所有者账户。(例如，地址(0))

### [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/Ownable2Step.sol)
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

**ERRORS**
OWNABLE
OwnableUnauthorizedAccount(account)

OwnableInvalidOwner(owner)

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
event#

## [IAccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/IAccessControl.sol)
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

**ERRORS**
AccessControlUnauthorizedAccount(account, neededRole)

AccessControlBadConfirmation()

#### hasRole(bytes32 role, address account) → bool
external#
如果账户已被授予角色，则返回true。

#### getRoleAdmin(bytes32 role) → bytes32
external#
返回控制角色的管理员角色。请参见*grantRole*和*revokeRole*。

要更改角色的管理员，请使用*AccessControl._setRoleAdmin*。

#### grantRole(bytes32 role, address account)
external#
授予账户角色。
如果账户之前没有被授予角色，则发出一个RoleGranted事件。
要求：
* 调用者必须具有角色管理员角色。

#### revokeRole(bytes32 role, address account)
external#
从账户中撤销角色。
如果账户已被授予角色，则发出一个*RoleRevoked*事件。
要求：
* 调用者必须具有角色的管理员角色。

#### renounceRole(bytes32 role, address account)
external#
从调用账户中撤销角色。
角色通常通过grantRole和revokeRole进行管理：此函数的目的是为了提供一种机制，以便在账户受到攻击时失去其特权（例如，当信任的设备遗失时）。
如果调用账户已被授予角色，则发出*RoleRevoked*事件。
要求：
* 调用者必须是账户。

#### RoleAdminChanged(bytes32 indexed role, bytes32 indexed previousAdminRole, bytes32 indexed newAdminRole)
event#
当将newAdminRole设置为角色的管理员角色时，替换了previousAdminRole时，会发出此事件。
尽管没有发出RoleAdminChanged信号，DEFAULT_ADMIN_ROLE仍是所有角色的起始管理员。

#### RoleGranted(bytes32 indexed role, address indexed account, address indexed sender)
event#
当账户被授予角色时发出。

发送者是发起合约调用的账户，除使用{AccessControl-_setupRole}时，其为管理员角色持有者。

#### RoleRevoked(bytes32 indexed role, address indexed account, address indexed sender)
event#
当账户被撤销角色时发出。
发送者是发起合约调用的账户：- 如果使用revokeRole，则是管理员角色持有人- 如果使用renounceRole，则是角色持有人（即账户）。

#### AccessControlUnauthorizedAccount(address account, bytes32 neededRole)
error#
该账户缺少一个角色。

#### AccessControlBadConfirmation()
error#
函数的调用者不是预期的那个。

> NOTE
不要与AccessControlUnauthorizedAccount混淆。

### [AccessControl](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/AccessControl.sol)
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
renounceRole(role, callerConfirmation)
_setRoleAdmin(role, adminRole)
_grantRole(role, account)
_revokeRole(role, account)
DEFAULT_ADMIN_ROLE()

**EVENTS**
IACCESSCONTROL
[RoleAdminChanged(role, previousAdminRole, newAdminRole)](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)
[RoleGranted(role, account, sender)](#rolegrantedbytes32-indexed-role-address-indexed-account-address-indexed-sender)
[RoleRevoked(role, account, sender)](#rolerevokedbytes32-indexed-role-address-indexed-account-address-indexed-sender)

**ERRORS**
IACCESSCONTROL
AccessControlUnauthorizedAccount(account, neededRole)
AccessControlBadConfirmation()

#### onlyRole(bytes32 role)
修饰词#
检查账户是否具有特定角色的修饰符。如果没有所需角色，将返回包含所需角色的*AccessControlUnauthorizedAccount*错误。

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查看[IERC165.supportsInterface](../API/Utils.md#supportsinterfacebytes4-interfaceid-→-bool)。

#### hasRole(bytes32 role, address account) → bool
public#
如果账户被授予了角色，则返回 true。

#### _checkRole(bytes32 role)
internal#
如果_msgSender()缺少角色，则会返回AccessControlUnauthorizedAccount错误。重写这个函数会改变onlyRole修饰符的行为。

#### _checkRole(bytes32 role, address account)
internal#
如果账户缺少角色，则会以AccessControlUnauthorizedAccount错误进行回退。

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
将adminRole设置为角色的管理员角色。

触发一个*RoleAdminChanged*事件。

#### _setRoleAdmin(bytes32 role, bytes32 adminRole)
internal#
将adminRole设置为角色的管理员角色。

触发一个[RoleAdminChanged](#roleadminchangedbytes32-indexed-role-bytes32-indexed-previousadminrole-bytes32-indexed-newadminrole)事件。

#### _grantRole(bytes32 role, address account) → bool
internal#
尝试向账户授予角色，并返回一个布尔值，表示是否成功授予角色。

这是一个没有访问限制的内部函数。

可能会触发一个*RoleGranted*事件。

#### _revokeRole(bytes32 role, address account) → bool
internal#
尝试撤销账户的角色，并返回一个布尔值，表示角色是否被撤销。

这是一个没有访问限制的内部函数。

可能会发出一个*RoleRevoked*事件。

#### DEFAULT_ADMIN_ROLE() → bytes32
public#

## Extensions

### [IAccessControlEnumerable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/extensions/IAccessControlEnumerable.sol)\

```
import "@openzeppelin/contracts/access/extensions/IAccessControlEnumerable.sol";
```

声明AccessControlEnumerable的外部接口以支持ERC165检测。

**FUNCTIONS**
getRoleMember(role, index)
getRoleMemberCount(role)

IACCESSCONTROL
hasRole(role, account)
getRoleAdmin(role)
grantRole(role, account)
revokeRole(role, account)
renounceRole(role, callerConfirmation)

**EVENTS**
IACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)
RoleGranted(role, account, sender)
RoleRevoked(role, account, sender)

**ERRORS**
IACCESSCONTROL
AccessControlUnauthorizedAccount(account, neededRole)
AccessControlBadConfirmation()

#### getRoleMember(bytes32 role, uint256 index) → address
external#
返回具有角色的其中一个账户。索引必须是0到*getRoleMemberCount*之间的值，不包括getRoleMemberCount。

角色承担者并未按任何特定方式排序，他们的排序可能在任何时候改变。

> WARNING
在使用*getRoleMember*和*getRoleMemberCount*时，确保在同一块上执行所有查询。有关更多信息，请参阅以下[论坛帖子](https://forum.openzeppelin.com/t/iterating-over-elements-on-enumerableset-in-openzeppelin-contracts/2296)。

#### getRoleMemberCount(bytes32 role) → uint256
external#
返回具有角色的账户数量。可以与*getRoleMember*一起使用，枚举所有角色的承担者。

### [AccessControlEnumerable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/extensions/AccessControlEnumerable.sol)
```
import "@openzeppelin/contracts/access/extensions/AccessControlEnumerable.sol";
```

扩展*访问控制*，允许枚举每个角色的成员。

**FUNCTIONS**
supportsInterface(interfaceId)
getRoleMember(role, index)
getRoleMemberCount(role)
_grantRole(role, account)
_revokeRole(role, account)

ACCESSCONTROL
hasRole(role, account)
_checkRole(role)
_checkRole(role, account)
getRoleAdmin(role)
grantRole(role, account)
revokeRole(role, account)
renounceRole(role, callerConfirmation)
_setRoleAdmin(role, adminRole)
DEFAULT_ADMIN_ROLE()

**EVENTS**
IACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)
RoleGranted(role, account, sender)
RoleRevoked(role, account, sender)

**ERRORS**
IACCESSCONTROL
AccessControlUnauthorizedAccount(account, neededRole)
AccessControlBadConfirmation()

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查阅 *IERC165.supportsInterface*.

#### getRoleMember(bytes32 role, uint256 index) → address
public#
返回具有角色的其中一个账户。索引必须是0到getRoleMemberCount之间的值，不包括*getRoleMemberCount*。

角色承担者并未按任何特定方式排序，他们的排序可能会在任何时候改变。

> WARNING
在使用*getRoleMember*和*getRoleMemberCount*时，确保你在同一块上执行所有查询。有关更多信息，请参阅以下[论坛帖子](https://forum.openzeppelin.com/t/iterating-over-elements-on-enumerableset-in-openzeppelin-contracts/2296)。

#### getRoleMemberCount(bytes32 role) → uint256
public#
返回具有角色的账户数量。可以与*getRoleMember*一起使用，列举所有角色的持有者。

#### _grantRole(bytes32 role, address account) → bool
internal#
重载*AccessControl._grantRole*以跟踪可枚举的成员资格

#### _revokeRole(bytes32 role, address account) → bool
internal#
重载*AccessControl._revokeRole*以跟踪可枚举的成员资格

### [IAccessControlDefaultAdminRules](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/extensions/IAccessControlDefaultAdminRules.sol)
