# Access
> CAUTION
预计这些模块会发展演变。

在智能合约世界中，访问控制即“谁被允许执行这个操作”非常重要。合约的访问控制可能会管理谁可以铸造代币、对提案进行投票、冻结转账等等许多事情。因此，了解如何实现访问控制非常关键，[以免被他人窃取整个系统](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7/)。

# 目录
- [Access](#access)
- [目录](#目录)
  - [Ownable](#ownable)
    - [快速入门](#快速入门)
    - [Ownable库API](#ownable库api)
      - [initializer](#initializer)
      - [assert\_only\_owner](#assert_only_owner)
      - [owner](#owner)
      - [transfer\_ownership](#transfer_ownership)
      - [renounce\_ownership](#renounce_ownership)
      - [\_transfer\_ownership](#_transfer_ownership)
    - [Ownable events](#ownable-events)
      - [OwnershipTransferred](#ownershiptransferred)
  - [AccessControl](#accesscontrol)
    - [用法](#用法)
    - [授予和撤销角色](#授予和撤销角色)
    - [创建角色标识符](#创建角色标识符)
    - [AccessControl library API](#accesscontrol-library-api)
      - [initializer](#initializer-1)
      - [assert\_only\_role](#assert_only_role)
      - [has\_role](#has_role)
      - [get\_role\_admin](#get_role_admin)
      - [grant\_role](#grant_role)
      - [revoke\_role](#revoke_role)
      - [renounce\_role](#renounce_role)
      - [\_grant\_role](#_grant_role)
      - [\_revoke\_role](#_revoke_role)
      - [\_set\_role\_admin](#_set_role_admin)
    - [AccessControl events](#accesscontrol-events)
      - [RoleGranted](#rolegranted)
      - [RoleRevoked](#rolerevoked)
      - [RoleAdminChanged](#roleadminchanged)

## Ownable
访问控制最常见和基本的形式是所有权的概念：有一个账户是合约的所有者，并且可以对其进行管理任务。对于只有一个管理用户的合约来说，这种方法是完全合理的。

OpenZeppelin Contracts for Cairo提供了[Ownable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.6.1/src/openzeppelin/access/ownable/library.cairo)来在合约中实现所有权。

### 快速入门
将[Ownable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.6.1/src/openzeppelin/access/ownable/library.cairo)集成到合约中首先需要分配一个所有者。实现合约的构造函数应该通过将所有者的地址传递给Ownable的[初始化器](./Accounts.md#构造函数)来设置初始所有者，就像这样:
```
from openzeppelin.access.ownable.library import Ownable

@constructor
func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(owner: felt) {
    Ownable.initializer(owner);
    return ();
}
```
为了限制函数只能被所有者访问，可以添加assert_only_owner方法。

```
from openzeppelin.access.ownable.library import Ownable

func protected_function{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    Ownable.assert_only_owner();
    return ();
}
```

### Ownable库API
```
func initializer(owner: felt) {
}

func assert_only_owner() {
}

func owner() -> (owner: felt) {
}

func transfer_ownership(new_owner: felt) {
}

func renounce_ownership() {
}

func _transfer_ownership(new_owner: felt) {
}
```

#### initializer
初始化Ownable访问控制，并应在实现合约的构造函数中调用。将owner分配为合约的初始所有者地址。

这个函数只能被调用一次。

参数:
```
owner: felt
```
返回：无。

#### assert_only_owner
如果由所有者之外的任何账户调用，则回滚。如果所有权被放弃，则来自默认的零地址的任何调用也将被回滚。

参数：无。

返回值：无。

#### owner
返回当前所有者的地址。

参数：无。

返回值：
```
owner: felt
```

#### transfer_ownership
将合约的所有权转移到一个新账户（new_owner）。只能由当前所有者调用。

触发一个[OwnershipTransferred](#ownershiptransferred)事件。

参数:
```
new_owner: felt
```
返回：无。

#### renounce_ownership
离开合约没有所有者。将不再能够使用assert_only_owner调用函数。只能由当前所有者调用。

发出一个[OwnershipTransferred](#ownershiptransferred)事件。

参数：无。

返回：无。

#### _transfer_ownership
将合约的所有权转移到一个新的账户（new_owner）。*内部函数*，没有访问限制。

触发一个[OwnershipTransferred](#ownershiptransferred)事件。

参数:
```
new_owner: felt
```
返回：无。

### Ownable events
```
func OwnershipTransferred(previousOwner: felt, newOwner: felt) {
}
```

#### OwnershipTransferred
当合约的所有权从previousOwner转移到newOwner时发出。

参数:
```
previousOwner: felt
newOwner: felt
```

## AccessControl
虽然所有权的简单性对于简单的系统或快速原型设计非常有用，但通常需要不同级别的授权。您可能希望一个帐户有权禁止用户使用系统，但不能创建新的代币。[基于角色的访问控制（RBAC）](https://en.wikipedia.org/wiki/Role-based_access_control)在这方面提供了灵活性。

本质上，我们将定义多个角色，每个角色可以执行不同的操作集。一个帐户可能具有“调解员”、“铸造者”或“管理员”等角色，您将检查这些角色而不是简单地使用[assert_only_owner](#assert_only_owner)。此检查可以通过[assert_only_role](#assert_only_role)来强制执行。另外，您还可以定义授予角色、撤销角色等的规则。

大多数软件使用基于角色的访问控制系统：一些用户是普通用户，一些可能是主管或经理，而少数人通常具有管理权限。

### 用法
对于要定义的每个角色，您将创建一个新的角色标识符，用于授予、撤销和检查帐户是否具有该角色（有关[创建标识符的信息](#创建角色标识符)，请参见创建角色标识符）。

下面是在[ERC20代币合约](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/src/openzeppelin/token/erc20/presets/ERC20.cairo)的一部分上实现AccessControl的简单示例，该示例定义并设置了“铸造者”角色。
```
from openzeppelin.token.erc20.library import ERC20

from openzeppelin.access.accesscontrol.library import AccessControl


const MINTER_ROLE = 0x4f96f87f6963bb246f2c30526628466840c642dc5c50d5a67777c6cc0e44ab5

@constructor
func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    name: felt, symbol: felt, decimals: felt, minter: felt
) {
    ERC20.initializer(name, symbol, decimals);
    AccessControl.initializer();
    AccessControl._grant_role(MINTER_ROLE, minter);
    return ();
}

@external
func mint{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    to: felt, amount: Uint256
) {
    AccessControl.assert_only_role(MINTER_ROLE);
    ERC20._mint(to, amount);
    return ();
}
```

> CAUTION
在使用[AccessControl](#accesscontrol)之前，请确保您完全理解它的工作原理，或者复制粘贴本指南中的示例。

虽然明确和明确，但这并不是我们不能通过[Ownable](#ownable)实现的任何东西。实际上，AccessControl在需要细粒度权限的场景中非常出色，可以通过定义多个角色来实现。

让我们通过定义一个“burner”角色来增强我们的ERC20代币示例，该角色允许账户销毁代币，并使用assert_only_role:
```
from openzeppelin.token.erc20.library import ERC20

from openzeppelin.access.accesscontrol.library import AccessControl


const MINTER_ROLE = 0x4f96f87f6963bb246f2c30526628466840c642dc5c50d5a67777c6cc0e44ab5
const BURNER_ROLE = 0x7823a2d975ffa03bed39c38809ec681dc0ae931ebe0048c321d4a8440aed509

@constructor
func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    name: felt, symbol: felt, decimals: felt, minter: felt, burner: felt
) {
    ERC20.initializer(name, symbol, decimals);
    AccessControl.initializer();
    AccessControl._grant_role(MINTER_ROLE, minter);
    AccessControl._grant_role(BURNER_ROLE, burner);
    return ();
}

@external
func mint{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    to: felt, amount: Uint256
) {
    AccessControl.assert_only_role(MINTER_ROLE);
    ERC20._mint(to, amount);
    return ();
}

@external
func burn{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    from_: felt, amount: Uint256
) {
    AccessControl.assert_only_role(BURNER_ROLE);
    ERC20._burn(from_, amount);
    return ();
}
```

如此干净！通过这种方式分离关注点，可以实现比简单的所有权访问控制更细粒度的权限级别。限制系统的每个组件能够做什么被称为[最小特权原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，这是一个良好的安全实践。请注意，如果需要，每个账户仍然可以拥有多个角色。

### 授予和撤销角色
上面的ERC20代币示例使用了_grant_role，这是一个在编程分配角色时有用的[内部](./Extensibility.md#该模式)函数（例如在构造过程中）。但是，如果我们以后想要将“铸造者”角色授予其他账户怎么办？

默认情况下，**具有角色的账户无法授予或撤销它**：拥有角色只是使assert_only_role检查通过。要动态授予和撤销角色，您需要获得该角色的管理员的帮助。

每个角色都有一个关联的管理员角色，该角色授予调用grant_role和revoke_role函数的权限。如果调用账户具有相应的管理员角色，则可以使用这些函数授予或撤销角色。为了简化管理，多个角色可以具有相同的管理员角色。角色的管理员甚至可以是角色本身，这将导致具有该角色的账户也能够授予和撤销它。

这个机制可以用来创建类似组织结构图的复杂权限结构，但它也提供了一种管理简单应用程序的简单方法。AccessControl包括一个特殊角色，角色标识符为0，称为DEFAULT_ADMIN_ROLE，它作为**所有角色的默认管理员角**色。具有此角色的账户将能够管理任何其他角色，除非使用_set_role_admin选择新的管理员角色。

让我们看一下ERC20代币的示例，这次利用默认管理员角色:
```
from openzeppelin.token.erc20.library import ERC20

from openzeppelin.access.accesscontrol.library import AccessControl

from openzeppelin.utils.constants import DEFAULT_ADMIN_ROLE


const MINTER_ROLE = 0x4f96f87f6963bb246f2c30526628466840c642dc5c50d5a67777c6cc0e44ab5
const BURNER_ROLE = 0x7823a2d975ffa03bed39c38809ec681dc0ae931ebe0048c321d4a8440aed509

@constructor
func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    name: felt, symbol: felt, decimals: felt, admin: felt,
) {
    ERC20.initializer(name, symbol, decimals);
    AccessControl.initializer();

    AccessControl._grant_role(DEFAULT_ADMIN_ROLE, admin);
    return ();
}

@external
func mint{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    to: felt, amount: Uint256
) {
    AccessControl.assert_only_role(MINTER_ROLE);
    ERC20._mint(to, amount);
    return ();
}

@external
func burn{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    from_: felt, amount: Uint256
) {
    AccessControl.assert_only_role(BURNER_ROLE);
    ERC20._burn(from_, amount);
    return ();
}
```

请注意，与之前的示例不同，没有任何账户被授予“铸币者”或“销毁者”角色。然而，因为这些角色的管理员角色是默认的管理员角色，并且该角色被授予了“管理员”，同样的账户可以调用grant_role来授予铸币或销毁权限，并调用revoke_role来删除它。

动态角色分配通常是一种理想的属性，例如在信任参与者的程度可能随时间变化的系统中。它还可以用于支持诸如[KYC](https://en.wikipedia.org/wiki/Know_your_customer)之类的用例，其中角色承担者的列表可能事先不知道，或者将其包含在单个事务中可能代价太高。

下面的示例使用了公开角色管理函数的[AccessControl模拟合约](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.4.0b/tests/mocks/AccessControl.cairo)。例如，在Python中授予和撤销角色：
```
MINTER_ROLE = 0x4f96f87f6963bb246f2c30526628466840c642dc5c50d5a67777c6cc0e44ab5
BURNER_ROLE = 0x7823a2d975ffa03bed39c38809ec681dc0ae931ebe0048c321d4a8440aed509

# grants MINTER_ROLE and BURNER_ROLE to account1 and account2 respectively
await signer.send_transactions(
    admin, [
        (accesscontrol.contract_address, 'grantRole', [MINTER_ROLE, account1.contract_address]),
        (accesscontrol.contract_address, 'grantRole', [BURNER_ROLE, account2.contract_address])
    ]
)

# revokes MINTER_ROLE from account1
await signer.send_transaction(
    admin,
    accesscontrol.contract_address,
    'revokeRole',
    [MINTER_ROLE, account1.contract_address]
)
```

### 创建角色标识符
在AccessControl的Solidity实现中，合约通常将角色的[keccak256哈希](https://docs.soliditylang.org/en/latest/units-and-global-variables.html?highlight=keccak256#mathematical-and-cryptographic-functions)称为角色标识符。例如：
```
bytes32 public constant SOME_ROLE = keccak256("SOME_ROLE")
```

这些标识符占用32字节（256位）。

Cairo字段元素最多存储252位。更进一步，StarkNet合约中声明的常量字段元素存储的位数更少（请参阅[cairo_constants](https://github.com/starkware-libs/cairo-lang/blob/release-v0.4.0b/src/starkware/cairo/lang/cairo_constants.py#L1)）。鉴于这种差异，该库对于合约如何创建标识符保持中立立场。以下是一些可以考虑的想法：

使用keccak256哈希摘要的前251位或后251位。

使用Cairo的[hash2](https://github.com/starkware-libs/cairo-lang/blob/master/src/starkware/cairo/common/hash.cairo)函数。

### AccessControl library API
```
func initializer() {
}

func assert_only_role(role: felt) {
}

func has_role(role: felt, user: felt) -> (has_role: felt) {
}

func get_role_admin(role: felt) -> (admin: felt) {
}

func grant_role(role: felt, user: felt) {
}

func revoke_role(role: felt, user: felt) {
}

func renounce_role(role: felt, user: felt) {
}

func _grant_role(role: felt, user: felt) {
}

func _revoke_role(role: felt, user: felt) {
}

func _set_role_admin(role: felt, admin_role: felt) {
}
```

#### initializer
初始化AccessControl并应在实现合约的构造函数中调用。

这个函数只能被调用一次。

参数：无。

返回值：无。

#### assert_only_role
检查一个账户是否具有特定角色。返回一个包含所需角色的消息。

参数:
```
role: felt
```
返回：无。

#### has_role
如果用户被授予了角色，则返回TRUE，否则返回FALSE。

参数：
```
role: felt
user: felt
```
返回值:
```
has_role: felt
```

#### get_role_admin
返回控制角色的管理员角色。请参阅[grant_role](#grant_role)和[revoke_role](#revoke_role)。

要更改角色的管理员，请使用[_set_role_admin](#_set_role_admin)。

参数:
```
role: felt
```

返回：
```
admin: felt
```

#### grant_role
授予用户角色。

如果用户尚未被授予角色，则发出[RoleGranted](#rolegranted)事件。

要求：

调用者必须具有角色的管理员角色。

参数:
```
role: felt
user: felt
```
返回值：无。

#### revoke_role
撤销用户的角色。

如果用户被授予了角色，则发出[RoleRevoked](#rolerevoked)事件。

要求：

调用者必须具有角色的管理员角色。

参数:
```
role: felt
user: felt
```
返回值：无。

#### renounce_role
从调用用户中撤销角色。

角色通常通过[授予角色](#grant_role)和[撤销角色](#revoke_role)进行管理：此函数的目的是提供一种机制，以防止账户被攻击者获取权限（例如当一个受信任的设备丢失时）。

如果调用用户已被撤销角色，则会发出一个[RoleGranted](#rolegranted)事件。

要求：

* 调用者必须是用户。

参数:
```
role: felt
user: felt
```

返回值：无。

#### _grant_role
向用户授予角色。

[内部](./Extensibility.md#该模式)函数，无访问限制。

触发一个[RoleGranted](#rolegranted) 事件。

参数:
```
role: felt
user: felt
```

返回值：无。

#### _revoke_role
撤销用户的角色。

[内部](./Extensibility.md#该模式)函数，没有访问限制。

触发[RoleRevoked](#rolerevoked)事件。

参数:
```
role: felt
user: felt
```

返回值：无。

#### _set_role_admin
[内部](./Extensibility.md#该模式)函数将admin_role设置为角色的管理员角色。

触发[RoleAdminChanged](#roleadminchanged)事件。

参数:
```
role: felt
admin_role: felt
```

返回值：无。

### AccessControl events
```
func RoleGranted(role: felt, account: felt, sender: felt) {
}

func RoleRevoked(role: felt, account: felt, sender: felt) {
}

func RoleAdminChanged(role: felt, previousAdminRole: felt, newAdminRole: felt) {
}
```

#### RoleGranted

当账户被授予角色时发出。

发送者是发起合约调用的账户，是管理员角色的持有者。

参数:
```
role: felt
account: felt
sender: felt
```

#### RoleRevoked
当账户被撤销角色时发出。

发送者是发起合约调用的账户：

* 如果使用[revoke_role](#revoke_role)，发送者是管理员角色的持有者。

* 如果使用[renounce_role](#renounce_role)，发送者是角色的持有者（即账户）。

```
role: felt
account: felt
sender: felt
```

#### RoleAdminChanged
当将newAdminRole设置为角色的管理员角色时，替换previousAdminRole时，会发出此事件。

DEFAULT_ADMIN_ROLE是所有角色的初始管理员，尽管没有发出RoleAdminChanged信号来表示这一点。
```
role: felt
previousAdminRole: felt
newAdminRole: felt
```
