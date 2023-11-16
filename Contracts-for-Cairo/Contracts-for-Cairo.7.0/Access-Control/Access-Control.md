# Access
访问控制——即"谁有权做这件事"——在智能合约的世界中非常重要。你的合约的访问控制可能决定谁可以铸造代币，对提案进行投票，冻结转账，以及许多其他事情。因此，理解你如何实施它至关重要，以免有人[窃取你的整个系统](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7/)。

## Ownership and Ownable
最常见和最基本的访问控制形式是所有权的概念：有一个账户是合约的所有者，可以对其进行管理任务。对于只有一个管理用户的合约，这种方法是完全合理的。

OpenZeppelin Contracts for Cairo为在你的合约中实现所有权提供了[Ownable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/access/ownable/ownable.cairo)。

### 使用
将此模块集成到合约中首先需要分配一个所有者。实施合约的构造函数应通过将所有者的地址传递给Ownable的*初始化器*来设置初始所有者，如下所示:
```
use openzeppelin::access::ownable::Ownable;

#[starknet::contract]
mod MyContract {
    use starknet::ContractAddress;
    use super::Ownable;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(self: @ContractState, owner: ContractAddress) {
        let mut unsafe_state = Ownable::unsafe_new_contract_state();
        Ownable::InternalImpl::initializer(ref unsafe_state, owner);
    }

    (...)
}
```

为了限制函数只对所有者开放，需要在assert_only_owner方法中添加限制:
```
#[starknet::contract]
mod MyContract {
    use openzeppelin::access::ownable::Ownable;

    (...)

    #[external(v0)]
    fn foo(ref self: ContractState) {
        // This function can only be called by the owner
        let unsafe_state = Ownable::unsafe_new_contract_state();
        Ownable::InternalImpl::assert_only_owner(@unsafe_state);

        (...)
    }
}
```

### 接口
这是Ownable实现的完整接口:
```
trait IOwnable {
    /// Returns the current owner.
    fn owner() -> ContractAddress;

    /// Transfers the ownership from the current owner to a new owner.
    fn transfer_ownership(new_owner: ContractAddress);

    /// Renounces the ownership of the contract.
    fn renounce_ownership();
}
```

Ownable还允许你：

* 将所有权从所有者账户转移到新的账户，以及

* 放弃所有权，以便所有者放弃这种管理权限，这是在初始阶段结束后常见的模式。

> WARING
完全去除所有者意味着受assert_only_owner保护的管理任务将无法再被调用！

## Role-Based AccessControl
虽然所有权的简单性对于简单系统或快速原型设计可能很有用，但通常需要不同级别的授权。你可能希望一个账户有权限从系统中禁止用户，但不能创建新的代币。[基于角色的访问控制（RBAC）](https://en.wikipedia.org/wiki/Role-based_access_control)在这方面提供了灵活性。

本质上，我们将定义多个角色，每个角色都被允许执行不同的操作集。例如，一个账户可能有'moderator'，'minter'或'admin'角色，然后你将检查这些角色，而不是简单地使用*assert_only_owner*。这个检查可以通过*assert_only_role*来强制执行。另外，你将能够定义规则，规定如何授予账户角色，如何撤销角色等。

大多数软件使用的访问控制系统都是基于角色的：有些用户是普通用户，有些可能是主管或经理，而少数人通常会有管理权限。

### 使用方法
对于你想要定义的每个角色，你将创建一个新的角色标识符，用于授予、撤销和检查账户是否具有该角色。有关创建标识符的信息，请参见创建角色标识符。

以下是在ERC20代币合约的一部分上实现AccessControl的一个简单示例，该示例定义并设置了一个'minter'角:
```
const MINTER_ROLE: felt252 = selector!('MINTER_ROLE');

#[starknet::contract]
mod MyContract {
    use openzeppelin::access::accesscontrol::AccessControl::InternalImpl::assert_only_role;
    use openzeppelin::access::accesscontrol::AccessControl;
    use openzeppelin::token::erc20::ERC20;
    use starknet::ContractAddress;
    use super::MINTER_ROLE;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(
        ref self: ContractState,
        name: felt252,
        symbol: felt252,
        initial_supply: u256,
        recipient: ContractAddress,
        minter: ContractAddress
    ) {
        // ERC20 related initialization
        let mut erc20_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref erc20_state, name, symbol);
        ERC20::InternalImpl::_mint(ref erc20_state, recipient, initial_supply);

        // AccessControl related initialization
        let mut access_state = AccessControl::unsafe_new_contract_state();
        AccessControl::InternalImpl::initializer(ref access_state);
        AccessControl::InternalImpl::_grant_role(
            ref access_state,
            MINTER_ROLE,
            minter
        );
    }

    // This function can only be called by a minter
    #[external(v0)]
    fn mint(ref self: ContractState, recipient: ContractAddress, amount: u256) {
        let access_state = AccessControl::unsafe_new_contract_state();
        assert_only_role(@access_state, MINTER_ROLE);

        let mut erc20_state = AccessControl::unsafe_new_contract_state();
        ERC20::InternalImpl::_mint(ref erc20_state, recipient, amount);
    }
}
```

> CAUTION
在你的系统上使*用AccessControl*或从本指南中复制粘贴示例之前，请确保你完全理解AccessControl的工作方式。

虽然这很清晰和明确，但这并不是我们无法通过*Ownable*实现的。AccessControl最出色的地方是在需要精细权限的场景中，这可以通过定义多个角色来实现。

让我们通过定义一个“燃烧者”角色来增强我们的ERC20代币示例，这让账户可以销毁代币:
```
const MINTER_ROLE: felt252 = selector!('MINTER_ROLE');
const BURNER_ROLE: felt252 = selector!('BURNER_ROLE');

#[starknet::contract]
mod MyContract {
    use openzeppelin::access::accesscontrol::AccessControl::InternalImpl::assert_only_role;
    use openzeppelin::access::accesscontrol::AccessControl;
    use openzeppelin::token::erc20::ERC20;
    use starknet::ContractAddress;
    use super::{MINTER_ROLE, BURNER_ROLE};

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(
        ref self: ContractState,
        name: felt252,
        symbol: felt252,
        initial_supply: u256,
        recipient: ContractAddress,
        minter: ContractAddress,
        burner: ContractAddress
    ) {
        // 与ERC20相关的初始化
        let mut erc20_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref erc20_state, name, symbol);
        ERC20::InternalImpl::_mint(ref erc20_state, recipient, initial_supply);

        // 与访问控制相关的初始化
        let mut access_state = AccessControl::unsafe_new_contract_state();
        AccessControl::InternalImpl::initializer(ref access_state);
        AccessControl::InternalImpl::_grant_role(
            ref access_state,
            MINTER_ROLE,
            minter
        );
        AccessControl::InternalImpl::_grant_role(
            ref access_state,
            BURNER_ROLE,
            burner
        );
    }


    // 只有铸币者才能调用此函数
    #[external(v0)]
    fn mint(ref self: ContractState, recipient: ContractAddress, amount: u256) {
        let access_state = AccessControl::unsafe_new_contract_state();
        assert_only_role(@access_state, MINTER_ROLE);

        let mut erc20_state = AccessControl::unsafe_new_contract_state();
        ERC20::InternalImpl::_mint(ref erc20_state, recipient, amount);
    }

    // 这个功能只能由销毁调用
    #[external(v0)]
    fn burn(ref self: ContractState, account: ContractAddress, amount: u256) {
        let access_state = AccessControl::unsafe_new_contract_state();
        assert_only_role(@access_state, BURNER_ROLE);

        let mut erc20_state = AccessControl::unsafe_new_contract_state();
        ERC20::InternalImpl::_burn(ref erc20_state, account, amount);
    }
}
```

如此清晰！通过这种方式分割关注点，可以实现比简单的所有权访问控制方式更细粒度的权限。限制系统的每个组件能够做什么被称为[principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，这是一种良好的安全实践。请注意，每个账户仍然可以拥有多个角色，如果需要的话。

授予和撤销角色
上面的ERC20代币示例使用了*_grant_role*，这是一个在编程分配角色时（例如在构造过程中）非常有用的内部函数。但是，如果我们后来想要授予更多账户“铸币者”角色怎么办？

默认情况下，拥有角色的账户不能授予或撤销其他账户的角色：拥有角色只是使*assert_only_role*检查通过。要动态地授予和撤销角色，你将需要角色的管理员的帮助。

每个角色都有一个相关的管理员角色，它授予调用*grant_role*和*revoke_role*函数的权限。如果调用账户拥有相应的管理员角色，就可以使用这些函数授予或撤销角色。多个角色可以拥有相同的管理员角色，以便于管理。角色的管理员甚至可以是角色本身，这将导致拥有该角色的账户也能够授予和撤销它。

这种机制可以用来创建类似于组织图表的复杂权限结构，但它也提供了一种简单的应用管理方式。AccessControl包括一个特殊的角色，角色标识符为0，称为DEFAULT_ADMIN_ROLE，**它充当所有角色的默认管理员角色**。拥有此角色的账户将能够管理任何其他角色，除非使用*_set_role_admin*选择新的管理员角色。

让我们看一下ERC20代币示例，这次利用默认的管理员角色：
```
const MINTER_ROLE: felt252 = selector!('MINTER_ROLE');
const BURNER_ROLE: felt252 = selector!('BURNER_ROLE');

#[starknet::contract]
mod MyContract {
    use openzeppelin::access::accesscontrol::AccessControl::InternalImpl::assert_only_role;
    use openzeppelin::access::accesscontrol::AccessControl;
    use openzeppelin::access::accesscontrol::DEFAULT_ADMIN_ROLE;
    use openzeppelin::token::erc20::ERC20;
    use starknet::ContractAddress;
    use super::{MINTER_ROLE, BURNER_ROLE};

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(
        ref self: ContractState,
        name: felt252,
        symbol: felt252,
        initial_supply: u256,
        recipient: ContractAddress,
        admin: ContractAddress
    ) {
        // 与ERC20相关的初始化
        let mut erc20_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref erc20_state, name, symbol);
        ERC20::InternalImpl::_mint(ref erc20_state, recipient, initial_supply);

        // 与访问控制相关的初始化
        let mut access_state = AccessControl::unsafe_new_contract_state();
        AccessControl::InternalImpl::initializer(ref access_state);
        AccessControl::InternalImpl::_grant_role(
            ref access_state,
            DEFAULT_ADMIN_ROLE,
            admin
        );
    }

    // 这个函数只能由铸币者调用
    #[external(v0)]
    fn mint(ref self: ContractState, recipient: ContractAddress, amount: u256) {
        let access_state = AccessControl::unsafe_new_contract_state();
        assert_only_role(@access_state, MINTER_ROLE);

        let mut erc20_state = AccessControl::unsafe_new_contract_state();
        ERC20::InternalImpl::_mint(ref erc20_state, recipient, amount);
    }

    // 这个功能只能由销毁调用
    #[external(v0)]
    fn burn(ref self: ContractState, account: ContractAddress, amount: u256) {
        let access_state = AccessControl::unsafe_new_contract_state();
        assert_only_role(@access_state, BURNER_ROLE);

        let mut erc20_state = AccessControl::unsafe_new_contract_state();
        ERC20::InternalImpl::_burn(ref erc20_state, account, amount);
    }

    // 这些功能只能由角色管理员调用
    #[external(v0)]
    fn grant_role(ref self: ContractState, role: felt252, account: ContractAddress) {
        let mut unsafe_state = AccessControl::unsafe_new_contract_state();
        AccessControl::AccessControlImpl::grant_role(ref unsafe_state, role, account);
    }
    #[external(v0)]
    fn revoke_role(ref self: ContractState, role: felt252, account: ContractAddress) {
        let mut unsafe_state = AccessControl::unsafe_new_contract_state();
        AccessControl::AccessControlImpl::revoke_role(ref unsafe_state, role, account);
    }
}
```

请注意，与前面的例子不同，没有任何账户被授予 'minter' 或 'burner' 角色。然而，由于这些角色的管理员角色是默认的管理员角色，并且该角色已经被授予给 'admin'，因此该账户可以调用 grant_role 来授予铸币或燃烧权限，并调用 revoke_role 来撤销这些权限。

动态角色分配通常是一种理想的属性，例如在参与者的信任度可能随时间变化的系统中。它也可以用来支持如[KYC](https://en.wikipedia.org/wiki/Know_your_customer)这样的用例，其中角色承担者的列表可能无法预先知道，或者在单个交易中包含可能过于昂贵。

### 创建角色标识符
在Solidity实现的AccessControl中，合约通常将角色的[keccak256哈希](https://docs.soliditylang.org/en/latest/units-and-global-variables.html?highlight=keccak256#mathematical-and-cryptographic-functions)作为角色标识符。

例如:
```
bytes32 public constant SOME_ROLE = keccak256("SOME_ROLE")
```

这些标识符占用32字节（256位）。

Cairo字段元素（felt252）最多存储252位。由于这种差异，该库对合约应如何创建标识符保持中立态度。一些值得考虑的想法：

* 使用[sn_keccak](https://docs.starknet.io/documentation/architecture_and_concepts/Cryptography/hash-functions/#starknet_keccak)代替。

* 使用Cairo友好的哈希算法，如Poseidon，这些算法在[Cairo corelib](https://github.com/starkware-libs/cairo/blob/main/corelib/src/poseidon.cairo)中实现。

> TIP
可以使用selector!宏在Cairo中计算sn_keccak。

### 接口
这是AccessControl实现的完整接口:
```
trait IAccessControl {
    /// 返回账户是否具有该角色。
    fn has_role(role: felt252, account: ContractAddress) -> bool;

    /// 返回控制`role`的管理员角色。
    fn get_role_admin(role: felt252) -> felt252;

    /// `account`授予`role`。
    fn grant_role(role: felt252, account: ContractAddress);

    /// 从`account`撤销`role`。
    fn revoke_role(role: felt252, account: ContractAddress);

    /// 撤销自己的`role`。
    fn renounce_role(role: felt252, account: ContractAddress);
}
```

AccessControl还允许你从调用账户中放弃角色。该方法需要一个账户作为输入，作为额外的安全措施，以确保你不会从意外的账户中放弃角色。