# Extending Contracts 
大多数OpenZeppelin合约都预计通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)来使用：在编写自己的合约时，您将继承它们。

这是常见的语法，例如合约MyToken是ERC20。

>NOTE
与合约不同，Solidity库不是继承自的，而是依赖于使用[for语法](https://solidity.readthedocs.io/en/latest/contracts.html#using-for)。
OpenZeppelin合约有一些库：大多数位于[Utils](https://docs.openzeppelin.com/contracts/4.x/api/utils)目录中。

## 覆盖
继承通常用于将父合约的功能添加到您自己的合约中，但这不是它的全部作用。您还可以使用覆盖来更改父级的某些部分行为。

例如，想象一下，您想更改*AccessControl*，以便无法调用*revokeRole*。这可以使用覆盖来实现：
```
// contracts/ModifiedAccessControl.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";

contract ModifiedAccessControl is AccessControl {
    // Override the revokeRole function
    function revokeRole(bytes32, address) public override {
        revert("ModifiedAccessControl: cannot revoke roles");
    }
}
```

旧的revokeRole被我们的override所取代，任何对它的调用都会立即回滚。我们无法从合约中删除该函数，但在所有调用上回滚是足够好的。

### 调用super
有时，您想要扩展父级的行为，而不是彻底更改它为其他内容。这就是super的作用。

super关键字将允许您调用在父合约中定义的函数，即使它们被覆盖。可以使用此机制添加附加检查到函数中，发出事件或以您认为合适的方式添加功能。

>TIP
有关如何使用重写的更多信息，请转到[官方Solidity文档](https://solidity.readthedocs.io/en/latest/contracts.html#index-17)。

这是*AccessControl*的修改版本，其中*revokeRole*无法用于撤销DEFAULT_ADMIN_ROLE：
```
// contracts/ModifiedAccessControl.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";

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
super.revokeRole语句在最后将调用AccessControl的原始版本revokeRole，即如果没有覆盖，则运行的相同代码。

>NOTE
*AccessControlDefaultAdminRules*中实现并扩展了相同的规则，该扩展还为DEFAULT_ADMIN_ROLE添加了强制安全措施。

## 使用钩子
有时，为了扩展父合约，您需要覆盖多个相关函数，这会导致代码重复和错误的可能性增加。

例如，考虑以*IERC721Receiver*的方式实现安全的*ERC20*转账。您可能认为覆盖*transfer*和*transferFrom*就足够了，但是*_transfer*和*_mint*呢？为了防止您处理这些细节，我们引入了**钩子**。

钩子只是在某些操作之前或之后调用的函数。它们提供了集中式的点来钩入和扩展原始行为。

以下是使用**_beforeTokenTransfer**钩子实现ERC20中的IERC721Receiver模式的方法：
```
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract ERC20WithSafeTransfer is ERC20 {
    function _beforeTokenTransfer(address from, address to, uint256 amount)
        internal virtual override
    {
        super._beforeTokenTransfer(from, to, amount);

        require(_validRecipient(to), "ERC20WithSafeTransfer: invalid recipient");
    }

    function _validRecipient(address to) private view returns (bool) {
        ...
    }

    ...
}
```

使用钩子的这种方式可以使代码更加清洁和安全，而无需依赖于父级内部的深入理解。

### 钩子的规则
在编写使用钩子的代码时，应遵循一些指导方针以防止出现问题。它们非常简单，但确保您遵循它们：

1. 每当覆盖父级的钩子时，请重新应用虚拟属性到钩子上。这将允许子合约向钩子添加更多功能。

2. **始终**在覆盖中使用super调用父级的钩子。这将确保调用继承树中的所有钩子：像*ERC20Pausable*这样的合约依赖于这种行为。

```
contract MyToken is ERC20 {
    function _beforeTokenTransfer(address from, address to, uint256 amount)
        internal virtual override // Add virtual here!
    {
        super._beforeTokenTransfer(from, to, amount); // Call parent hook
        ...
    }
}
```
就是这样！享受使用钩子更简单的代码吧！

## 安全性
OpenZeppelin Contracts 的维护者主要关注发布在库中的代码的正确性和安全性，以及基础合约与库中官方扩展的组合。

自定义覆盖和特别是钩子的覆盖可能会破坏一些重要的假设，并在原本安全的代码中引入漏洞。虽然我们尽力确保合约在面对各种潜在的自定义时仍然安全，但这只是尽力而为。虽然我们尝试记录所有重要的假设，但不应依赖此。自定义覆盖应仔细审查，并与它们定制的合约的源代码进行检查，以充分了解它们的影响并保证其安全性。

函数在内部交互的方式不应该被认为在库的不同版本中保持稳定。例如，在特定版本中使用的函数可能在下一个版本中不在相同的上下文中使用。覆盖函数的合约应该在更新构建在 OpenZeppelin Contracts 上的版本时重新验证它们的假设。