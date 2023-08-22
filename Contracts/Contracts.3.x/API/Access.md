# Access
这个目录提供了限制谁可以访问合约功能以及何时可以访问的方法。

* *AccessControl*提供了一种基于角色的访问控制机制。可以创建多个层级角色，并将每个角色分配给多个账户。

* *Ownable*是一种更简单的机制，只有一个所有者“角色”，可以分配给单个账户。这种更简单的机制对于快速测试可能很有用，但对于有生产问题的项目可能会不够用。

* *TimelockController*与上述两种机制结合使用。通过将角色分配给TimelockController合约的实例，受该角色控制的功能的访问将被延迟一段时间。

## 授权
### Ownable
这是一个合约模块，提供了基本的访问控制机制，其中有一个账户（所有者）可以被授予对特定功能的独占访问权限。

默认情况下，所有者账户将是部署合约的账户。可以使用*transferOwnership*函数将其更改。

通过继承使用此模块。它将提供修饰符onlyOwner，可以应用于您的函数，以限制它们只能被所有者使用。

**MODIFIERS**
onlyOwner()

**FUNCTIONS**
constructor()

owner()

renounceOwnership()

transferOwnership(newOwner)

**EVENTS**
OwnershipTransferred(previousOwner, newOwner)

#### onlyOwner()
修饰符#
如果被除了所有者之外的任何账户调用，则会抛出异常。

#### constructor()
内部#
初始化合约，将部署者设为初始所有者。

#### owner() → address
公开#
返回当前所有者的地址。

#### renounceOwnership()
公开#
没有所有者的合约将被废弃。将无法再调用只能由所有者调用的函数。只能由当前所有者调用。

> NOTE
放弃所有权将使合约失去所有者，从而删除只有所有者才能使用的功能。

#### transferOwnership(address newOwner)
公开#
将合约的所有权转移到一个新账户（newOwner）。只能由当前所有者调用。

#### OwnershipTransferred(address previousOwner, address newOwner)
事件#

### AccessControl
允许children实施基于角色的访问控制机制的合约模块。

角色由其bytes32标识符引用。这些标识符应在外部API中公开，并且应该是唯一的。实现这一点的最佳方法是使用公共常量哈希摘要：
```
bytes32 public constant MY_ROLE = keccak256("MY_ROLE");
```

角色可以用来表示一组权限。要限制对函数调用的访问，请使用*hasRole*。

```
function foo() public {
    require(hasRole(MY_ROLE, msg.sender));
    ...
}
```
角色可以通过*grantRole*和*revokeRole*函数动态地授予和撤销。每个角色都有一个关联的管理角色，只有具有角色的管理角色的帐户才能调用*grantRole*和*revokeRole*。

默认情况下，所有角色的管理角色是DEFAULT_ADMIN_ROLE，这意味着只有具有这个角色的帐户才能授予或撤销其他角色。可以通过使用*_setRoleAdmin*来创建更复杂的角色关系。

> WARNING
DEFAULT_ADMIN_ROLE也是其自身的管理员：它有权限授予和撤销此角色。应采取额外的预防措施来保护被授予此角色的帐户。

**FUNCTIONS**
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
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

#### hasRole(bytes32 role, address account) → bool
公开#
如果帐户已被授予角色，则返回true。

#### getRoleMemberCount(bytes32 role) → uint256
公开#
返回具有角色的账户数量。可以与*getRoleMember*一起使用，以枚举角色的所有承担者。

#### getRoleMember(bytes32 role, uint256 index) → address
公开#
返回具有角色的帐户之一。索引必须是0和*getRoleMemberCount*之间的值，不包括这两个值。

角色持有人不以任何特定方式排序，其排序可能在任何时刻发生变化。

> WARNING
在使用*getRoleMember*和*getRoleMemberCount*时，请确保在同一个区块上执行所有查询。有关更多信息，请参阅以下[论坛帖子](https://forum.openzeppelin.com/t/iterating-over-elements-on-enumerableset-in-openzeppelin-contracts/2296)。

#### getRoleAdmin(bytes32 role) → bytes32
公开#
返回控制角色的管理员角色。请参见*grantRole*和*revokeRole*。

要更改角色的管理员，请使用*_setRoleAdmin*。

#### grantRole(bytes32 role, address account)
公开#
授予账户角色。

如果账户尚未被授予角色，则发出*RoleGranted*事件。

要求：
* 调用者必须具有该角色的管理员角色。

#### revokeRole(bytes32 role, address account)
公开#
撤销账户的角色。

如果账户已被授予角色，则发出一个*RoleRevoked*事件。

要求：
* 调用者必须具有角色的管理员角色。

#### renounceRole(bytes32 role, address account)
公开#
从调用账户中撤销角色。

角色通常通过*grantRole*和*revokeRole*进行管理：该函数的目的是为账户提供一种机制，以防它们被损害而丧失特权（例如当一个受信任的设备丢失时）。

如果调用账户已被授予角色，则发出*RoleRevoked*事件。

要求：
* 调用者必须是账户。

#### _setupRole(bytes32 role, address account)
内部#
将角色授予账户。

如果账户尚未被授予角色，则触发“*RoleGranted*”事件。请注意，与*grantRole*不同，此函数不会对调用账户进行任何检查。

> WARNING
此函数只应在构造函数中设置系统的初始角色时调用。

以其他方式使用此函数实际上是规避*AccessControl*强加的管理员系统。

#### _setRoleAdmin(bytes32 role, bytes32 adminRole)
内部#
将adminRole设置为role的管理员角色。

发出*RoleAdminChanged*事件。

#### RoleAdminChanged(bytes32 role, bytes32 previousAdminRole, bytes32 newAdminRole) 
事件#
当将newAdminRole设置为角色的管理员角色时，取代previousAdminRole时，会发出此事件。

尽管没有发出*RoleAdminChanged*信号，DEFAULT_ADMIN_ROLE仍然是所有角色的起始管理员。

*从v3.1版本开始可用。*

#### RoleGranted(bytes32 role, address account, address sender)
事件#
当账户被授予角色时发出。

发送者是发起合约调用的账户，是管理员角色的持有者，除非使用 *_setupRole*。

#### RoleRevoked(bytes32 role, address account, address sender)
事件#
当账户被撤销角色时发出。

发送者是发起合约调用的账户：- 如果使用revokeRole，发送者是管理员角色的持有者- 如果使用renounceRole，发送者是角色的持有者（即账户）

## 时间锁定

### TimelockController

合约模块作为一个时间锁定控制器。当它被设置为一个可拥有的智能合约的所有者时，它会对所有只有所有者的维护操作进行时间锁定。这样，在应用可能危险的维护操作之前，使用受控合约的用户有时间退出。

默认情况下，这个合约是自我管理的，意味着管理任务必须通过时间锁定的过程进行。提案者（执行者）角色负责提出（执行）操作。一个常见的用例是将这个*TimelockController*作为一个智能合约的所有者，将多签或DAO作为唯一的提案者。

*自v3.3起可用。*

**MODIFIERS**
onlyRole(role)

**FUNCTIONS**
constructor(minDelay, proposers, executors)

receive()

isOperation(id)

isOperationPending(id)

isOperationReady(id)

isOperationDone(id)

getTimestamp(id)

getMinDelay()

hashOperation(target, value, data, predecessor, salt)

hashOperationBatch(targets, values, datas, predecessor, salt)

schedule(target, value, data, predecessor, salt, delay)

scheduleBatch(targets, values, datas, predecessor, salt, delay)

cancel(id)

execute(target, value, data, predecessor, salt)

executeBatch(targets, values, datas, predecessor, salt)

updateDelay(newDelay)

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
CallScheduled(id, index, target, value, data, predecessor, delay)

CallExecuted(id, index, target, value, data)

Cancelled(id)

MinDelayChange(oldDuration, newDuration)

ACCESSCONTROL
RoleAdminChanged(role, previousAdminRole, newAdminRole)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

#### onlyRole(bytes32 role)
修饰符#
修改以使特定角色才能调用函数。除了检查发送者的角色外，还要考虑address(0)的角色。将角色授予address(0)等效于为所有人启用该角色。

#### constructor(uint256 minDelay, address[] proposers, address[] executors)
公开#
使用给定的minDelay初始化合约。

#### receive()
外部#
合约可能在维护过程中接收/持有ETH。

#### isOperation(bytes32 id) → bool pending
公开#
返回一个id是否对应于已注册的操作。这包括待处理、准备好和完成的操作。

#### isOperationPending(bytes32 id) → bool pending
公开#
返回操作是否挂起。

### isOperationReady(bytes32 id) → bool ready
公开#
返回操作是否准备好。

#### isOperationDone(bytes32 id) → bool done
公开#
返回操作是否已完成。

#### getTimestamp(bytes32 id) → uint256 timestamp
公开#
返回操作准备就绪的时间戳（对于未设置的操作为0，对于完成的操作为1）。

#### getMinDelay() → uint256 duration
公开#
返回操作变为有效所需的最小延迟时间。

通过执行调用updateDelay的操作可以更改此值。

#### hashOperation(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt) → bytes32 hash
公开#
返回包含单个事务的操作的标识符。

#### hashOperationBatch(address[] targets, uint256[] values, bytes[] datas, bytes32 predecessor, bytes32 salt) → bytes32 hash
公开#
返回包含一批交易的操作的标识符。

#### schedule(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt, uint256 delay)
公开#
安排一个包含单个交易的操作。

发出一个*CallScheduled*事件。

要求：
* 调用者必须具有“提议者”角色。

#### scheduleBatch(address[] targets, uint256[] values, bytes[] datas, bytes32 predecessor, bytes32 salt, uint256 delay)
公开#
安排一个包含一批交易的操作。

每个交易在批处理中发出一个*CallScheduled*事件。

要求：
* 调用者必须具有'proposer'角色。

#### cancel(bytes32 id)
公开#
取消一项操作。

要求：
* 呼叫者必须具有“提议者”角色。

#### execute(address target, uint256 value, bytes data, bytes32 predecessor, bytes32 salt)
公开#
执行一个包含单个事务的（准备好的）操作。

发出一个*CallExecuted*事件。

要求：
* 调用者必须具有“执行者”角色。

#### executeBatch(address[] targets, uint256[] values, bytes[] datas, bytes32 predecessor, bytes32 salt)
公开#
执行一个包含一批交易的准备操作。

每个批次中的交易都会发出一个*CallExecuted*事件。

要求：
* 调用者必须具有'executor'角色。

#### updateDelay(uint256 newDelay)
外部#
更改未来操作的最小时间锁持续时间。

触发*MinDelayChange*事件。

要求：
* 调用者必须是时间锁本身。只有通过调度并稍后执行将时间锁作为目标，数据为ABI编码调用此函数的操作才能实现。

#### CallScheduled(bytes32 id, uint256 index, address target, uint256 value, bytes data, bytes32 predecessor, uint256 delay)
事件#
当一个调用被计划为操作ID的一部分时触发。

#### CallExecuted(bytes32 id, uint256 index, address target, uint256 value, bytes data) 被执行的调用（id，索引，目标地址，数值，数据）
事件#
当作为操作id的一部分执行呼叫时发出的信号。

#### Cancelled(bytes32 id)
事件#
当操作ID被取消时发出。

#### MinDelayChange(uint256 oldDuration, uint256 newDuration)
事件#
当将来操作的最小延迟被修改时发出。

### Terminology
* 操作：一笔交易（或一组交易），是时间锁的主题。它必须由提议者安排并由执行者执行。时间锁强制在提议和执行之间存在最小延迟（参见*操作生命周期*）。如果操作包含多个交易（批处理模式），它们将以原子方式执行。操作通过其内容的哈希来标识。

* **操作状态：**

    * **未设置：**不是时间锁机制的一部分的操作。

    * **挂起：**已安排但计时器尚未到期的操作。

    * **准备就绪：**已安排且计时器已到期的操作。

    * **完成：**已执行的操作。

* **前置操作：**操作之间的（可选）依赖关系。一个操作可以依赖于另一个操作（其前置操作），强制执行这两个操作的顺序。

* **角色：**

* **提议者：**负责安排（和取消）操作的地址（智能合约或EOA）。

* **执行者：**负责执行操作的地址（智能合约或EOA）。

### 操作结构
由*TimelockControler*执行的操作可以包含一个或多个后续调用。根据您是否需要多个调用以原子方式执行，您可以使用简单或批处理操作。

两种操作都包含：

* **目标**，时间锁应该操作的智能合约的地址。

* **价值**，以wei为单位，应该随交易发送的金额。大多数情况下，这将为0。可以在执行交易之前或之后存入以太币。

* **数据**，包含调用的编码函数选择器和参数。可以使用许多工具来生成此数据。例如，使用web3js可以将授予ROLE角色给ACCOUNT的维护操作编码如下：

```
const data = timelock.contract.methods.grantRole(ROLE, ACCOUNT).encodeABI()
```

* Predecessor，用于指定操作之间的依赖关系。此依赖关系是可选的。如果操作没有任何依赖关系，则使用bytes32(0)。

* Salt，用于消除两个否则相同的操作之间的歧义。可以是任意随机值。

对于批量操作，目标、价值和数据被指定为数组，这些数组必须具有相同的长度。

### 操作生命周期
定时操作通过唯一的id（其哈希）进行标识，并遵循特定的生命周期：

未设置 → 待定 → 待定 + 准备就绪 → 完成

* 通过调用*schedule*（或*scheduleBatch*），提议者将操作从未设置状态移至待定状态。这将启动一个定时器，其持续时间必须超过最小延迟。定时器在通过*getTimestamp*方法可访问的时间戳到期。

* 一旦定时器到期，操作自动进入准备就绪状态。此时，可以执行操作。

* 通过调用*execute*（或*executeBatch*），执行者触发操作的底层交易并将其移至完成状态。如果操作有前任，则必须处于完成状态才能进行此过渡。

*cancel*允许提议者取消任何待定操作。这将重置操作为未设置状态。因此，提议者可以重新安排已取消的操作。在这种情况下，操作重新安排时定时器重新启动。

可以使用以下函数查询操作状态：

* *isOperationPending（bytes32）*

* *isOperationReady（bytes32）*

* *isOperationDone（bytes32）*

## 角色
### 管理员
管理员负责管理提议者和执行者。为了使时间锁能够自我管理，此角色应仅授予时间锁本身。部署后，时间锁和部署者都具有此角色。在进一步配置和测试之后，部署者可以放弃此角色，以便所有后续的维护操作都必须通过时间锁流程进行。

此角色由**TIMELOCK_ADMIN_ROLE**值标识：0x5f58e3a2316349923ce3780f8d587db2d72378aed66a8261c916544fa6846ca5

### 提议者
提议者负责安排（和取消）操作。这是一个关键角色，应该授予治理实体。这可以是EOA、多签名或DAO。

> WARNING
**提议者之争**：拥有多个提议者在一个不可用时提供冗余，但可能很危险。由于提议者对所有操作都有发言权，他们可以取消他们不同意的操作，包括删除他们自己的提议者的操作。

此角色由**PROPOSER_ROLE**值标识：0xb09aa5aeb3702cfd50b6b62bc4532604938f21248a27a1d5ca736082b6819cc1

### 执行者
执行者负责执行提议者安排的操作，一旦时间锁到期。逻辑上讲，作为提议者的多签名或DAO也应该是执行者，以确保已安排的操作最终会被执行。然而，拥有额外的执行者可以降低成本（执行交易不需要由提议者进行验证），同时确保负责执行的人员不能触发未经提议者安排的操作。

此角色由**EXECUTOR_ROLE**值标识：0xd8aa0f3194971a2a116679f7c2090f6939c8d4e01a2a8d7e41d55e5351469e63

> WARNING
没有至少一个提议者和一个执行者的活动合约是被锁定的。在部署者放弃其管理权利，转交给时间锁合约本身之前，请确保这些角色由可靠的实体填充。有关角色管理的更多信息，请参阅*AccessControl*文档。