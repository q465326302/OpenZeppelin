# Extending Contracts
大多数OpenZeppelin合约预计将通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)来使用：当编写自己的合约时，您将从它们继承。

这是常见的is语法，如在contract MyToken is ERC20中。

> NOTE
与合约不同，Solidity库不是从中继承的，而是依赖于[using for](https://solidity.readthedocs.io/en/latest/contracts.html#using-for)语法。
OpenZeppelin合约有一些库：大多数在*Utils*目录中。

## 覆盖
继承通常用于将父合约的功能添加到您自己的合约中，但这并非它的全部功能。您还可以使用覆盖来更改父合同的某些部分的行为。

例如，假设您想要更改*AccessControl*，使得*revokeRole*不能再被调用。这可以通过覆盖来实现。
```
// contracts/ModifiedAccessControl.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

contract ModifiedAccessControl is AccessControl {
    // Override the revokeRole function
    function revokeRole(bytes32, address) public override {
        revert("ModifiedAccessControl: cannot revoke roles");
    }
}
```

旧的revokeRole然后被我们的覆盖替换，任何对它的调用都将立即恢复。我们不能从合同中删除该功能，但是在所有调用上恢复已经足够好了。

### 调用super
有时你想要扩展父类的行为，而不是直接将其改为其他东西。这就是super的作用。

super关键字将让你调用在父合同中定义的函数，即使它们被覆盖。这种机制可以用来对函数添加额外的检查，发出事件，或者根据你的需要添加功能。

> TIP
关于覆盖如何工作的更多信息，请前往[官方Solidity文档](https://solidity.readthedocs.io/en/latest/contracts.html#index-17)。

这是一个修改过的*AccessControl*版本，其中*revokeRole*不能用来撤销DEFAULT_ADMIN_ROLE:
```
// contracts/ModifiedAccessControl.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {AccessControl} from "@openzeppelin/contracts/access/AccessControl.sol";

contract ModifiedAccessControl is AccessControl {
    function revokeRole(bytes32 role, address account) public override {
        require(
            role != DEFAULT_ADMIN_ROLE,
            "ModifiedAccessControl: cannot revoke default admin role"
        );

        super.revokeRole(role, account);
    }
}
```

在最后的super.revokeRole声明中，将调用AccessControl的原始版本的revokeRole，这是如果没有覆盖在位的话，将会运行的相同的代码。

> NOTE
同样的规则在*AccessControlDefaultAdminRules*中得到了实现和扩展，这是一个扩展，它还增加了对DEFAULT_ADMIN_ROLE的强制性安全措施。

## 安全性
OpenZeppelin Contracts的维护者主要关注的是库中发布的代码的正确性和安全性，以及库的官方扩展与基础合约的组合。

自定义的覆盖，特别是钩子的覆盖，可能会打破一些重要的假设，并在原本安全的代码中引入漏洞。虽然我们尽力确保合约在面对广泛的可能的自定义时仍然保持安全，但这是以最大的努力来做的。虽然我们尝试记录所有重要的假设，但这不应被依赖。自定义的覆盖应该被仔细审查，并根据它们正在定制的合约的源代码进行检查，以便完全理解它们的影响并保证它们的安全性。

函数内部的交互方式不应假设在库的发布版本之间保持稳定。例如，在特定版本中在一个上下文中使用的函数可能在下一个版本中不在同一上下文中使用。覆盖函数的合约在更新它们所依赖的OpenZeppelin Contracts的版本时，应重新验证它们的假设。