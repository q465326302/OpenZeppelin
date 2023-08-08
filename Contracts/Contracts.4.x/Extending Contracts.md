# Extending Contracts 
大多数OpenZeppelin合约都通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)来使用：你可以在编写自己的合约时从它们继承。

这是常见的语法，例如合约MyToken是继承ERC20。

> NOTE
与合约不同，Solidity库不是通过继承来使用的，而是依赖于使用[for语法](https://solidity.readthedocs.io/en/latest/contracts.html#using-for)。
OpenZeppelin合约有一些库：大多数位于[Utils](../Contracts.4.x/Utilities.md)目录中。

## 重写
继承通常用于将父级合约的功能添加到您自己的合约中，但这不是它的全部作用。您还可以使用重写来更改父级的某些部分行为。

例如，想象一下，您想改变[AccessControl](./API/Access.md)，使[revokeRole](./API/Access.md)不能再被调用。这可以使用重写来实现：
```
// contracts/ModifiedAccessControl.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/access/AccessControl.sol";

contract ModifiedAccessControl is AccessControl {
    // 重写revokeRole函数
    function revokeRole(bytes32, address) public override {
        revert("ModifiedAccessControl: cannot revoke roles");
    }
}
```

我们的重写替换了旧的revokeRole函数，任何对它的调用都会立即回滚。我们无法从合约中删除该函数，但在所有调用上回滚足够了。

### 调用super
有时，您想要扩展父级的行为，而不是彻底更改它的其他内容。这就是super的作用。

super关键字将允许您调用在父合约中定义的函数，即使它们被重写。这个机制可以用来在函数中添加额外的检查、触发事件或以其他你认为合适的方式添加功能。

> TIP
有关如何使用重写的更多信息，请转到[官方Solidity文档](https://solidity.readthedocs.io/en/latest/contracts.html#index-17)。

这是[AccessControl](./API/Access.md)的修改版本，其中[revokeRole](./API/Access.md)无法用于撤销DEFAULT_ADMIN_ROLE：
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

最后super.revokeRole语句将调用AccessControl的原始版本revokeRole，即如果没有重写，则运行的相同代码。

> NOTE
[AccessControlDefaultAdminRules](./API/Access.md)中实现并扩展了相同的规则，该扩展还为DEFAULT_ADMIN_ROLE添加了强制安全措施。

## 使用钩子
有时，为了扩展父合约，您需要重写多个相关函数，这会导致代码重复和错误的可能性增加。

例如，考虑以[IERC721Receiver](./Tokens/ERC721.md)的方式实现安全的[ERC20](./API/ERC%2020.md)转账。您可能认为重写[transfer和transferFrom](./API/ERC%2020.md)就足够了，但是[_transfer和_mint](./API/ERC%2020.md)呢？为了避免你处理这些细节，我们引入了**hooks(钩子)**。

钩子只是在某些操作发生之前或之后调用的函数。它们提供了一个集中的点来挂钩和扩展原始行为。

以下是使用**_beforeTokenTransfer**钩子实现ERC20中的IERC721Receiver模式的示例：
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

使用钩子的这种方式可以使代码更简洁、更安全，而无需依赖于父合约的内部实现。

### 钩子的规则
在编写使用钩子的代码时，应遵循一些指导方针以防止出现问题。它们非常简单，但确保您遵循它们：

1. 每当重写父级的钩子时，请重新应用virtual属性到钩子上。这将允许子合约向钩子添加更多功能。

2. 在你的重写中**始终**使用super调用父合约的hook。这将确保调用继承树中的所有钩子：像[ERC20Pausable](./API/ERC%2020.md)这样的合约依赖于这种行为。

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
就是这样！享受使用钩子的简化代码吧！

## 安全性
OpenZeppelin Contracts 的维护者主要关注库中发布的代码的正确性和安全性，以及与库中的官方扩展的基本合约组合。

自定义重写，特别是hooks的重写，可能会破坏一些重要的假设，并在原本安全的代码中引入漏洞。虽然我们尽力确保合约在面对各种潜在的自定义时仍然安全，但这只是尽力而为。虽然我们尝试记录所有重要的假设，但不应依赖于此。自定义重写应该经过仔细审查，并与它们定制的合约的源代码进行检查，以充分了解它们的影响并保证其安全性。

不应该假设函数在不同版本的库中的内部交互方式保持稳定。例如，在特定版本中使用的函数可能在下一个版本中不再用于相同的上下文。重写函数的合约在更新所构建的OpenZeppelin Contracts版本时应重新验证其假设。