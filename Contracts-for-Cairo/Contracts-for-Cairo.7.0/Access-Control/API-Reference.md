# Access Control
该目录提供了限制谁可以访问合约功能或何时可以访问的方法。

* *Ownable*是一种简单的机制，它有一个可以分配给单个账户的"所有者"角色。在简单的场景中，这种机制可能很有用，但对于需要细粒度访问控制的场景，可能会超出其能力。

* *AccessControl*提供了一种通用的基于角色的访问控制机制。可以创建多个分层角色，并将每个角色分配给多个账户。

## Authorization

### [Ownable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/access/ownable/ownable.cairo)
```
use openzeppelin::access::ownable::Ownable;
```

Ownable提供了一种基本的访问控制机制，其中一个账户（所有者）可以被授予对特定功能的专有访问权限。

这个模块包括了内部的assert_only_owner，用于限制只有所有者才能使用的功能。

**EXTERNAL FUNCTIONS**
OWNABLEIMPL
owner(self)

transfer_ownership(self, new_owner)

renounce_ownership(self)

**INTERNAL FUNCTIONS**
INTERNALIMPL
initializer(self, owner)

assert_only_owner(self)

_transfer_ownership(self, new_owner)

**EVENTS**
OwnershipTransferred(previous_owner, new_owner)

#### External Functions

##### owner(self: @ContractState) → ContractAddress
external#
返回当前所有者的地址。

##### transfer_ownership(ref self: ContractState, new_owner: ContractAddress)
external#
将合约的所有权转移给新账户（new_owner）。只有当前所有者才能调用。

触发一个*OwnershipTransferred*事件。

##### renounce_ownership(ref self: ContractState)
external#
放弃合约的所有权。将不再可能调用仅所有者函数。只有当前所有者才能调用。

> NOTE
放弃所有权将使合约没有所有者，从而删除只有所有者才能使用的任何功能。

#### Internal Functions

##### initializer(ref self: ContractState, owner: ContractAddress)
internal#
初始化合约并将所有者设为初始所有者。

触发一个*OwnershipTransferred*事件。

##### assert_only_owner(self: @ContractState)
internal#
如果被非所有者的账户调用，将会引发警告。

##### _transfer_ownership(ref self: ContractState, new_owner: ContractAddress)
internal#
将合同的所有权转移给新的账户（new_owner）。这是一个没有访问限制的内部函数。

触发一个*OwnershipTransferred*事件。

#### Events

##### OwnershipTransferred(previous_owner: ContractAddress, new_owner: ContractAddress)
event#
在所有权转移时发出。

#### [IAccessControl](https://github.com/OpenZeppelin/cairo-contracts/blob/05429e4fd34a250ce7a01450190c53275e5c1c0b/src/access/accesscontrol/interface.cairo#L10)
```
use openzeppelin::access::accesscontrol::interface::IAccessControl;
```
访问控制的外部接口

**SRC5 ID**
0x23700be02858dbe2ac4dc9c9f66d0b6b0ed81ec7f970ca6844500a56ff61751

**FUNCTIONS**
has_role(role, account)

get_role_admin(role)

grant_role(role, account)

revoke_role(role, account)

renounce_role(role, account)

**EVENTS**
RoleAdminChanged(role, previous_admin_role, new_admin_role)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

#### Functions

##### has_role(role: felt252, account: ContractAddress) → bool
external#
如果账户已被授予角色，则返回真。

##### get_role_admin(role: felt252) → felt252
external#
返回控制角色的管理员角色。参见 *grant_role* 和 *revoke_role*。

要更改角色的管理员，请使用 *_set_role_admin*。

##### grant_role(role: felt252, account: ContractAddress)
external#
向账户授予角色。

如果账户尚未被授予角色，则发出一个*RoleGranted*事件。

要求：
* 调用者必须具有角色的管理员角色。

##### revoke_role(role: felt252, account: ContractAddress)
external#
从账户撤销角色。

如果账户已被授予角色，则会发出一个*RoleRevoked*事件。

要求：
* 调用者必须拥有角色的管理员角色。

##### renounce_role(role: felt252, account: ContractAddress)
external#
从调用账户撤销角色。

角色通常通过*grant_role*和*revoke_role*进行管理。此函数的目的是为账户提供一种机制，如果它们被泄露（例如，当信任的设备被遗失时），可以失去其权限。

如果调用账户已被授予角色，则发出*RoleRevoked*事件。

要求：
* 调用者必须是账户。

#### Events

##### RoleAdminChanged(role: felt252, previous_admin_role: ContractAddress, new_admin_role: ContractAddress)
event#
当new_admin_role被设置为角色的管理员角色，替换previous_admin_role时发出。

DEFAULT_ADMIN_ROLE是所有角色的初始管理员，尽管没有发出*RoleAdminChanged*信号表示这一点。

##### RoleGranted(role: felt252, account: ContractAddress, sender: ContractAddress)
event#
当账户被授予角色时发出。

发送者是发起合同呼叫的账户，是管理员角色的持有者。

##### RoleRevoked(role: felt252, account: ContractAddress, sender: ContractAddress)
event#
当账户被撤销角色时发出。

发送者是发起合约调用的账户：

如果使用revoke_role，它是管理员角色的持有者。

如果使用renounce_role，它是角色持有者（即账户）。

### [AccessControl](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/access/accesscontrol/accesscontrol.cairo)
```
use openzeppelin::access::accesscontrol::AccessControl;
```

允许子级实现基于角色的访问控制机制的合同模块。角色由其felt252标识符引用。

```
const MY_ROLE: felt252 = selector!('MY_ROLE');
```

角色可以用来表示一组权限。为了限制对函数调用的访问，请使用*assert_only_role*:
```
use openzeppelin::access::accesscontrol::AccessControl::InternalImpl::assert_only_role;
use openzeppelin::access::accesscontrol::AccessControl;
use openzeppelin::token::erc20::ERC20;

#[external(v0)]
fn foo(ref self: ContractState, account: ContractAddress, amount: u256) {
    let access_state = AccessControl::unsafe_new_contract_state();
    assert_only_role(@access_state, BURNER_ROLE);

    let mut erc20_state = ERC20::unsafe_new_contract_state();
    ERC20::InternalImpl::_burn(ref erc20_state, account, amount);
}
```

角色可以通过*grant_role*和*revoke_role*函数动态地授予和撤销。每个角色都有一个关联的管理员角色，只有拥有角色的管理员角色的账户才能调用*grant_role*和*revoke_role*。

默认情况下，所有角色的管理员角色都是DEFAULT_ADMIN_ROLE，这意味着只有拥有此角色的账户才能授予或撤销其他角色。可以通过使用*_set_role_admin*创建更复杂的角色关系。

> WARNING
DEFAULT_ADMIN_ROLE也是它自己的管理员：它有权限授予和撤销这个角色。应该采取额外的预防措施来保护已经被授予它的账户。

**EXTERNAL FUNCTIONS**
ACCESSCONTROLIMPL
has_role(self, role, account)

get_role_admin(self, role)

grant_role(self, role, account)

revoke_role(self, role, account)

renounce_role(self, role, account)

SRC5IMPL
supports_interface(self, interface_id: felt252)

**INTERNAL FUNCTIONS**
INTERNALIMPL
initializer(self)

_set_role_admin(self, role, admin_role)

_grant_role(self, role, account)

_revoke_role(self, role, account)

assert_only_role(self, role)

**EVENTS**
IACCESSCONTROL
RoleAdminChanged(role, previous_admin_role, new_admin_role)

RoleGranted(role, account, sender)

RoleRevoked(role, account, sender)

#### External Functions

##### has_role(self: @ContractState, role: felt252, account: ContractAddress) → bool
external#
如果账户已被授予角色，则返回真。

##### get_role_admin(self: @ContractState, role: felt252) → felt252
external#
返回控制角色的管理员角色。请参阅*grant_role*和*revoke_role*。

要更改角色的管理员，请使用*_set_role_admin*。

##### grant_role(ref self: ContractState, role: felt252, account: ContractAddress)
external#
授予账户角色。

如果账户尚未被授予角色，则会触发*RoleGranted*事件。

要求：
*调用者必须拥有角色的管理员角色。

可能会触发*RoleGranted*事件。

##### revoke_role(ref self: ContractState, role: felt252, account: ContractAddress)
external#
从账户撤销角色。

如果账户已被授予角色，则会触发一个*RoleRevoked*事件。

要求：
* 调用者必须拥有角色的管理员角色。

可能会触发一个*RoleRevoked*事件。

##### renounce_role(ref self: ContractState, role: felt252, account: ContractAddress)
external#
从调用账户撤销角色。

角色通常通过*grant_role*和*revoke_role*进行管理。此函数的目的是为账户提供一种机制，使其在受到威胁时（例如，信任的设备丢失时）失去其权限。

如果调用账户的角色已被撤销，将发出*RoleRevoked*事件。

要求：
* 调用者必须是账户。

可能会发出*RoleRevoked*事件。

##### supports_interface(self: @ContractState, interface_id: felt252) → bool
external#
See *ISRC5::supports_interface*.

#### Internal Functions

##### initializer(ref self: ContractState)
internal#
通过注册IAccessControl接口ID来初始化合约。

##### _set_role_admin(ref self: ContractState, role: felt252, admin_role: felt252)
internal#
将admin_role设置为角色的管理员角色。

触发一个*RoleAdminChanged*事件。

##### _grant_role(ref self: ContractState, role: felt252, account: ContractAddress)
internal#
向账户授予角色。

内部功能，无访问限制。

可能会触发*RoleGranted*事件。

##### _revoke_role(ref self: ContractState, role: felt252, account: ContractAddress)
internal#
从账户撤销角色。

内部功能，无访问限制。

可能会触发*RoleRevoked*事件。

##### assert_only_role(self: @ContractState, role: felt252)
internal#
如果被没有给定角色的任何账户调用，将会引发恐慌。

#### Events

##### RoleAdminChanged(role: felt252, previous_admin_role: ContractAddress, new_admin_role: ContractAddress)
event#
请查阅 *IAccessControl::RoleAdminChanged*.

##### RoleGranted(role: felt252, account: ContractAddress, sender: ContractAddress)
event#
请查阅 *IAccessControl::RoleGranted*.

##### RoleRevoked(role: felt252, account: ContractAddress, sender: ContractAddress)
event#
请查阅 *IAccessControl::RoleRevoked*.