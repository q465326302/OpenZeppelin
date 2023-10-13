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

```
import "@openzeppelin/contracts/access/extensions/IAccessControlDefaultAdminRules.sol";
```

声明访问控制默认管理员规则的外部接口以支持ERC165检测。

**FUNCTIONS**
defaultAdmin()

pendingDefaultAdmin()

defaultAdminDelay()

pendingDefaultAdminDelay()

beginDefaultAdminTransfer(newAdmin)

cancelDefaultAdminTransfer()

acceptDefaultAdminTransfer()

changeDefaultAdminDelay(newDelay)

rollbackDefaultAdminDelay()

defaultAdminDelayIncreaseWait()

IACCESSCONTROL
hasRole(role, account)

getRoleAdmin(role)

grantRole(role, account)

revokeRole(role, account)

renounceRole(role, callerConfirmation)

**EVENTS**
DefaultAdminTransferScheduled(newAdmin, acceptSchedule)

DefaultAdminTransferCanceled()

DefaultAdminDelayChangeScheduled(newDelay, effectSchedule)

DefaultAdminDelayChangeCanceled()

IACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

**ERRORS**
AccessControlInvalidDefaultAdmin(defaultAdmin)

AccessControlEnforcedDefaultAdminRules()

AccessControlEnforcedDefaultAdminDelay(schedule)

IACCESSCONTROL
AccessControlUnauthorizedAccount(account, neededRole)

AccessControlBadConfirmation()

#### defaultAdmin() → address
external#
返回当前DEFAULT_ADMIN_ROLE持有者的地址。

#### pendingDefaultAdmin() → address newAdmin, uint48 acceptSchedule
external#
返回一个新管理员和接受时间表的元组。

在时间表通过后，新管理员将能够通过调用*acceptDefaultAdminTransfer*来接受*defaultAdmin*，完成角色转移。

只有在接受时间表中的零值表示没有待处理的管理员转移。

> NOTE
零地址的新管理员意味着正在放弃*defaultAdmin*。

#### defaultAdminDelay() → uint48
external#
返回开始*defaultAdmin*转移所需的延迟。

在调用*beginDefaultAdminTransfer*设置接受计划时，此延迟将添加到当前时间戳。

> NOTE
如果已经安排了延迟更改，一旦计划通过，它将立即生效，使此函数返回新的延迟。参见*changeDefaultAdminDelay*。

#### pendingDefaultAdminDelay() → uint48 newDelay, uint48 effectSchedule
external#
返回一个新延迟和效果计划的元组。

计划过后，新的延迟将立即对每个*beginDefaultAdminTransfer*。

只在效果计划中的零值表示没有待处理的延迟更改。

只对新延迟的零值意味着在效果计划后，下一个*默认管理员延返*将为零。

#### beginDefaultAdminTransfer(address newAdmin)
external#
启动*默认管理员转移*，通过设置一个*待处理的默认管理员*，该管理员将在当前时间戳加上*默认管理员延迟*后被接受。

要求：
* 只能由当前的*默认管理员调用*。

触发一个DefaultAdminRoleChangeStarted事件。

#### cancelDefaultAdminTransfer()
external#
取消之前已经开始的*默认管理员转移*。

还可以使用此功能取消尚未接受的*待处理默认管理员*。

要求：

只能由当前的*默认管理员*调用。

可能会发出一个DefaultAdminTransferCanceled事件。

#### acceptDefaultAdminTransfer()
external#
完成先前通过*beginDefaultAdminTransfer*开始的*默认管理员*转移。

调用此函数后：

* DEFAULT_ADMIN_ROLE应授予给调用者。

* DEFAULT_ADMIN_ROLE应从前持有者那里撤销。

* *pendingDefaultAdmin*应重置为零值。

要求：
* 只能由*pendingDefaultAdmin*的newAdmin调用。

* *pendingDefaultAdmin*的接受时间表应已过。

#### changeDefaultAdminDelay(uint48 newDelay)
external#
此函数通过设置一个*待定的默认管理员*延迟（*pendingDefaultAdminDelay*），在当前时间戳加上默认管理员延迟后生效，从而启动默认管理员延迟的更新。

此函数保证在此方法被调用的时间戳和*待定的默认管理员延迟*生效计划之间进行的任何对*beginDefaultAdminTransfer*的调用都将使用在调用之前设置的当前*默认管理员*延迟。

*待定的默认管理员延迟*的生效计划是这样定义的：等待到计划时间，然后用新的延迟调用*beginDefaultAdminTransfer*，至少需要与另一个*默认管理员*完整转移（包括接受）一样长的时间。

计划设计有两种情况：

* 当延迟被改为更长的时候，计划是block.timestamp + newDelay，上限是*defaultAdminDelayIncreaseWait*。

* 当延迟被改为更短的时候，计划是block.timestamp + (当前延迟 - 新延迟)。

一个从未生效的*待定默认管理员延迟*将被取消，以便于新的计划变更。

要求：

* 只能由当前的*默认管理员*调用。

会触发一个DefaultAdminDelayChangeScheduled事件，并可能触发一个DefaultAdminDelayChangeCanceled事件。

#### rollbackDefaultAdminDelay()
external#
取消预定的*defaultAdminDelay*更改。

要求：
* 只能由当前的*默认管理员*调用。

可能会发出一个DefaultAdminDelayChangeCanceled事件。\

#### defaultAdminDelayIncreaseWait() → uint48
external# 
此函数定义了将*默认管理员延迟*（通过*changeDefaultAdminDelay*计划）增加到生效的最长时间。默认为5天。

当计划增加*defaultAdminDelay*时，新的延迟过去后才会生效，目的是为了给足够的时间来撤销任何可能锁定合约的意外更改（例如，使用毫秒而不是秒）。然而，为了避免过多的计划，这个等待时间被此函数限制，并且可以被覆盖以自定义*defaultAdminDelay*增加的计划。

> IMPORTANT
在覆盖此值时，请确保添加合理的时间，否则，如果输入错误（例如，设置毫秒而不是秒），可能会立即生效一个新的高延迟，而没有人工干预的可能性。

#### DefaultAdminTransferScheduled(address indexed newAdmin, uint48 acceptSchedule)
event#
当启动*默认管理员*转移时发出，将newAdmin设置为只有在接受计划通过后才能通过调用*acceptDefaultAdminTransfer*成为*默认管理员*的下一个地址。

#### DefaultAdminTransferCanceled()
event#
当一个*待处理的默认管理员*被重置时发出，无论其计划如何，都会发出这个信号。

#### DefaultAdminDelayChangeScheduled(uint48 newDelay, uint48 effectSchedule)
event#
当开始更改*defaultAdminDelay*时发出，将newDelay设置为在effectSchedule通过后应用的下一个延迟。

#### DefaultAdminDelayChangeCanceled()
event#
当未通过其计划的*pendingDefaultAdminDelay*被重置时发出。

#### AccessControlInvalidDefaultAdmin(address defaultAdmin)
error#
新的默认管理员不是有效的默认管理员。

#### AccessControlEnforcedDefaultAdminRules()
error#
至少违反了以下规则之一：

* DEFAULT_ADMIN_ROLE只能由其自身管理。

* DEFAULT_ADMIN_ROLE一次只能由一个账户持有。

* 任何DEFAULT_ADMIN_ROLE的转移必须分两步进行，且需有延迟。

#### AccessControlEnforcedDefaultAdminDelay(uint48 schedule)
error#
默认管理员延迟的传输延迟是强制执行的，操作必须等待直到计划。

> NOTE
计划可以为0，表示没有计划的传输。

### [AccessControlDefaultAdminRules](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/extensions/AccessControlDefaultAdminRules.sol)
```
import "@openzeppelin/contracts/access/extensions/AccessControlDefaultAdminRules.sol";
```

扩展*AccessControl*，允许指定特殊规则来管理DEFAULT_ADMIN_ROLE持有者，这是一个敏感角色，具有对系统中可能具有特权权利的其他角色的特殊权限。

如果一个特定的角色没有分配管理员角色，那么DEFAULT_ADMIN_ROLE的持有者将有能力授予和撤销它。

该合约在*AccessControl*的基础上实施了以下风险缓解措施：

* 自部署以来，只有一个帐户持有DEFAULT_ADMIN_ROLE，直到可能被放弃。

* 强制执行将DEFAULT_ADMIN_ROLE转移到另一个帐户的两步过程。

* 在两步之间强制执行一个可配置的延迟，有能力在接受转移之前取消。

* 可以通过调度更改延迟，参见*changeDefaultAdminDelay*。

* 不可能使用另一个角色来管理DEFAULT_ADMIN_ROLE。

示例用法:
```
contract MyToken is AccessControlDefaultAdminRules {
  constructor() AccessControlDefaultAdminRules(
    3 days,
    msg.sender // Explicit initial `DEFAULT_ADMIN_ROLE` holder
   ) {}
}
```

**FUNCTIONS**
constructor(initialDelay, initialDefaultAdmin)

supportsInterface(interfaceId)

owner()

grantRole(role, account)

revokeRole(role, account)

renounceRole(role, account)

_grantRole(role, account)

_revokeRole(role, account)

_setRoleAdmin(role, adminRole)

defaultAdmin()

pendingDefaultAdmin()

defaultAdminDelay()

pendingDefaultAdminDelay()

defaultAdminDelayIncreaseWait()

beginDefaultAdminTransfer(newAdmin)

_beginDefaultAdminTransfer(newAdmin)

cancelDefaultAdminTransfer()

_cancelDefaultAdminTransfer()

acceptDefaultAdminTransfer()

_acceptDefaultAdminTransfer()

changeDefaultAdminDelay(newDelay)

_changeDefaultAdminDelay(newDelay)

rollbackDefaultAdminDelay()

_rollbackDefaultAdminDelay()

_delayChangeWait(newDelay)

ACCESSCONTROL
hasRole(role, account)

_checkRole(role)

_checkRole(role, account)

getRoleAdmin(role)

DEFAULT_ADMIN_ROLE()

**EVENTS**
IACCESSCONTROLDEFAULTADMINRULES
DefaultAdminTransferScheduled(newAdmin, acceptSchedule)

DefaultAdminTransferCanceled()

DefaultAdminDelayChangeScheduled(newDelay, effectSchedule)

DefaultAdminDelayChangeCanceled()

IACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

**ERRORS**
IACCESSCONTROLDEFAULTADMINRULES
AccessControlInvalidDefaultAdmin(defaultAdmin)

AccessControlEnforcedDefaultAdminRules()

AccessControlEnforcedDefaultAdminDelay(schedule)

IACCESSCONTROL
AccessControlUnauthorizedAccount(account, neededRole)

AccessControlBadConfirmation()

#### constructor(uint48 initialDelay, address initialDefaultAdmin)
internal#
设置*defaultAdminDelay*和*defaultAdmin*地址的初始值。

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查阅 *IERC165.supportsInterface*.

#### owner() → address
public#
请查阅 *IERC5313.owner*.

#### grantRole(bytes32 role, address account)
public#
请查阅*AccessControl.grantRole*。对于DEFAULT_ADMIN_ROLE，进行还原。

#### revokeRole(bytes32 role, address account)
public#
请查阅*AccessControl.revokeRole*。对于DEFAULT_ADMIN_ROLE，进行还原。

#### renounceRole(bytes32 role, address account)
public#
请查阅*AccessControl.renounceRole*。

对于DEFAULT_ADMIN_ROLE，它只允许通过首先调用*beginDefaultAdminTransfer*到地址（0）来放弃，因此在调用此函数时，还需要通过*pendingDefaultAdmin*计划。

执行后，将无法调用onlyRole(DEFAULT_ADMIN_ROLE)函数。

> NOTE
放弃DEFAULT_ADMIN_ROLE将使合约没有*默认管理员*，从而禁用仅对其可用的任何功能，并且无法重新分配非管理员角色。

#### _grantRole(bytes32 role, address account) → bool
internal#
请查阅*AccessControl._grantRole*。

对于DEFAULT_ADMIN_ROLE，只有在还没有*defaultAdmin*或者该角色已经被放弃的情况下，才允许授予。

> NOTE
通过另一种机制公开这个函数可能会使DEFAULT_ADMIN_ROLE再次可分配。在你的实现中，确保这是预期的行为。

#### _revokeRole(bytes32 role, address account) → bool
internal#
请查阅 *AccessControl._revokeRole*.

#### _setRoleAdmin(bytes32 role, bytes32 adminRole)
internal#
请查阅* AccessControl._setRoleAdmin*。对于DEFAULT_ADMIN_ROLE，进行还原。

#### defaultAdmin() → address
public#
返回当前DEFAULT_ADMIN_ROLE持有者的地址。

#### pendingDefaultAdmin() → address newAdmin, uint48 schedule
public#
返回一个新管理员和接受计划的元组。

在计划通过后，新管理员将能够通过调用*acceptDefaultAdminTransfer*来接受*默认管理员*角色，完成角色转移。

只在接受计划中的零值表示没有待处理的管理员转移。

> NOTE
零地址的新管理员意味着正在放弃默认管理员。

#### defaultAdminDelay() → uint48
public#
返回开始*默认管理员*转移所需的延迟。

在调用*beginDefaultAdminTransfer*设置接受计划时，此延迟将添加到当前时间戳。

> NOTE
如果已经安排了延迟更改，它将在计划通过后立即生效，使此函数返回新的延迟。参见*changeDefaultAdminDelay*。

#### pendingDefaultAdminDelay() → uint48 newDelay, uint48 schedule
public#
返回一个新延迟和效果计划的元组。

计划过后，新的延迟将立即对*每个开始的默认管理员转移生效*。

只在效果计划中指示零值表示没有待处理的延迟更改。

> NOTE
只对新延迟的零值意味着在效果计划后，下一个*默认管理员延迟*将为零。

#### defaultAdminDelayIncreaseWait() → uint48
public#
*默认管理员延迟*增加的最大时间（以秒为单位），该增加是通过*changeDefaultAdminDelay*进行调度的。默认为5天。

当*默认的管理员延迟*被计划增加时，它会在新的延迟过去后生效，目的是给出足够的时间来撤销任何可能锁定合约的意外更改（例如，使用毫秒而不是秒）。然而，为了避免过多的调度，这个等待时间是由这个函数限制的，它可以被覆盖以自定义*默认管理员延迟*增加的调度。

> IMPORTANT
在覆盖这个值时，确保添加合理的时间，否则，有设置一个几乎立即生效的新高延迟的风险，而没有人工干预的可能性，以防输入错误（例如，设置毫秒而不是秒）。

#### beginDefaultAdminTransfer(address newAdmin)
public#
启动一个*默认管理员*转移，通过设置一个等待默认管理员接受的*pendingDefaultAdmin*，该pendingDefaultAdmin的时间设定为当前时间戳加上一个*defaultAdminDelay*。

要求：
* 只能由当前的*默认管理员*调用。

触发一个DefaultAdminRoleChangeStarted事件。

#### _beginDefaultAdminTransfer(address newAdmin)
internal#
请查阅 *beginDefaultAdminTransfer*.

内部函数没有访问限制。

#### cancelDefaultAdminTransfer()
public#
取消先前使用*beginDefaultAdminTransfer*开始的*默认管理员*转移。

还可以使用此功能取消尚未接受的*待处理默认管理员*。

要求：
* 只能由当前的*默认管理员*调用。

可能会发出DefaultAdminTransferCanceled事件。

#### _cancelDefaultAdminTransfer()
internal#
请查阅 *cancelDefaultAdminTransfer*.
内部函数，无访问限制。

#### acceptDefaultAdminTransfer()
public#
完成先前通过*beginDefaultAdminTransfer*开始的*默认管理员*转移。

调用此函数后：

* DEFAULT_ADMIN_ROLE应授予给调用者。

* DEFAULT_ADMIN_ROLE应从前持有者那里撤销。

* *pendingDefaultAdmin*应重置为零值。

要求：

* 只能由*pendingDefaultAdmin*的newAdmin调用。

* *pendingDefaultAdmin*的接受计划应已通过。

#### _acceptDefaultAdminTransfer()
internal#
请查阅 *acceptDefaultAdminTransfer*.
内部函数，无访问限制。

#### changeDefaultAdminDelay(uint48 newDelay)
public#
此函数通过设置一个*待处理的默认管理员*延迟来启动*默认管理员*延迟的更新，该延迟将在当前时间戳加上*默认管理员*延迟后生效。

该函数保证在此方法被调用的时间戳和*待处理的默认管理员延迟生效计划*之间进行的任何对*beginDefaultAdminTransfer*的调用都将使用在调用之前设置的当*前默认管理员延迟*。

*待处理的默认管理员延迟的生效计划*是这样定义的：等待直到计划然后用新的延迟调用*beginDefaultAdminTransfer*将至少需要与另一个*默认管理员*完整转移（包括接受）一样的时间。

计划设计有两种情况：

* 当延迟被改为更长的时候，计划是block.timestamp + newDelay，上限是*defaultAdminDelayIncreaseWait*。

* 当延迟被改为更短的时候，计划是block.timestamp + (当前延迟 - 新延迟)。

一个从未生效的*待处理的默认管理员延迟*将被取消，以便进行新的计划更改。

要求：
* 只能由当前的*默认管理员*调用。

会发出一个DefaultAdminDelayChangeScheduled事件，可能会发出一个DefaultAdminDelayChangeCanceled事件。

#### _changeDefaultAdminDelay(uint48 newDelay)
_changeDefaultAdminDelay(uint48 newDelay)
internal#
请查阅 *changeDefaultAdminDelay*.
内部函数，无访问限制。

#### rollbackDefaultAdminDelay()
public#
取消预定的*defaultAdminDelay*更改。

要求：
* 只能由当前的*defaultAdmin*调用。

可能会触发一个DefaultAdminDelayChangeCanceled事件。

#### _rollbackDefaultAdminDelay()
internal#
请查阅 *rollbackDefaultAdminDelay*.
内部函数，无访问限制。

#### _delayChangeWait(uint48 newDelay) → uint48
internal#
返回在newDelay成为新的*defaultAdminDelay*后需要等待的秒数。

返回的值保证，如果延迟减少，它将在尊重之前设置的延迟后生效。

参见*defaultAdminDelayIncreaseWait*。

## AccessManager

### [IAuthority](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/manager/IAuthority.sol)

```
import "@openzeppelin/contracts/access/manager/IAuthority.sol";
```

最初在Dappsys中定义的权限设置的标准接口。

**FUNCTIONS**
canCall(caller, target, selector)

#### canCall(address caller, address target, bytes4 selector) → bool allowed
external#
如果调用者可以在目标上调用由函数选择器标识的函数，则返回true。

#### [IAccessManager](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/manager/IAccessManager.sol)
```
import "@openzeppelin/contracts/access/manager/IAccessManager.sol";
```

**FUNCTIONS**
canCall(caller, target, selector)

expiration()

minSetback()

isTargetClosed(target)

getTargetFunctionRole(target, selector)

getTargetAdminDelay(target)

getRoleAdmin(roleId)

getRoleGuardian(roleId)

getRoleGrantDelay(roleId)

getAccess(roleId, account)

hasRole(roleId, account)

labelRole(roleId, label)

grantRole(roleId, account, executionDelay)

revokeRole(roleId, account)

renounceRole(roleId, callerConfirmation)

setRoleAdmin(roleId, admin)

setRoleGuardian(roleId, guardian)

setGrantDelay(roleId, newDelay)

setTargetFunctionRole(target, selectors, roleId)

setTargetAdminDelay(target, newDelay)

setTargetClosed(target, closed)

getSchedule(id)

getNonce(id)

schedule(target, data, when)

execute(target, data)

cancel(caller, target, data)

consumeScheduledOp(caller, data)

hashOperation(caller, target, data)

updateAuthority(target, newAuthority)

**EVENTS**
OperationScheduled(operationId, nonce, schedule, caller, target, data)

OperationExecuted(operationId, nonce)

OperationCanceled(operationId, nonce)

RoleLabel(roleId, label)

RoleGranted(roleId, account, delay, since, newMember)

RoleRevoked(roleId, account)

RoleAdminChanged(roleId, admin)

RoleGuardianChanged(roleId, guardian)

RoleGrantDelayChanged(roleId, delay, since)

TargetClosed(target, closed)

TargetFunctionRoleUpdated(target, selector, roleId)

TargetAdminDelayUpdated(target, delay, since)

**ERRORS**
AccessManagerAlreadyScheduled(operationId)

AccessManagerNotScheduled(operationId)

AccessManagerNotReady(operationId)

AccessManagerExpired(operationId)

AccessManagerLockedAccount(account)

AccessManagerLockedRole(roleId)

AccessManagerBadConfirmation()

AccessManagerUnauthorizedAccount(msgsender, roleId)

AccessManagerUnauthorizedCall(caller, target, selector)

AccessManagerUnauthorizedConsume(target)

AccessManagerUnauthorizedCancel(msgsender, caller, target, selector)

AccessManagerInvalidInitialAdmin(initialAdmin)

#### canCall(address caller, address target, bytes4 selector) → bool allowed, uint32 delay
external#
检查一个地址（调用者）是否被授权直接调用给定合约上的某个函数（无任何限制）。此外，它还返回通过*调度*和*执行*工作流间接执行调用所需的延迟。

这个函数通常由目标合约调用，以控制受限函数的即时执行。因此，我们只在可以无延迟执行调用时返回true。如果调用受到之前设置的延迟（非零）的影响，那么函数应返回false，调用者应将操作安排在未来执行。

如果immediate为true，可以忽略延迟，操作可以立即执行，否则操作只能在延迟大于0的情况下执行。

> NOTE
IAuthority接口不包括uint32延迟。这是该接口的一个向后兼容的扩展。因此，一些合约可能会忽略第二个返回参数。在这种情况下，它们将无法识别间接工作流，并将认为需要延迟的调用是被禁止的。

> NOTE
此函数不报告此管理器本身的权限。这些权限由{_canCallSelf}函数定义。

#### expiration() → uint32
external#
计划提案的过期延迟。默认为1周。

> IMPORTANT
避免将过期时间设置为0。否则，每个合约提案将立即过期，使得任何计划使用都无效。

#### minSetback() → uint32
external#
除执行延迟外，所有延迟更新的最小回退。它可以在没有回退的情况下增加（并在意外增加的情况事件中通过*撤销角色*重置）。默认为5天。

#### isTargetClosed(address target) → bool
external#
判断合约是否已关闭并禁止任何访问。否则，将应用角色权限。

#### getTargetFunctionRole(address target, bytes4 selector) → uint64
external#
调用函数所需的角色。

#### getTargetAdminDelay(address target) → uint32
external#
获取目标合约的管理员延迟。对合约配置的更改将受到此延迟的影响。

#### getRoleAdmin(uint64 roleId) → uint64
external#
获取作为给定角色的管理员的角色的ID。

管理员权限是授予角色，撤销角色和更新执行延迟以执行仅限于此角色的操作所必需的。

#### getRoleGuardian(uint64 roleId) → uint64
external#
获取作为给定角色的保护者的角色。

保护者权限允许取消已经在角色下安排的操作。

#### getRoleGrantDelay(uint64 roleId) → uint32
external#
获取当前角色授权延迟。

此值可能在任何时候改变，而不会在调用*8*后发出事件。此值的更改，包括生效时间点，将通过*RoleGrantDelayChanged*事件提前通知。

#### getAccess(uint64 roleId, address account) → uint48, uint32, uint32, uint48
external#
获取给定角色的给定账户的访问详情。这些详情包括会员资格生效的时间点，以及对需要此权限级别的用户的所有操作应用的延迟。

返回：[0] 账户会员资格生效的时间戳。0表示未授予角色。[1] 账户当前的执行延迟。[2] 账户的待处理执行延迟。[3] 待处理执行延迟将生效的时间戳。0表示没有计划的延迟更新。

#### hasRole(uint64 roleId, address account) → bool, uint32
external#
检查给定账户当前是否具有对应于给定角色的权限级别。请注意，这种权限可能与执行延迟相关联。*getAccess*可以提供更多详细信息。

#### labelRole(uint64 roleId, string label)
external#
为角色添加标签，以便UI更好地发现角色。

要求：
* 调用者必须是全局管理员

触发一个*RoleLabel*事件。

#### grantRole(uint64 roleId, address account, uint32 executionDelay)
external#
将账户添加到角色ID，或更改其执行延迟。

这将授权账户调用任何限制为此角色的功能。可以设置一个可选的执行延迟（以秒为单位）。如果该延迟不为0，用户需要预定任何限制为此角色成员的操作。用户只能在延迟过后，且在其过期之前执行操作。在此期间，管理员和监护人可以取消操作（参见*cancel*）。

如果账户已经被授予了这个角色，执行延迟将被更新。这个更新不是立即的，而是遵循延迟规则。例如，如果一个用户当前的延迟是3小时，然后调用这个来将延迟减少到1小时，新的延迟将需要一些时间才能生效，确保在此更新之后的3小时内执行的任何操作确实是在此更新之前预定的。

要求：
* 调用者必须是角色的管理员（参见*getRoleAdmin*）

授予的角色不能是PUBLIC_ROLE

触发一个*RoleGranted*事件。

#### revokeRole(uint64 roleId, address account)
external#
从角色中移除一个账户，立即生效。如果账户没有该角色，此调用无效。

要求：
* 调用者必须是角色的管理员（参见*getRoleAdmin*）

* 被撤销的角色不能是PUBLIC_ROLE

如果账户拥有该角色，会触发一个*RoleRevoked*事件。

#### renounceRole(uint64 roleId, address callerConfirmation)
external#
立即撤销呼叫账户的角色权限。如果发送者不在此角色中，此调用无效。

要求：
* 调用者必须是callerConfirmation。

如果账户拥有角色，将触发*RoleRevoked*事件。

#### setRoleAdmin(uint64 roleId, uint64 admin)
external#
更改给定角色的管理员角色。

要求：

调用者必须是全局管理员

触发一个*RoleAdminChanged*事件

#### setRoleGuardian(uint64 roleId, uint64 guardian)
external#
更改给定角色保护者角色。

需求：

调用者必须是全局管理员

触发一个*RoleGuardianChanged*事件

#### setGrantDelay(uint64 roleId, uint32 newDelay)
external#
更新授予角色ID的延迟。

要求：
* 调用者必须是全局管理员

触发一个*RoleGrantDelayChanged*事件。

#### setTargetFunctionRole(address target, bytes4[] selectors, uint64 roleId)
external#
设置调用目标合约中由选择器标识的函数所需的角色。

要求：
* 调用者必须是全局管理员

每个选择器都会触发一个*TargetFunctionRoleUpdated*事件。

#### setTargetAdminDelay(address target, uint32 newDelay)
external#
设置更改给定目标合约配置的延迟。

要求：
* 调用者必须是全局管理员

触发一个*TargetAdminDelayUpdated*事件。

#### setTargetClosed(address target, bool closed)
external#
为合约设置关闭标志。

需求：
* 调用者必须是全局管理员

触发一个*TargetClosed*事件。

#### getSchedule(bytes32 id) → uint48
external#
返回计划操作准备执行的时间点。如果操作尚未计划，已过期，已执行或已取消，则返回0。

#### getNonce(bytes32 id) → uint32
external#
返回给定id的最新计划操作的随机数。如果操作从未被计划过，则返回0。

#### schedule(address target, bytes data, uint48 when) → bytes32, uint32
external#
安排一个延迟操作在未来执行，并返回操作标识符。只要满足调用者所需的执行延迟，就可以选择操作可执行的时间戳。特殊值零将自动设置最早可能的时间。

返回被安排的操作ID。由于此值是参数的哈希，因此当使用相同的参数时，它可以重复出现；如果这是相关的，返回的随机数可以用来唯一地识别这个安排的操作，从其他在*执行*和*取消调用*中的相同操作ID的发生中。

发出一个*OperationScheduled*事件。

> NOTE
不可能同时安排具有相同目标和数据的多个操作。如果这是必要的，可以在数据后面附加一个随机字节作为盐，如果目标合约使用标准的Solidity ABI编码，它将被忽略。

#### execute(address target, bytes data) → uint32
external#
执行一个受到延迟限制的函数，前提是它之前已经被正确地安排好，或者执行延迟为0。

返回标识之前安排的操作的nonce，如果操作之前没有被安排（如果调用者没有执行延迟），则返回0。

只有在调用被安排并延迟时，才会发出一个*OperationExecuted*事件。

#### cancel(address caller, address target, bytes data) → uint32
external#
取消预定（延迟）的操作。返回标识被取消的预定操作的随机数。

要求：
* 调用者必须是提议者，目标函数的监护人，或全局管理员

触发一个*OperationCanceled*事件。

#### consumeScheduledOp(address caller, bytes data)
external#
消耗针对调用者的预定操作。如果存在这样的操作，将其标记为已消耗（发出*OperationExecuted*事件并清理状态）。否则，抛出错误。

这对于想要强制在管理器上预定针对它们的调用的合约非常有用，这涉及到所有的验证。

发出一个*OperationExecuted*事件。

#### hashOperation(address caller, address target, bytes data) → bytes32
external#
为延迟操作的哈希函数

#### updateAuthority(address target, address newAuthority)
external#
更改由此管理器实例管理的目标的权限。

要求：
* 调用者必须是全局管理员
  
#### OperationScheduled(bytes32 indexed operationId, uint32 indexed nonce, uint48 schedule, address caller, address target, bytes data)
event#
计划了一个延迟的操作。

#### OperationExecuted(bytes32 indexed operationId, uint32 indexed nonce)
event#
一个预定的操作已经执行。

#### OperationCanceled(bytes32 indexed operationId, uint32 indexed nonce)
event#
计划的操作被取消了。

#### RoleLabel(uint64 indexed roleId, string label)
event#
为角色ID提供的信息标签

#### RoleGranted(uint64 indexed roleId, address indexed account, uint32 delay, uint48 since, bool newMember)
event#
当账户被授予roleId时发出。

> NOTE
since参数的含义取决于newMember参数。如果角色被授予给新成员，since参数表示账户成为角色成员的时间，否则它表示此账户和roleId的执行延迟更新。

#### RoleRevoked(uint64 indexed roleId, address indexed account)
event#
当账户成员资格或角色ID被撤销时发出。与授予权限不同，撤销是即时的。

#### RoleAdminChanged(uint64 indexed roleId, uint64 indexed admin)
event#
作为给定角色ID的管理员的角色已更新。

#### RoleGuardianChanged(uint64 indexed roleId, uint64 indexed guardian)
event#
更新作为指定角色ID的监护人的角色。

#### RoleGrantDelayChanged(uint64 indexed roleId, uint32 delay, uint48 since)
event#
当达到指定的时间时，将会更新给定角色ID的授权延迟。

#### TargetClosed(address indexed target, bool closed)
event#
目标模式已更新（true=关闭，false=开启）。

#### TargetFunctionRoleUpdated(address indexed target, bytes4 selector, uint64 indexed roleId)
event#
需要更新为roleId以在目标上调用选择器的角色。

#### TargetAdminDelayUpdated(address indexed target, uint32 delay, uint48 since)
event#
当达到给定的时间点时，将会更新给定目标的管理员延迟。

#### AccessManagerAlreadyScheduled(bytes32 operationId)
error#

#### AccessManagerNotScheduled(bytes32 operationId)
error#

#### AccessManagerNotReady(bytes32 operationId)
error#

#### AccessManagerExpired(bytes32 operationId)
error#

#### AccessManagerLockedAccount(address account)
error#

#### AccessManagerLockedRole(uint64 roleId)
error#

#### AccessManagerBadConfirmation()
error#

#### AccessManagerUnauthorizedAccount(address msgsender, uint64 roleId)
error#

#### AccessManagerUnauthorizedCall(address caller, address target, bytes4 selector)
error#

#### AccessManagerUnauthorizedConsume(address target)
error#

#### AccessManagerUnauthorizedCancel(address msgsender, address caller, address target, bytes4 selector)
error#

#### AccessManagerInvalidInitialAdmin(address initialAdmin)
error#

### [AccessManager](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/manager/AccessManager.sol)
```
import "@openzeppelin/contracts/access/manager/AccessManager.sol";
```
AccessManager是一个中心合约，用于存储系统的权限。

一个由AccessManager实例控制的智能合约被称为目标，它将继承*AccessManaged*合约，作为其管理器与此合约连接，并在选定要进行权限管理的一组函数上实施*AccessManaged.restricted*修饰符。请注意，没有这种设置的任何函数都不会被有效地限制。

这些函数的限制规则是以"角色"的形式定义的，这些角色由uint64标识，并由目标（地址）和函数选择器（bytes4）限定。这些角色存储在此合约中，可以由管理员（ADMIN_ROLE成员）在延迟后配置（参见*getTargetAdminDelay*）。

对于每个目标合约，管理员可以立即配置以下内容：
* 通过*updateAuthority*配置目标的*AccessManaged.authority*。

* 通过*setTargetClosed*关闭或打开目标，同时保持权限不变。

* 通过*setTargetFunctionRole*配置允许（或不允许）调用给定函数（由其选择器标识）的角色。

默认情况下，每个地址都是PUBLIC_ROLE的成员，每个目标函数都限制为ADMIN_ROLE，直到配置为其他方式。此外，每个角色都有以下配置选项，限制为此管理器的管理员：

* 可以授予或撤销角色的角色的管理员角色，通过*setRoleAdmin*设置。

* 允许取消操作的角色的监护人角色，通过*setRoleGuardian*设置。

* 通过*setGrantDelay*设置授予角色后生效的延迟。

* 通过*setTargetAdminDelay*设置任何目标的管理员操作的延迟。

* 用于发现目的的角色标签，通过*labelRole*设置。

任何帐户都可以通过使用*grantRole*和*revokeRole*函数添加和删除这些角色，这些函数受到每个角色的管理员的限制（参见*getRoleAdmin*）。

由于管理系统的所有权限都可以由此实例的管理员修改，因此预计他们将被高度保护（例如，多重签名或配置良好的DAO）。

> NOTE
此合约实现了*IAuthority*接口的一种形式，但是*canCall*有额外的返回数据，所以它不继承IAuthority。然而，它与IAuthority接口兼容，因为返回数据的前32字节是IAuthority接口所期望的布尔值。

> NOTE
实现其他访问控制机制的系统（例如使用*Ownable*）可以通过直接将权限（在*Ownable*的情况下为所有权）转移给*AccessManager*来配对。用户将能够通过*execute*函数与这些合约进行交互，遵循在*AccessManager*中注册的访问规则。请记住，在这种情况下，受限函数看到的msg.sender将是*AccessManager*本身。

> WARNING
在将*Ownable*或*AccessControl*合约的权限授予*AccessManager*时，要非常注意与{*Ownable.renounceOwnership*}或{*AccessControl.renounceRole*}等函数相关的危险。

**MODIFIERS**
onlyAuthorized()

**FUNCTIONS**
constructor(initialAdmin)

canCall(caller, target, selector)

expiration()

minSetback()

isTargetClosed(target)

getTargetFunctionRole(target, selector)

getTargetAdminDelay(target)

getRoleAdmin(roleId)

getRoleGuardian(roleId)

getRoleGrantDelay(roleId)

getAccess(roleId, account)

hasRole(roleId, account)

labelRole(roleId, label)

grantRole(roleId, account, executionDelay)

revokeRole(roleId, account)

renounceRole(roleId, callerConfirmation)

setRoleAdmin(roleId, admin)

setRoleGuardian(roleId, guardian)

setGrantDelay(roleId, newDelay)

_grantRole(roleId, account, grantDelay, executionDelay)

_revokeRole(roleId, account)

_setRoleAdmin(roleId, admin)

_setRoleGuardian(roleId, guardian)

_setGrantDelay(roleId, newDelay)

setTargetFunctionRole(target, selectors, roleId)

_setTargetFunctionRole(target, selector, roleId)

setTargetAdminDelay(target, newDelay)

_setTargetAdminDelay(target, newDelay)

setTargetClosed(target, closed)

_setTargetClosed(target, closed)

getSchedule(id)

getNonce(id)

schedule(target, data, when)

execute(target, data)

cancel(caller, target, data)

consumeScheduledOp(caller, data)

_consumeScheduledOp(operationId)

hashOperation(caller, target, data)

updateAuthority(target, newAuthority)

ADMIN_ROLE()

PUBLIC_ROLE()

MULTICALL
multicall(data)

**EVENTS**
IACCESSMANAGER
OperationScheduled(operationId, nonce, schedule, caller, target, data)

OperationExecuted(operationId, nonce)

OperationCanceled(operationId, nonce)

RoleLabel(roleId, label)

RoleGranted(roleId, account, delay, since, newMember)

RoleRevoked(roleId, account)

RoleAdminChanged(roleId, admin)

RoleGuardianChanged(roleId, guardian)

RoleGrantDelayChanged(roleId, delay, since)

TargetClosed(target, closed)

TargetFunctionRoleUpdated(target, selector, roleId)

TargetAdminDelayUpdated(target, delay, since)

**ERRORS**
IACCESSMANAGER
AccessManagerAlreadyScheduled(operationId)

AccessManagerNotScheduled(operationId)

AccessManagerNotReady(operationId)

AccessManagerExpired(operationId)

AccessManagerLockedAccount(account)

AccessManagerLockedRole(roleId)

AccessManagerBadConfirmation()

AccessManagerUnauthorizedAccount(msgsender, roleId)

AccessManagerUnauthorizedCall(caller, target, selector)

AccessManagerUnauthorizedConsume(target)

AccessManagerUnauthorizedCancel(msgsender, caller, target, selector)

AccessManagerInvalidInitialAdmin(initialAdmin)

#### onlyAuthorized()
modifier#
检查呼叫者是否有权执行操作，遵循在{_getAdminRestrictions}中编码的限制。

#### constructor(address initialAdmin)
public#

#### canCall(address caller, address target, bytes4 selector) → bool immediate, uint32 delay
public#
检查一个地址（调用者）是否被授权直接调用给定合约上的给定函数（无任何限制）。此外，它还返回*执行*和间接*调用*所需的延迟时间。

这个函数通常由目标合约调用，以控制受限函数的即时执行。因此，我们只在可以无延迟执行调用时返回true。如果调用受到之前设置的延迟（非零）的影响，那么函数应返回false，调用者应安排将来执行操作。

如果immediate为true，可以忽略延迟并立即执行操作，否则只有当延迟大于0时才能执行操作。

> NOTE
IAuthority接口不包括uint32延迟。这是该接口的向后兼容扩展。因此，一些合约可能会忽略第二个返回参数。在这种情况下，它们将无法识别间接工作流，并将认为需要延迟的调用是被禁止的。

> NOTE
此函数不报告此管理器本身的权限。这些权限由{_canCallSelf}函数定义。

#### expiration() → uint32
public#
计划提案的到期延迟。默认为1周。

避免将过期时间设置为0。否则，每个合约提案将立即过期，使得任何计划使用都无效。

#### minSetback() → uint32
public#
除执行延迟外，所有延迟更新的最小回退。它可以在没有回退的情况下增加（并在意外增加的情况事件中通过*撤销角色*重置）。默认为5天。

#### isTargetClosed(address target) → bool
public#
判断合约是否已关闭并禁止任何访问。否则，将应用角色权限。

#### getTargetFunctionRole(address target, bytes4 selector) → uint64
public#
调用函数所需的角色。

#### getTargetAdminDelay(address target) → uint32
public#
获取目标合约的管理员延迟。对合约配置的更改将受此延迟的影响。

#### getRoleAdmin(uint64 roleId) → uint64
public#
获取作为给定角色的管理员的角色的ID。

需要管理员权限才能授予角色，撤销角色和更新执行延迟以执行仅限于此角色的操作。

#### getRoleGuardian(uint64 roleId) → uint64
public#
获取作为给定角色的监护人的角色。

监护人权限允许取消已在角色下安排的操作。

#### getRoleGrantDelay(uint64 roleId) → uint32
public#
获取当前角色授权延迟。

此值可能在任何时候改变，而不会在调用*setGrantDelay*后发出事件。对此值的更改，包括生效时间点，都会通过*RoleGrantDelayChanged*事件提前通知。

#### getAccess(uint64 roleId, address account) → uint48 since, uint32 currentDelay, uint32 pendingDelay, uint48 effect
public#
获取给定角色的给定账户的访问详情。这些详情包括会员资格生效的时间点，以及对该用户需要此权限级别的所有操作应用的延迟。

返回：[0] 账户会员资格生效的时间戳。0表示未授予角色。[1] 账户当前的执行延迟。[2] 账户的待处理执行延迟。[3] 待处理执行延迟将生效的时间戳。0表示没有计划的延迟更新。

#### hasRole(uint64 roleId, address account) → bool isMember, uint32 executionDelay
public#
检查给定账户当前是否具有对应于给定角色的权限级别。请注意，此权限可能与执行延迟相关联。*getAccess*可以提供更多详细信息。

#### labelRole(uint64 roleId, string label)
public#
为角色添加标签，以便UI更好地发现角色。

要求：
* 调用者必须是全局管理员

触发一个*RoleLabel*事件。

#### grantRole(uint64 roleId, address account, uint32 executionDelay)
public#
将账户添加到roleId，或更改其执行延迟。

这将授权账户调用任何限制为此角色的功能。可以设置一个可选的执行延迟（以秒为单位）。如果该延迟非0，则要求用户安排任何限制为此角色成员的操作。用户只能在延迟过后，且在其过期之前执行操作。在此期间，管理员和监护人可以取消操作（参见 *cancel*）。

如果账户已经被授予了这个角色，执行延迟将被更新。这个更新不是立即的，而是遵循延迟规则。例如，如果一个用户当前的延迟是3小时，然后调用这个来将延迟减少到1小时，新的延迟将需要一些时间才能生效，确保在这次更新后的3小时内执行的任何操作确实是在这次更新之前安排的。

要求：
* 调用者必须是角色的管理员（参见*getRoleAdmin*）

* 授予的角色不能是PUBLIC_ROLE

会触发一个*RoleGranted*事件。

#### revokeRole(uint64 roleId, address account)
public#
从角色中移除一个账户，立即生效。如果账户没有该角色，此操作无效。

要求：
* 调用者必须是该角色的管理员（参见* *）

* 被撤销的角色不能是PUBLIC_ROLE

如果账户拥有该角色，会触发一个*RoleRevoked*事件。

#### renounceRole(uint64 roleId, address callerConfirmation)
public#
立即放弃调用账户的角色权限。如果发送者不在该角色中，此调用无效。

要求：
* 调用者必须是callerConfirmation。

如果账户拥有角色，将触发*RoleRevoked*事件。

#### setRoleAdmin(uint64 roleId, uint64 admin)
public#
更改给定角色的管理员角色。

要求：
* 调用者必须是全局管理员

触发一个*RoleAdminChanged*事件

#### setRoleGuardian(uint64 roleId, uint64 guardian)
public#
更改给定角色的监护人角色。

要求：
* 调用者必须是全局管理员

触发一个*RoleGuardianChanged*事件

#### setGrantDelay(uint64 roleId, uint32 newDelay)
public#
更新授予角色ID的延迟。

要求：
* 调用者必须是全局管理员

触发一个*RoleGrantDelayChanged*事件。

#### _grantRole(uint64 roleId, address account, uint32 grantDelay, uint32 executionDelay) → bool
internal#
内部版本的*grantRole*没有访问控制。如果角色是新授予的，则返回true。

触发一个*RoleGranted*事件。

#### _revokeRole(uint64 roleId, address account) → bool
internal#
无访问控制的*撤销角色*的内部版本。这个逻辑也被*放弃角色*使用。如果之前授予了角色，返回true。

如果账户拥有角色，会发出一个*RoleRevoked*事件。

#### _setRoleAdmin(uint64 roleId, uint64 admin)
internal#
内部版本的*setRoleAdmin*没有访问控制。

会发出一个*RoleAdminChanged*事件。

> NOTE
允许将管理员角色设置为PUBLIC_ROLE，但这实际上会允许任何人设置或撤销此类角色。

#### _setRoleGuardian(uint64 roleId, uint64 guardian)
internal#
内部版本的*setRoleGuardian*函数，没有访问控制。

触发一个*RoleGuardianChanged*事件。

> NOTE
将监护人角色设置为PUBLIC_ROLE是允许的，但实际上它会允许任何人取消此类角色的任何计划操作。

#### _setGrantDelay(uint64 roleId, uint32 newDelay)
internal#
内部版本的*setGrantDelay*函数，没有访问控制。

触发一个*RoleGrantDelayChanged*事件。

#### setTargetFunctionRole(address target, bytes4[] selectors, uint64 roleId)
public#
设置需要调用目标合约中由选择器标识的函数的角色。

要求：
* 调用者必须是全局管理员

每个选择器会触发一个*TargetFunctionRoleUpdated*事件。

#### _setTargetFunctionRole(address target, bytes4 selector, uint64 roleId)
internal#
内部版本的*setTargetFunctionRole*函数，没有访问控制。

发出一个*TargetFunctionRoleUpdated*事件。

#### setTargetAdminDelay(address target, uint32 newDelay)
public#
设置更改给定目标合约配置的延迟。

要求：
* 调用者必须是全局管理员

会触发一个*TargetAdminDelayUpdated*事件。

#### _setTargetAdminDelay(address target, uint32 newDelay)
internal#
内部版本的*setTargetAdminDelay*，没有访问控制。

发出一个*TargetAdminDelayUpdated*事件。

#### setTargetClosed(address target, bool closed)
public#
为合约设置关闭标志。

要求：
* 调用者必须是全局管理员

触发一个*TargetClosed*事件。

#### _setTargetClosed(address target, bool closed)
internal#
为合约设置关闭标志。这是一个没有访问限制的内部设置器。

发出一个*TargetClosed*事件。

#### getSchedule(bytes32 id) → uint48
public#
返回计划操作准备执行的时间点。如果操作尚未计划，已过期，已执行或已取消，则返回0。

#### getNonce(bytes32 id) → uint32
public#
返回具有给定ID的最新计划操作的nonce。如果操作从未被计划过，则返回0。

#### schedule(address target, bytes data, uint48 when) → bytes32 operationId, uint32 nonce
public#
为未来的执行安排一个延迟操作，并返回操作标识符。只要满足调用者所需的执行延迟，就可以选择操作变为可执行的时间戳。特殊值零将自动设置为最早可能的时间。

返回被安排的操作Id。由于此值是参数的哈希，因此当使用相同的参数时，它可以重新出现；如果这是相关的，返回的nonce可以用来唯一地识别这个安排的操作，从其他在*执行*和*取消调用*中的相同操作Id的出现中区分出来。

发出一个*OperationScheduled*事件。

> NOTE
不可能同时安排具有相同目标和数据的多个操作。如果这是必要的，可以在数据后面附加一个随机字节，作为一个盐，如果目标合约使用标准的Solidity ABI编码，它将被忽略。

#### execute(address target, bytes data) → uint32
public#
执行一个受到延迟限制的函数，前提是它之前已经被正确地安排过，或者执行延迟为0。

返回标识之前安排的操作的nonce，如果操作之前没有被安排（如果调用者没有执行延迟）则返回0。

只有在调用被安排并延迟时，才会触发一个*OperationExecuted*事件。

#### cancel(address caller, address target, bytes data) → uint32
public#
取消预定（延迟）的操作。返回标识被取消的预定操作的随机数。

要求：

调用者必须是提议者，目标函数的监护人，或全局管理员

触发一个*OperationCanceled*事件。

#### consumeScheduledOp(address caller, bytes data)
public#
消费针对调用者的预定操作。如果此类操作存在，将其标记为已消费（发出*OperationExecuted*事件并清理状态）。否则，抛出错误。

这对于希望强制在管理器上预定针对它们的调用的合约非常有用，这涉及所有的验证。

发出一个*OperationExecuted*事件。

#### _consumeScheduledOp(bytes32 operationId) → uint32
internal#
内部变体的*consumeScheduledOp*，该变体在bytes32 operationId上操作。

返回被消耗的计划操作的nonce。

#### hashOperation(address caller, address target, bytes data) → bytes32
public#
为延迟操作的哈希函数。

#### updateAuthority(address target, address newAuthority)
public#
更改此管理实例管理的目标的权限。

要求：
* 调用者必须是全局管理员

#### ADMIN_ROLE() → uint64
public#

#### PUBLIC_ROLE() → uint64
public#

### [IAccessManaged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/manager/IAccessManaged.sol)
```
import "@openzeppelin/contracts/access/manager/IAccessManaged.sol";
```

**FUNCTIONS**
authority()

setAuthority()

isConsumingScheduledOp()

**EVENTS**
AuthorityUpdated(authority)

**ERRORS**
AccessManagedUnauthorized(caller)

AccessManagedRequiredDelay(caller, delay)

AccessManagedInvalidAuthority(authority)

#### authority() → address
external#
返回当前权限。

#### setAuthority(address)
external#
将控制权转移给新的管理员。呼叫者必须是当前的管理员。

#### isConsumingScheduledOp() → bytes4
external#
仅在延迟限制调用的上下文中返回真，即在计划操作被消耗的那一刻。防止在合约执行攻击者控制的调用的情况下，对延迟限制调用的服务拒绝。

#### AuthorityUpdated(address authority)
event#
管理此合约的权力机构已更新。

#### AccessManagedUnauthorized(address caller)
error#

#### AccessManagedRequiredDelay(address caller, uint32 delay)
error#

#### AccessManagedInvalidAuthority(address authority)
error#

### [AccessManaged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/manager/AccessManaged.sol)
```
import "@openzeppelin/contracts/access/manager/AccessManaged.sol";
```

此合约模块提供了一个*restricted*修饰符。使用此修饰符修饰的函数将根据"权限"进行权限分配：像*AccessManager*这样的合约遵循*IAuthority*接口，实现一种策略，允许某些调用者访问某些函数。

> IMPORTANT
受限制的修饰符永远不应该用在内部函数上，应谨慎地用在公共函数上，理想情况下只用在外部函数上。参见*restricted*。

**MODIFIERS**
restricted()

**FUNCTIONS**
constructor(initialAuthority)

authority()

setAuthority(newAuthority)

isConsumingScheduledOp()

_setAuthority(newAuthority)

_checkCanCall(caller, data)

**EVENTS**
IACCESSMANAGED
AuthorityUpdated(authority)

**ERRORS**
IACCESSMANAGED
AccessManagedUnauthorized(caller)

AccessManagedRequiredDelay(caller, delay)

AccessManagedInvalidAuthority(authority)

#### restricted()
modifier#
此修饰符限制了对函数的访问，这是由此合约的连接权限、调用者和进入合约的函数的选择器定义的。

> IMPORTANT
通常，这个修饰符只应用于外部函数。在作为外部入口点使用并且不被内部调用的公共函数上使用它是可以的。除非你知道你在做什么，否则永远不应该在内部函数上使用它。不遵循这些规则可能会有严重的安全影响！这是因为权限是由进入合约的函数，即调用堆栈底部的函数，而不是源代码中可见修饰符的函数来确定的。

> WARNING
避免在*receive()*函数或*fallback()*函数中添加此修饰符。这些函数是唯一的执行路径，其中函数选择器不能从calldata中明确确定，因为在receive()函数中选择器默认为0x00000000，如果没有提供calldata，fallback()函数也是如此。（参见*_checkCanCall*）。

receive()函数将始终出现panic，而fallback()函数可能会根据calldata的长度出现panic。

#### constructor(address initialAuthority)
internal#

#### authority() → address
public#
返回当前权限。

#### setAuthority(address newAuthority)
public#
将控制权转移给新的权威。调用者必须是当前的权威。

#### isConsumingScheduledOp() → bytes4
public#
仅在延迟限制呼叫的上下文中返回真值，即在计划操作被消耗的时刻。防止在合约执行攻击者控制的呼叫的情况下，对延迟限制呼叫的服务拒绝。

#### _setAuthority(address newAuthority)
internal#
将控制权转移给新的权儿。内部功能，没有访问限制。允许绕过当前权儿设置的权限。

#### _checkCanCall(address caller, bytes data)
internal#
如果调用者不被允许调用由选择器标识的函数，则撤销。如果调用数据少于4字节，则会引发恐慌。

### [AuthorityUtils](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/access/manager/AuthorityUtils.sol)
```
import "@openzeppelin/contracts/access/manager/AuthorityUtils.sol";
```

**FUNCTIONS**
canCallWithDelay(authority, caller, target, selector)

#### canCallWithDelay(address authority, address caller, address target, bytes4 selector) → bool immediate, uint32 delay
internal#
由于AccessManager实现了扩展的IAuthority接口，以向后兼容的方式调用canCall需要特别注意，以避免因返回数据不足而回退。这个辅助函数负责以向后兼容的方式调用canCall，而不会回退。