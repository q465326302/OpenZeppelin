# Access Control
访问控制——即“谁被允许做这件事”——在智能合约世界中非常重要。您的合约的访问控制可能会管理谁可以铸造代币、投票支持提案、冻结转账等等。因此，了解如何实现它非常关键，以免其他人[窃取您的整个系统](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)。

## 所有权和Ownable
最常见和基本的访问控制形式是所有权的概念：有一个帐户是合约的所有者，可以在其中执行行政任务。对于具有单个行政用户的合约，这种方法是完全合理的。

OpenZeppelin Contracts提供了*Ownable*，用于在您的合约中实现所有权。
```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

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
默认情况下，*Ownable*合约的所有者是部署它的账户，这通常是您想要的。

Ownable还允许您：

* *将所有权从所有者账户*转移到新账户，以及

* *放弃所有权*，让所有者放弃这种管理特权，这是初始阶段集中管理结束后的常见模式。

>WARNING
删除所有者将意味着仅由onlyOwner保护的管理任务将不再可调用！

请注意，**合约也可以是另一个合约的所有者**！这打开了使用[Gnosis Safe](https://gnosis-safe.io/)、[Aragon DAO](https://aragon.org/)或您创建的完全自定义合约的大门。

通过这种方式，您可以使用组合性将额外的访问控制复杂性层添加到您的合约中。例如，您可以使用由项目领导运行的2-of-3多签名，而不是使用单个常规以太坊帐户（外部拥有帐户或EOA）作为所有者。在空间中的知名项目，例如[MakerDAO](https://makerdao.com/)，使用类似于此的系统。

## 基于角色的访问控制
虽然所有权的简单性对于简单的系统或快速原型设计可能很有用，但通常需要不同级别的授权。您可能希望某个帐户有权禁止用户使用系统，但不能创建新代币。[基于角色的访问控制（RBAC）](https://en.wikipedia.org/wiki/Role-based_access_control)在这方面提供了灵活性。

本质上，我们将定义多个角色，每个角色允许执行不同的操作集。一个帐户可能有“调解员”、“铸造者”或“管理员”角色，然后您将检查这些角色，而不是仅使用onlyOwner。这个检查可以通过onlyRole修饰符强制执行。另外，您将能够为如何授予角色、撤销角色等定义规则。

大多数软件使用基于角色的访问控制系统：有些用户是普通用户，有些可能是监管人员或经理，还有一些人通常具有管理特权。

### 使用AccessControl
OpenZeppelin Contracts提供了*AccessControl*，用于实现基于角色的访问控制。它的使用很简单：对于要定义的每个角色，您将创建一个新的角色标识符，用于授予、撤销和检查帐户是否具有该角色。

以下是在*ERC20代币*中使用AccessControl定义“铸造者”角色的简单示例，该角色允许具有它的帐户创建新代币：
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    // Create a new role identifier for the minter role
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");

    constructor(address minter) ERC20("MyToken", "TKN") {
        // Grant the minter role to a specified account
        _grantRole(MINTER_ROLE, minter);
    }

    function mint(address to, uint256 amount) public {
        // Check that the calling account has the minter role
        require(hasRole(MINTER_ROLE, msg.sender), "Caller is not a minter");
        _mint(to, amount);
    }
}
```

> NOTE
在使用*AccessControl*之前，请确保您完全理解它的工作原理，或者从本指南中复制粘贴示例之前，请确保您已经完全理解它的工作原理。

尽管清晰明确，但这并不是我们无法通过Ownable实现的任何内容。事实上，AccessControl的优势在于需要实现精细权限的场景，可以通过定义多个角色来实现。

让我们通过定义“burner”角色来增强我们的ERC20令牌示例，该角色允许帐户销毁代币，并使用onlyRole修饰符：
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

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

通过这种方式分离关注点，可以实现比简单的所有权访问控制更精细的权限级别。限制系统的每个组件所能做的事情被称为[最小权限原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，这是一种良好的安全实践。请注意，每个账户仍然可以拥有多个角色，如果需要的话。

### 授予和撤销角色
上面的ERC20代币示例使用了_grantRole，这是一个内部函数，用于在构建过程中以编程方式分配角色。但是，如果我们稍后想要将“铸造者”角色授予其他账户，该怎么办？

默认情况下，**具有角色的账户无法授予或撤销它**。拥有角色只是使hasRole检查通过。要动态地授予和撤销角色，您需要从角色的管理员那里获得帮助。

每个角色都有一个关联的管理员角色，该角色授予调用grantRole和revokeRole函数的权限。如果调用账户具有相应的管理员角色，则可以使用这些函数来授予或撤销角色。多个角色可以具有相同的管理员角色，以使管理更容易。一个角色的管理员甚至可以是它自己，这将导致具有该角色的账户也能够授予和撤销它。

这种机制可以用于创建类似组织结构图的复杂权限结构，但它也为管理简单应用程序提供了一种简便的方法。AccessControl包括一个特殊的角色，称为DEFAULT_ADMIN_ROLE，它作为所有角色的**默认管理员角色**。具有此角色的账户将能够管理任何其他角色，除非使用_setRoleAdmin选择新的管理员角色。

由于默认情况下它是所有角色的管理员，实际上它还是它自己的管理员，因此该角色存在重大风险。为了减轻这种风险，我们提供了*AccessControlDefaultAdminRules*，这是AccessControl的推荐扩展，增加了一些强制性安全措施，用于此角色：管理员受限于单个账户，具有两步转移过程，并在步骤之间有延迟。

让我们看一下ERC20代币示例，这次利用默认管理员角色：
```
// contracts/MyToken.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20, AccessControl {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant BURNER_ROLE = keccak256("BURNER_ROLE");

    constructor() ERC20("MyToken", "TKN") {
        // Grant the contract deployer the default admin role: it will be able
        // to grant and revoke any roles
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

请注意，与之前的示例不同，没有任何帐户被授予“铸造者”或“销毁者”角色。然而，因为这些角色的管理员角色是默认的管理员角色，并且该角色被授予给了msg.sender，因此同一个帐户可以调用grantRole授予铸造或销毁权限，并调用revokeRole来删除它。

动态角色分配通常是一种理想的属性，例如在信任参与者的情况下可能会随时间变化。它也可以用于支持[KYC](https://en.wikipedia.org/wiki/Know_your_customer)等用例，其中角色持有者的列表可能不是预先知道的，或者可能在单个事务中包含的成本过高。

### 查询特权帐户
因为帐户可能会动态地*授予和撤销角色*，所以并不总是能够确定哪些帐户持有特定的角色。这很重要，因为它允许证明关于系统的某些属性，例如管理帐户是多重签名或DAO，或者某个角色已从所有用户中删除，从而禁用任何相关的功能。

在幕后，AccessControl使用EnumerableSet，Solidity的映射类型的更强大的变体，它允许键枚举。getRoleMemberCount可以用于检索具有特定角色的帐户数量，然后可以调用getRoleMember来获取这些帐户的地址。
```
const minterCount = await myToken.getRoleMemberCount(MINTER_ROLE);

const members = [];
for (let i = 0; i < minterCount; ++i) {
    members.push(await myToken.getRoleMember(MINTER_ROLE, i));
}
```

## 延迟操作
访问控制对于防止未经授权的访问关键功能至关重要。这些功能可能用于铸造代币、冻结转账或执行完全更改智能合约逻辑的升级。虽然* Ownable* 和* AccessControl *可以防止未经授权的访问，但它们无法解决管理员恶意攻击其自己的系统以损害其用户的问题。

这就是 *TimelockController* 要解决的问题。

*TimelockController* 是一个由提议者和执行者管理的代理。当将其设置为智能合约的所有者/管理员/控制器时，它确保提议者下达的任何维护操作都要受到延迟。这种延迟可以通过给用户时间来审查维护操作并根据自己的最佳利益退出系统来保护智能合约的用户。

### 使用 TimelockController
默认情况下，部署 *TimelockController* 的地址获得 timelock 的管理特权。该角色授予分配提议者、执行者和其他管理员的权利。

配置 *TimelockController* 的第一步是分配至少一个提议者和一个执行者。这些可以在构建期间分配，也可以由具有管理员角色的任何人稍后分配。这些角色不是互斥的，这意味着一个账户可以拥有这两个角色。

角色使用 *AccessControl* 接口进行管理，每个角色的 bytes32 值可通过 ADMIN_ROLE、PROPOSER_ROLE 和 EXECUTOR_ROLE 常量访问。

在 AccessControl 上构建了一个附加功能：将 executor 角色授予 address(0) 就开放了任何人在 timelock 过期后执行提议的访问权限。尽管这个功能很有用，但应该谨慎使用。

此时，分配了提议者和执行者角色，timelock 可以执行操作。

下一步是可选的，即部署者放弃其管理特权，让 timelock 自我管理。如果部署者决定这样做，所有后续维护操作，包括分配新的提议者/调度程序或更改 timelock 持续时间，都必须遵循 timelock 工作流程。这将将 timelock 的治理与附加到 timelock 的合约的治理联系起来，并对 timelock 维护操作强制实施延迟。

>WARNING
如果部署者放弃自己的管理权而将其转移给 timelock 本身，则分配新的提议者或执行者将需要一个 timelocked 操作。这意味着如果负责这两个角色的账户不可用，那么整个合约（以及它控制的任何合约）将无限期被锁定。

分配了提议者和执行者角色并让 timelock 管理自己的情况下，现在可以将任何合约的所有权/控制权转移到 timelock。

> TIP
建议的配置是将这两个角色授予安全治理合约，例如 DAO 或多重签名，并额外将执行者角色授予一些负责帮助维护操作的人持有的 EOAs。这些钱包无法接管 timelock 的控制权，但可以帮助平滑工作流程。


### 最小延迟
由 *TimelockController* 执行的操作不受固定延迟的限制，而是受到最小延迟的限制。一些重大更新可能需要更长的延迟。例如，如果几天的延迟足以让用户审计铸造操作，那么在安排智能合约升级时使用几周甚至几个月的延迟是有意义的。

最小延迟（可通过 *getMinDelay* 方法访问）可以通过调用 *updateDelay* 函数进行更新。请注意，只有 timelock 本身可以访问此函数，这意味着该维护操作必须通过 timelock 本身进行。