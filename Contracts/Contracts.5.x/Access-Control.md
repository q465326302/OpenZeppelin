# Access Control
访问控制——即"谁被允许做这件事"——在智能合约的世界中非常重要。你的合约的访问控制可能决定谁可以铸造代币，对提案进行投票，冻结转账，以及许多其他事情。因此，理解你如何实施它至关重要，以免别人[窃取你的整个系统](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)。

## 所有权和Ownable
最常见和基本的访问控制形式是所有权的概念：有一个账户是合约的所有者，可以在其上执行管理任务。对于只有一个管理用户的合约，这种方法非常合理。

OpenZeppelin合约提供了*Ownable*，用于在你的合约中实现所有权。
```
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

import {Ownable} from "@openzeppelin/contracts/access/Ownable.sol";

contract MyContract is Ownable {
    constructor(address initialOwner) Ownable(initialOwner) {}

    function normalThing() public {
        // 任何人都可以调用这个normalThing()
    }

    function specialThing() public onlyOwner {
        // 只有所有者才能调用specialThing()！
    }
}
```

默认情况下，Ownable合约的*所有者*是部署它的账户，这通常正是你想要的。

Ownable还允许你：

* 从*所有者账户转移所有权*到一个新的账户，以及

* *所有者放弃所有权*，这是在初始阶段的集中式管理结束后常见的模式。

> WARNING
完全移除所有者将意味着只有Owner才能调用的管理任务将无法再被调用！

请注意，**一个合约也可以是另一个合约的所有者**！这为你使用例如[Gnosis Safe](https://gnosis-safe.io/)，[Aragon DAO](https://aragon.org/)，或者你自己创建的完全自定义合约打开了大门。

通过这种方式，你可以使用组合性为你的合约添加额外的访问控制复杂性层。你可以使用2-of-3多签名由你的项目负责人运行，而不是使用一个单一的常规以太坊账户（外部拥有的账户，或EOA）作为所有者。例如，[MakerDAO](https://makerdao.com/)等知名项目就使用了类似的系统。

## 基于角色的访问控制
虽然所有权的简单性对于简单的系统或快速原型设计可能很有用，但通常需要不同级别的授权。你可能希望一个账户有权限从系统中禁止用户，但不能创建新的代币。[基于角色的访问控制（RBAC）](https://en.wikipedia.org/wiki/Role-based_access_control)在这方面提供了灵活性。

本质上，我们将定义多个角色，每个角色都被允许执行不同的操作集。例如，一个账户可能有'moderator'，'minter'或'admin'的角色，你将检查这些角色，而不是简单地使用onlyOwner。这个检查可以通过onlyRole修饰符来强制执行。另外，你将能够定义规则，规定如何授予账户角色，如何撤销角色等。

大多数软件使用的访问控制系统都是基于角色的：一些用户是普通用户，一些可能是主管或经理，少数用户通常会有管理权限。

### 使用AccessControl
OpenZeppelin合约提供了*AccessControl*用于实现基于角色的访问控制。它的使用非常直接：对于你想要定义的每个角色，你将创建一个新的角色标识符，用于授予、撤销和检查账户是否具有该角色。

以下是一个简单的例子，使用AccessControl在*ERC20代币*中定义一个'minter'角色，允许拥有它的账户创建新的代币。
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    // 为铸币者角色创建一个新的角色标识符
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor(address minter) ERC20("MyToken", "TKN") {
        // 将铸币者角色授予指定账户
        _grantRole(MINTER_ROLE, minter);
    }

    function mint(address to, uint256 amount) public {
        // 检查呼叫账户的是否具有铸币者角色
        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
        _mint(to, amount);
    }
}
```

> NOTE
在你在系统上使用*AccessControl*或从本指南中复制粘贴示例之前，请确保你完全理解AccessControl的工作原理。

虽然这很清晰和明确，但这并不是我们不能通过Ownable来实现的。实际上，AccessControl在需要精细权限的场景中表现得最好，这可以通过定义多个角色来实现。

让我们通过定义一个'burner'角色来增强我们的ERC20代币示例，这个角色让账户可以销毁代币，并使用onlyRole修饰符。
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor(address minter, address burner) ERC20("MyToken", "TKN") {
        _grantRole(MINTER_ROLE, minter);
        _grantRole(BURNER_ROLE, burner);
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) public onlyRole(BURNER_ROLE) {
        _burn(from, amount);
    }
}
```

如此明了！通过这种方式分割关注点，可以实现比简单的所有权访问控制方式更精细的权限级别。限制系统的每个组件能够做什么被称为[最小权限原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，这是一种良好的安全实践。请注意，每个账户仍然可以拥有多个角色，如果需要的话。

### 授予和撤销角色
上面的ERC20代币示例使用了_grantRole，这是一个在程序化分配角色时（例如在构建过程中）非常有用的内部函数。但是，如果我们稍后想要授予更多账户“铸币者”角色，该怎么办呢？

默认情况下，**拥有角色的账户不能授予或撤销其他账户的角色**：拥有角色只是使hasRole检查通过。要动态地授予和撤销角色，你将需要角色的管理员的帮助。

每个角色都有一个关联的管理员角色，它授予调用grantRole和revokeRole函数的权限。如果调用账户拥有相应的管理员角色，就可以使用这些函数授予或撤销角色。多个角色可以有相同的管理员角色，以便于管理。角色的管理员甚至可以是角色本身，这将导致拥有该角色的账户也能够授予和撤销它。

这种机制可以用来创建类似于组织图的复杂权限结构，但它也提供了一种简单的应用管理方式。AccessControl包括一个特殊的角色，称为DEFAULT_ADMIN_ROLE，它充当**所有角色的默认管理员角色**。拥有此角色的账户将能够管理任何其他角色，除非使用_setRoleAdmin选择新的管理员角色。

由于它默认是所有角色的管理员，实际上它也是自己的管理员，所以这个角色带有很大的风险。为了降低这种风险，我们提供了*AccessControlDefaultAdminRules*，这是AccessControl的推荐扩展，为此角色添加了一些强制性的安全措施：管理员被限制为单个账户，转移程序需要两步，且两步之间有延迟。

让我们再看一下ERC20代币示例，这次利用默认的管理员角色：
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor() ERC20("MyToken", "TKN") {
        // 授予合约部署者默认的管理员角色：它将能够授予和撤销任何角色。
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) public onlyRole(BURNER_ROLE) {
        _burn(from, amount);
    }
}
```

请注意，与之前的例子不同，没有任何账户被授予 'minter' 或 'burner' 角色。然而，由于这些角色的管理员角色是默认的管理员角色，并且该角色被授予给了msg.sender，因此同一个账户可以调用grantRole来给予铸币或燃烧权限，也可以调用revokeRole来移除这些权限。

动态角色分配通常是一个令人期待的属性，例如在信任度可能随时间变化的系统中。它也可以用来支持[KYC](https://en.wikipedia.org/wiki/Know_your_customer)等用例，其中角色承担者的列表可能无法预先知道，或者在单个交易中包含可能代价过高。

### 查询特权账户
由于账户可能会动态地*授予和撤销角色*，因此并不总是可以确定哪些账户拥有特定的角色。这一点很重要，因为它允许证明系统的某些属性，例如管理员账户是多签名或DAO，或者某个角色已经从所有用户中移除，从而有效地禁用任何相关的功能。

在底层，AccessControl使用了EnumerableSet，这是Solidity的映射类型的一个更强大的变体，允许键的枚举。可以使用getRoleMemberCount来检索拥有特定角色的账户数量，然后可以调用getRoleMember来获取这些账户的地址。
```
const minterCount = await myToken.getRoleMemberCount(MINTER_ROLE);

const members = [];
for (let i = 0; i < minterCount; ++i) {
    members.push(await myToken.getRoleMember(MINTER_ROLE, i));
}
```

## 延迟操作
访问控制对于防止未经授权的访问关键功能是必不可少的。这些功能可能被用于铸造代币，冻结转账或执行完全改变智能合约逻辑的升级。虽然*Ownable*和*AccessControl*可以防止未经授权的访问，但它们并未解决管理员恶意攻击自己的系统，损害用户利益的问题。

这就是*TimelockController*要解决的问题。

*TimelockController*是一个由提议者和执行者管理的代理。当它被设置为智能合约的所有者/管理员/控制器时，它确保由提议者下达的任何维护操作都需要经过一段延迟。这种延迟保护了智能合约的用户，给予他们时间审查维护操作，并在他们认为这样做符合他们的最佳利益时退出系统。

### 使用TimelockController
默认情况下，部署*TimelockController*的地址获得对时间锁的管理权限。这个角色授予分配提议者、执行者和其他管理员的权利。

配置*TimelockController*的第一步是分配至少一个提议者和一个执行者。这些可以在构造过程中分配，或者稍后由拥有管理员角色的任何人分配。这些角色不是互斥的，意味着一个账户可以拥有两个角色。

角色使用*AccessControl*接口管理，每个角色的bytes32值可以通过ADMIN_ROLE、PROPOSER_ROLE和EXECUTOR_ROLE常量访问。

在AccessControl之上有一个额外的功能：将执行者角色赋予address(0)可以让任何人在时间锁过期后执行提案。这个功能虽然有用，但应谨慎使用。

此时，只要分配了提议者和执行者，时间锁就可以执行操作。

下一步可选的操作是部署者放弃其管理权限，让时间锁自我管理。如果部署者决定这样做，所有进一步的维护，包括分配新的提议者/调度者或更改时间锁持续时间，都将必须遵循时间锁工作流。这将时间锁的治理与附加到时间锁的合约的治理联系起来，并对时间锁维护操作强制执行延迟。

> WARNING
如果部署者放弃管理权，转让给时间锁本身，那么分配新的提议者或执行者将需要一个时间锁操作。这意味着，如果负责这两个角色的任何账户变得无法使用，那么整个合约（以及它控制的任何合约）将无限期地被锁定。

在分配了提议者和执行者角色，并由时间锁负责自身的管理后，你现在可以将任何合约的所有权/控制权转让给时间锁。

> TIP
推荐的配置是将两个角色都授予一个安全的治理合约，如DAO或多签，同时将执行者角色授予几个由负责维护操作的人持有的EOAs。这些钱包不能接管时间锁的控制，但他们可以帮助平滑工作流程。

### 最小延迟
由*TimelockController*执行的操作不受固定延迟的限制，而是受最小延迟的限制。一些重大更新可能需要更长的延迟。例如，如果几天的延迟可能足够用户审核一个铸币操作，那么在安排智能合约升级时，使用几周甚至几个月的延迟是有意义的。

最小延迟（可以通过*getMinDelay*方法访问）可以通过调用*updateDelay*函数进行更新。请注意，只有时间锁本身才能访问此函数，这意味着这个维护操作必须通过时间锁本身进行。