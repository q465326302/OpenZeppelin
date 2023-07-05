# Access Control
在智能合约的世界中，访问控制——即“谁可以做这件事”——非常重要。合约的访问控制可能涉及谁可以铸造代币、对提案投票、冻结转账等许多事情。因此，了解如何实施访问控制非常关键，以免被[其他人窃取整个系统](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)。

所有权和Ownable
最常见和基本的访问控制形式是所有权的概念：有一个账户是合约的所有者，可以对其进行管理任务。对于只有一个管理用户的合约来说，这种方法是完全合理的。

OpenZeppelin提供了*Ownable*来实现合约的所有权。
```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/access/Ownable.sol";

contract MyContract is Ownable {
    function normalThing() public {
        // anyone can call this normalThing()
    }

    function specialThing() public onlyOwner {
        // only the owner can call specialThing()!
    }
}
```
默认情况下，*Ownable*合约的所有者是部署它的账户，这通常是你想要的。

Ownable还可以让你：

* 将*所有权*从所有者账户转移给新账户，以及

* 放弃*所有权*，使所有者放弃这个管理特权，在初始阶段的集中管理结束后，这是一个常见的模式。

> WARNING
完全移除所有者意味着只有所有者才能调用的管理任务将无法调用！

请注意，一个合约也可以是另一个合约的所有者！这为使用[Gnosis Multisig](https://github.com/gnosis/MultiSigWallet)或[Gnosis Safe](https://safe.gnosis.io/)、[Aragon DAO](https://aragon.org/)、[ERC725/uPort](https://www.uport.me/)身份合约或您创建的完全自定义合约打开了大门。

通过这种方式，您可以使用组合性将额外的访问控制复杂性层添加到您的合约中。例如，您可以使用由项目负责人运行的2-of-3多签名，而不是将单个常规以太坊账户（Externally Owned Account，EOA）作为所有者。像[MakerDAO](https://makerdao.com/)这样的知名项目使用类似的系统。

## 基于角色的访问控制
虽然所有权的简单性对于简单的系统或快速原型设计很有用，但通常需要不同级别的授权。您可能希望一个账户有权限禁止用户进入系统，但不能创建新的代币。基于角色的[访问控制（RBAC）](https://en.wikipedia.org/wiki/Role-based_access_control)在这方面提供了灵活性。

本质上，我们将定义多个角色，每个角色允许执行不同的操作集。一个账户可能有' moderator '、' minter '或' admin '角色，您将检查这些角色而不是仅使用onlyOwner。另外，您还可以定义授予角色、撤销角色等的规则。

大多数软件使用的是基于角色的访问控制系统：一些用户是普通用户，一些可能是主管或经理，还有一些可能具有管理员特权。

### 使用AccessControl
OpenZeppelin Contracts提供了*AccessControl*来实现基于角色的访问控制。它的使用非常简单：对于您想要定义的每个角色，您将创建一个新的角色标识符，用于授予、撤销和检查一个账户是否具有该角色。

以下是在*ERC20代币*中使用AccessControl定义'minter'角色的简单示例，允许具有该角色的账户创建新的代币：
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    // Create a new role identifier for the minter role
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor(address minter) public ERC20("MyToken", "TKN") {
        // Grant the minter role to a specified account
        _setupRole(MINTER_ROLE, minter);
    }

    function mint(address to, uint256 amount) public {
        // Check that the calling account has the minter role
        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
        _mint(to, amount);
    }
}
```
> NOTE
在使用*AccessControl*之前，请确保充分理解其工作原理，或者复制粘贴本指南中的示例。

虽然清晰明确，但这并不是我们无法通过Ownable实现的任何功能。事实上，AccessControl在需要细粒度权限的场景中表现出色，可以通过定义多个角色来实现。

让我们通过定义一个“burner”角色来增强我们的ERC20代币示例，该角色允许账户销毁代币：
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor(address minter, address burner) public ERC20("MyToken", "TKN") {
        _setupRole(MINTER_ROLE, minter);
        _setupRole(BURNER_ROLE, burner);
    }

    function mint(address to, uint256 amount) public {
        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) public {
        require(hasRole(BURNER_ROLE, msg.sender), "Caller is not a burner");
        _burn(from, amount);
    }
}
```
如此干净！通过以这种方式分割关注点，可以实现比简单的所有权访问控制方法更细粒度的权限级别。限制系统的每个组件能够执行的操作被称为[最小权限原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，这是一种良好的安全实践。请注意，每个账户仍然可以拥有多个角色，如果需要的话。

### 授予和撤销角色
上面的ERC20代币示例使用了一个内部函数_setupRole，该函数在编程方式分配角色（例如在构造过程中）时非常有用。但是，如果以后我们想要将“铸币者”角色授予其他账户怎么办？

默认情况下，**拥有某个角色的账户无法将其授予或从其他账户撤销**：拥有角色只是使hasRole检查通过而已。要动态地授予和撤销角色，您需要获得该角色的管理员的帮助。

每个角色都有一个关联的管理员角色，该角色授予调用grantRole和revokeRole函数的权限。如果调用账户具有相应的管理员角色，则可以使用这些函数来授予和撤销角色。为了更容易管理，多个角色可以具有相同的管理员角色。一个角色的管理员甚至可以是该角色本身，这将使具有该角色的账户能够同时授予和撤销它。

这个机制可以用来创建类似组织结构图的复杂权限结构，但它也为管理更简单的应用程序提供了一种简单的方法。AccessControl包括一个特殊的角色DEFAULT_ADMIN_ROLE，它作为所有角色的默认管理员角色。具有此角色的账户将能够管理任何其他角色，除非使用_setRoleAdmin选择新的管理员角色。

让我们再次看一下ERC20代币的示例，这次利用默认管理员角色：
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor() public ERC20("MyToken", "TKN") {
        // Grant the contract deployer the default admin role: it will be able
        // to grant and revoke any roles
        _setupRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public {
        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) public {
        require(hasRole(BURNER_ROLE, msg.sender), "Caller is not a burner");
        _burn(from, amount);
    }
}
```
请注意，与之前的示例不同，没有任何账户被授予“铸币者”或“销毁者”角色。然而，因为这些角色的管理角色是默认的管理角色，并且该角色被授予了msg.sender，同样的账户可以调用grantRole来授予铸币或销毁权限，并调用revokeRole来取消权限。

动态角色分配通常是一种理想的属性，例如在对参与者的信任可能随时间变化的系统中。它还可以用于支持诸如[KYC](https://en.wikipedia.org/wiki/Know_your_customer)之类的用例，其中角色承载人的列表可能事先不知道，或者在单个交易中包含这些列表可能过于昂贵。

### 查询特权账户
由于账户可能动态地*授予和撤销*角色，因此不总是能够确定哪些账户拥有特定的角色。这一点很重要，因为它允许证明关于系统的某些属性，例如管理账户是多重签名或DAO，或者某个角色已从所有用户中移除，从而有效地禁用任何相关功能。

在底层，AccessControl使用了EnumerableSet，这是Solidity的映射类型的更强大的变体，它允许键的枚举。getRoleMemberCount可以用来检索具有特定角色的账户数量，然后可以调用getRoleMember来获取每个这些账户的地址。
```
const minterCount = await myToken.getRoleMemberCount(MINTER_ROLE);

const members = [];
for (let i = 0; i < minterCount; ++i) {
    members.push(await myToken.getRoleMember(MINTER_ROLE, i));
}
```

## 延迟操作
访问控制对于防止未经授权访问关键功能至关重要。这些功能可能用于铸造代币、冻结转账或执行完全更改智能合约逻辑的升级。虽然*Ownable*和*AccessControl*可以防止未经授权的访问，但它们不能解决管理员恶意攻击自己系统以损害用户利益的问题。

这就是*TimelockControler*要解决的问题。

*TimelockControler*是一个由提议者和执行者管理的代理。当将其设置为智能合约的所有者/管理员/控制器时，它确保提议者下达的维护操作受到延迟的限制。这种延迟保护了智能合约的用户，使他们有时间审查维护操作，并在认为有必要时退出系统。

### 使用TimelockControler
默认情况下，部署*TimelockControler*的地址获得了对定时锁的管理权限。该角色赋予了分配提议者、执行者和其他管理员的权利。

配置*TimelockControler*的第一步是分配至少一个提议者和一个执行者。这些角色可以在构造过程中或稍后由具有管理员角色的任何人分配。这些角色不是排他性的，也就是说一个账户可以同时拥有这两个角色。

角色使用*AccessControl*接口进行管理，每个角色的bytes32值可通过ADMIN_ROLE、PROPOSER_ROLE和EXECUTOR_ROLE常量访问。

在AccessControl之上还有一个附加功能：将提议者或执行者角色分配给address(0)可以向任何人开放访问权限。这个功能在测试和某些情况下对于执行者角色可能很有用，但是需要小心使用，因为它是危险的。

此时，既有提议者又有执行者被分配，定时锁可以执行操作。

下一步是可选的，部署者可以放弃其管理权限，使定时锁自我管理。如果部署者决定这样做，所有进一步的维护操作，包括分配新的提议者/调度器或更改定时锁持续时间，都需要遵循定时锁的工作流程。这将将定时锁的治理与附属于定时锁的合约的治理联系起来，并对定时锁维护操作进行延迟。

> WARNING
如果部署者放弃了对定时锁自己的管理权，将新的提议者或执行者分配给定时锁将需要一个定时锁操作。这意味着如果负责这两个角色的账户不可用，那么整个合约（以及它控制的任何合约）将无限期地被锁定。

在分配了提议者和执行者角色并且定时锁负责自己的管理后，您现在可以将任何合约的所有权/控制权转移到定时锁。

> TIP
建议的配置是将这两个角色都授予一个安全的治理合约，如DAO或多签，同时还将执行者角色授予一些由负责协助维护操作的人持有的EOA。这些钱包不能接管定时锁的控制，但它们可以帮助平滑工作流程。

### 最小延迟
由*TimelockControler*执行的操作不受固定延迟的限制，而是受到最小延迟的限制。一些重大更新可能需要更长的延迟。例如，如果仅几天的延迟足以让用户审计铸币操作，那么在安排智能合约升级时使用几周甚至几个月的延迟是有意义的。

最小延迟（通过*getMinDelay*方法可访问）可以通过调用*updateDelay*函数进行更新。请注意，只有定时锁本身才能访问此函数，这意味着此维护操作必须通过定时锁本身进行。
