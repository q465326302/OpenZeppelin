# Access Control
在智能合约的世界中，访问控制即“谁被允许做这件事”非常重要。智能合约的访问控制可以控制谁可以铸造代币、对提案进行投票、冻结转账等等。因此，了解如何实施访问控制非常关键，以免被其他人[窃取整个系统](https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7)。

## 所有权和Ownable

最常见和基本的访问控制形式是所有权的概念：一个帐户是合约的所有者，可以在其上执行管理任务。这种方法对于只有一个管理员用户的合约是完全合理的。

OpenZeppelin提供了[Ownable](./API/Ownership.md#ownable)来实现合约的所有权。

```
pragma solidity ^0.5.0;

import "@openzeppelin/contracts/ownership/Ownable.sol";

contract MyContract is Ownable {
    function normalThing() public {
        // anyone can call this normalThing()
    }

    function specialThing() public onlyOwner {
        // only the owner can call specialThing()!
    }
}
```

默认情况下，Ownable合约的[owner](./API/Ownership.md#owner-→-address)是部署它的账户，这通常是您想要的。

Ownable还允许您：

* 将[transferOwnership](./API/Ownership.md#transferownershipaddress-newowner)从所有者账户转移给新账户，以及

* 放弃[transferOwnership](./API/Ownership.md#transferownershipaddress-newowner)，让所有者放弃这种管理特权，在初始阶段的集中管理结束后，这是一种常见的模式。

> WARNING
完全删除所有者将意味着只有所有者保护的管理任务将不再可调用！

请注意，一个合约也可以成为另一个合约的所有者！这为您使用例如[Gnosis Multisig](https://github.com/gnosis/MultiSigWallet)或[Gnosis Safe](https://safe.gnosis.io/)、[Aragon DAO](https://aragon.org/)、[ERC725/uPort](https://www.uport.me/)身份合约或您创建的完全自定义合约打开了大门。

通过这种方式，您可以使用组合性为您的合约添加额外的访问控制复杂性层次。例如，您可以使用一个由项目负责人运行的2-of-3多签名来代替一个普通的以太坊账户（外部拥有账户，EOA）作为所有者。像[MakerDAO](https://makerdao.com/)这样的知名项目使用类似于这个系统的系统。

## 基于角色的访问控制
虽然所有权的简单性对于简单的系统或快速原型设计非常有用，但通常需要不同级别的授权。一个账户可能能够禁止用户访问系统，但不能创建新的代币。基于角色的访问控制（RBAC）在这方面提供了灵活性。

本质上，我们将定义多个角色，每个角色允许执行不同的操作集。您将在某些地方使用onlyAdminRole，而在其他地方使用onlyModeratorRole，而不是在每个地方都使用onlyOwner。此外，您还可以定义账户如何被分配角色、转移角色等规则。

大多数软件开发使用基于角色的访问控制系统：一些用户是普通用户，一些用户可能是主管或经理，而少数用户通常具有管理员特权。

### 使用角色
OpenZeppelin提供了[Roles](./API/Access.md#roles)来实现基于角色的访问控制。它的使用非常简单：对于要定义的每个角色，您将存储一个类型为Role的变量，该变量将保存具有该角色的账户列表。

以下是在[ERC20代币](./Tokens/ERC20/ERC20.md中使用Roles的简单示例：我们将定义两个角色，即铸币者（minters）和销毁者（burners），它们将能够铸造新代币和销毁代币，分别。
```
pragma solidity ^0.5.0;

import "@openzeppelin/contracts/access/Roles.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20Detailed.sol";

contract MyToken is ERC20, ERC20Detailed {
    using Roles for Roles.Role;

    Roles.Role private _minters;
    Roles.Role private _burners;

    constructor(address[] memory minters, address[] memory burners)
        ERC20Detailed("MyToken", "MTKN", 18)
        public
    {
        for (uint256 i = 0; i < minters.length; ++i) {
            _minters.add(minters[i]);
        }

        for (uint256 i = 0; i < burners.length; ++i) {
            _burners.add(burners[i]);
        }
    }

    function mint(address to, uint256 amount) public {
        // Only minters can mint
        require(_minters.has(msg.sender), "DOES_NOT_HAVE_MINTER_ROLE");

        _mint(to, amount);
    }

    function burn(address from, uint256 amount) public {
        // Only burners can burn
        require(_burners.has(msg.sender), "DOES_NOT_HAVE_BURNER_ROLE");

       _burn(from, amount);
    }
}
```

如此整洁！通过以这种方式分离关注点，可以实现比简单的所有权访问控制方法更精细级别的权限控制。请注意，如果需要，一个账户可以拥有多个角色。

OpenZeppelin广泛使用角色，使用预定义的合约来编码每个特定角色的规则。一些例子包括：[ERC20Mintable](./API/ERC20.md#erc20mintable)使用[MinterRole](./API/Access.md#minterrole)来确定谁可以铸造代币，而[WhitelistCrowdsale](./API/Crowdsale.md#whitelistcrowdsale)则同时使用[WhitelistAdminRole](./API/Access.md#whitelistadminrole)和[WhitelistedRole](./API/Access.md#whitelistedrole)来创建一组可以购买代币的账户。

这种灵活性允许实现有趣的设置：例如，[MintedCrowdsale](./API/Crowdsale.md#mintedcrowdsale)期望被赋予ERC20Mintable的MinterRole以便正常工作，但代币合约也可以扩展[ERC20Pausable](./API/ERC20.md#erc20pausable)并将[PauserRole](./API/Access.md#pauserrole)分配给作为应急机制的DAO，以防发现合约代码中的漏洞。限制系统的每个组件能够做什么被称为[最小权限原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，这是一个良好的安全实践。

## 在OpenZeppelin中的使用
您会注意到，OpenZeppelin的合约中没有使用Ownable。Roles是首选的解决方案，因为它为库的使用者提供了足够的灵活性，以使提供的合约适应他们的需求。

然而，有一些情况下，合约之间存在直接关系。例如，[RefundableCrowdsale](./API/Crowdsale.md#refundablecrowdsale)在构造函数中部署一个[RefundEscrow](./API/Payment.md#refundescrow)来保存其资金。对于这些情况，我们将使用[Secondary](./API/Ownership.md#secondary)来创建一个辅助合约，允许主合约对其进行管理。您也可以将这些看作是辅助合约。