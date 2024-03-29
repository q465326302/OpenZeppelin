# Extending Contracts
大多数OpenZeppelin合约都是通过[继承](https://solidity.readthedocs.io/en/latest/contracts.html#inheritance)来使用的：在编写自己的合约时，你将从它们继承。

这是常见的语法，例如 contract MyToken is ERC20。

> NOTE
与合约不同，Solidity库不会继承，而是依赖于[using for](https://solidity.readthedocs.io/en/latest/contracts.html#using-for)语法。
OpenZeppelin合约中有一些库：大部分位于*Utils*目录中。

## 重写
继承经常用于将父合约的功能添加到你自己的合约中，但这并不是它的全部作用。你还可以使用重写来更改父合约的某些部分的行为。

例如，假设你想要更改[AccessControl](./API/Access.md)，使[revokeRole](./API/Access.md#revokerolebytes32-role-address-account)无法被调用。可以使用重写来实现这一点：
```
// contracts/ModifiedAccessControl.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts/access/AccessControl.sol";

contract ModifiedAccessControl is AccessControl {
    // Override the revokeRole function
    function revokeRole(bytes32, address) public override {
        revert("ModifiedAccessControl: cannot revoke roles");
    }
}
```

然后，旧的revokeRole被我们的override替换了，对它的任何调用都会立即回滚。我们无法从合约中删除该函数，但在所有调用上回滚足够了。

### 调用super
有时候你想要扩展父级的行为，而不是完全改变它为其他东西。这就是super的作用所在。

super关键字允许你调用在父合约中定义的函数，即使它们被重写。这个机制可以用来给函数添加额外的检查、触发事件或以你认为合适的方式添加功能。

> TIP
有关override的更多信息，请查阅官方[Solidity文档](https://solidity.readthedocs.io/en/latest/contracts.html#index-17)。

这是*[AccessControl](./API/Access.md)的修改版本，其中[revokeRole](./API/Access.md#revokerolebytes32-role-address-account)不能用于撤销DEFAULT_ADMIN_ROLE的权限：
```
// contracts/ModifiedAccessControl.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

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

最后的super.revokeRole语句将调用AccessControl的原始版本的revokeRole，即如果没有设置重写的情况下将运行的相同代码。

> NOTE
自v3.0.0起，OpenZeppelin中的视图函数不是虚拟的，因此无法被重写。我们正在考虑在即将发布的版本中[解除这个限制](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2154)。如果你对此有兴趣，请告诉我们！

## 使用Hooks
有时，为了扩展父合约，你需要重写多个相关函数，这会导致代码重复和错误发生的可能性增加。

例如，考虑以[IERC721Receiver](./Tokens/ERC721.md#预设erc721合约)的方式实现安全的[ERC20](./API/ERC20.md#erc20)转账。你可能认为重写[transfer](./API/ERC20.md#transferaddress-to-uint256-amount-→-bool)和[transferFrom](./API/ERC20.md#_transferaddress-from-address-to-uint256-amount)就足够了，但是[_transfer](./API/ERC20.md#_transferaddress-from-address-to-uint256-amount)和[_mint](./API/ERC20.md#_mintaddress-account-uint256-amount)呢？为了防止你处理这些细节，我们引入了**Hooks**。

Hooks只是在某个操作之前或之后调用的函数。它们提供了一个集中的点来连接和扩展原始行为。

以下是使用[_beforeTokenTransfer](./API/ERC20.md#_burnaddress-account-uint256-amount) hook在ERC20中实现IERC721Receiver模式的示例：
```
pragma solidity ^0.6.0;

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

以这种方式使用 hooks 可以使代码更清晰、更安全，而不需要依赖于父级的内部细节的深入理解。

> NOTE
 hooks 是OpenZeppelin Contracts v3.0.0的一个新功能，我们渴望了解你打算如何使用它们！
到目前为止，唯一可用的 hooks 是_beforeTransferHook，在[ERC20](./API/ERC20.md#_beforetokentransferaddress-from-address-to-uint256-amount)、[ERC721](./API/ERC721.md#_beforetokentransferaddress-from-address-to-uint256-tokenid)、[ERC777](./API/ERC777.md#_beforetokentransferaddress-operator-address-from-address-to-uint256-amount)和[ERC1155](./API/ERC1155.md#_beforetokentransferaddress-operator-address-from-address-to-uint256-ids-uint256-amounts-bytes-data)中都有。如果你对新的 hooks 有想法，请告诉我们！

## Hooks的规则
在编写使用hooks的代码时，有一些准则可以防止出现问题。它们非常简单，但请确保遵循以下规则：

1. 每当重写父级的 hooks 时，请重新应用virtual属性到 hooks 上。这将允许子合约向 hooks 添加更多功能。

2. 在你的重写中**始终**使用super调用父合约的hook。这将确保调用继承树中的所有 hooks ：像[ERC20Pausable](./API/ERC20.md#erc20pausable)这样的合约依赖于这种行为。
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

就是这样！享受使用hooks的更简单的代码吧！

