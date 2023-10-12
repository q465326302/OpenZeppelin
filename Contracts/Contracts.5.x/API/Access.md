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