# Writing Upgradeable Contracts
在使用OpenZeppelin Upgrades处理升级合约时，编写Solidity代码时需要注意一些小细节。

值得一提的是，这些限制是基于以太坊虚拟机的工作原理，并适用于所有使用升级合约的项目，而不仅仅是OpenZeppelin Upgrades。

## 初始化函数
你可以在不做任何修改的情况下使用Solidity合约与OpenZeppelin Upgrades，除了它们的构造函数。由于基于代理的升级系统的要求，升级合约中不能使用构造函数。要了解这一限制背后的原因，请参阅*代理*。

这意味着，在使用OpenZeppelin Upgrades处理合约时，你需要将其构造函数更改为一个常规函数，通常命名为initialize，在其中运行所有的设置逻辑：

```
// NOTE: Do not use this code snippet, it's incomplete and has a critical vulnerability!

pragma solidity ^0.6.0;

contract MyContract {
    uint256 public x;

    function initialize(uint256 _x) public {
        x = _x;
    }
}
```
然而，尽管Solidity确保构造函数在合约的生命周期内仅被调用一次，但常规函数可以被多次调用。为了防止合约被多次初始化，您需要添加一个检查，以确保initialize函数仅被调用一次。

```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract MyContract {
    uint256 public x;
    bool private initialized;

    function initialize(uint256 _x) public {
        require(!initialized, "Contract instance has already been initialized");
        initialized = true;
        x = _x;
    }
}
```
由于在编写可升级合约时，这种模式非常常见，OpenZeppelin Contracts提供了一个Initializable基础合约，它具有一个初始化器修饰符，可以处理这个问题。
```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract MyContract is Initializable {
    uint256 public x;

    function initialize(uint256 _x) public initializer {
        x = _x;
    }
}
```

构造函数和普通函数之间的另一个区别是Solidity会自动调用所有合约祖先的构造函数。当编写一个初始化函数时，您需要特别注意手动调用所有父合约的初始化函数。请注意，即使使用继承，初始化函数修饰符也只能调用一次，因此父合约应该使用onlyInitializing修饰符。
```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract BaseContract is Initializable {
    uint256 public y;

    function initialize() public onlyInitializing {
        y = 42;
    }
}

contract MyContract is BaseContract {
    uint256 public x;

    function initialize(uint256 _x) public initializer {
        BaseContract.initialize(); // Do not forget this call!
        x = _x;
    }
}
```

### 使用可升级的智能合约库
请记住，这个限制不仅影响您的合约，还影响您从库中导入的合约。例如，考虑OpenZeppelin Contracts中的[ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.7.3/contracts/token/ERC20/ERC20.sol)合约：该合约在其构造函数中初始化代币的名称和符号。
```
// @openzeppelin/contracts/token/ERC20/ERC20.sol
pragma solidity ^0.8.0;

...

contract ERC20 is Context, IERC20 {

    ...

    string private _name;
    string private _symbol;

    constructor(string memory name_, string memory symbol_) {
        _name = name_;
        _symbol = symbol_;
    }

    ...
}
```

这意味着你不应该在你的OpenZeppelin Upgrades项目中使用这些合约。相反，请确保使用@openzeppelin/contracts-upgradeable，这是OpenZeppelin Contracts的官方分支，已经修改为使用initializers而不是constructors。看一下在@openzeppelin/contracts-upgradeable中[ERC20Upgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/v4.7.3/contracts/token/ERC20/ERC20Upgradeable.sol)是什么样子的：
```
// @openzeppelin/contracts-upgradeable/contracts/token/ERC20/ERC20Upgradeable.sol
pragma solidity ^0.8.0;

...

contract ERC20Upgradeable is Initializable, ContextUpgradeable, IERC20Upgradeable {
    ...

    string private _name;
    string private _symbol;

    function __ERC20_init(string memory name_, string memory symbol_) internal onlyInitializing {
        __ERC20_init_unchained(name_, symbol_);
    }

    function __ERC20_init_unchained(string memory name_, string memory symbol_) internal onlyInitializing {
        _name = name_;
        _symbol = symbol_;
    }

    ...
}
```

无论是使用OpenZeppelin Contracts还是其他智能合约库，都要确保该库已设置为处理可升级合约。

请在Contracts: Using with Upgrades中了解更多关于*OpenZeppelin Contracts Upgradeable*的信息。

### 在字段声明中避免初始值
Solidity允许在合约中声明字段时定义初始值。
```
contract MyContract {
    uint256 public hasInitialValue = 42; // equivalent to setting in the constructor
}
```

这等同于在构造函数中设置这些值，因此对于可升级的合约是行不通的。请确保所有初始值都在下面所示的初始化函数中设置；否则，任何可升级的实例将没有这些字段设置。

```
contract MyContract is Initializable {
    uint256 public hasInitialValue;

    function initialize() public initializer {
        hasInitialValue = 42; // set initial value in initializer
    }
}
```

> NOTE
仍然可以定义常量状态变量，因为编译器[不会为这些变量保留存储槽](https://solidity.readthedocs.io/en/latest/contracts.html#constant-state-variables)，而是将每个出现的地方替换为相应的常量表达式。因此，在OpenZeppelin Upgrades中仍然可以使用以下代码：
```
contract MyContract {
    uint256 public constant hasInitialValue = 42; // define as constant
}
```

### 初始化实现合约

不要将实现合约保持为未初始化状态。未初始化的实现合约可能会被攻击者接管，从而可能影响代理。为了防止实现合约被使用，您应该在构造函数中调用_disableInitializers函数来在部署时自动锁定它：
```
/// @custom:oz-upgrades-unsafe-allow constructor
constructor() {
    _disableInitializers();
}
```

## 从合约代码中创建新实例
当从合约代码中创建合约的新实例时，这些创建是由Solidity直接处理的，而不是由OpenZeppelin Upgrades处理，**这意味着这些合约将无法升级**。

例如，在下面的示例中，即使MyContract被部署为可升级的，创建的token合约却不是可升级的：
```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyContract is Initializable {
    ERC20 public token;

    function initialize() public initializer {
        token = new ERC20("Test", "TST"); // This contract will not be upgradeable
    }
}
```

如果您希望ERC20实例可以升级，最简单的方法是接受该合约的一个实例作为参数，并在创建后注入它：

```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";

contract MyContract is Initializable {
    IERC20Upgradeable public token;

    function initialize(IERC20Upgradeable _token) public initializer {
        token = _token;
    }
}
```

## 潜在的不安全操作
在使用可升级智能合约时，您将始终与合约实例进行交互，而不是与底层逻辑合约进行交互。然而，没有任何防止恶意参与者直接向逻辑合约发送交易的措施。这并不构成威胁，因为逻辑合约的状态更改不会影响您的合约实例，因为您的项目中从未使用过逻辑合约的存储。

然而，有一个例外情况。如果对逻辑合约的直接调用触发了自毁操作，则逻辑合约将被销毁，并且您的所有合约实例将最终将所有调用委托给一个没有任何代码的地址。这将有效地破坏您项目中的所有合约实例。

如果逻辑合约包含委托调用操作，也可以实现类似的效果。如果可以使合约委托调用到包含自毁操作的恶意合约中，那么调用合约将被销毁。

因此，不允许在您的合约中使用selfdestruct或delegatecall操作。

## 修改合约
当编写合约的新版本时，无论是因为新增功能还是修复错误，还有一个额外的限制需要遵守：您不能更改合约状态变量的声明顺序或类型。您可以通过了解我们的代理来了解有关此限制背后原因的更多信息。

违反这些存储布局限制将导致升级后的合约的存储值混乱，并可能导致应用程序中的严重错误。
这意味着，如果您有一个初始合约，看起来像这样：

## 修改合约
当编写合约的新版本时，无论是因为新增功能还是修复错误，还有一个额外的限制需要遵守：您不能更改合约状态变量的声明顺序或类型。您可以通过了解我们的*代理*来了解有关此限制背后原因的更多信息。

> WARNING
违反这些存储布局限制将导致升级后的合约的存储值混乱，并可能导致应用程序中的严重错误。

这意味着，如果您有一个初始合约，看起来像这样：
```
contract MyContract {
    uint256 private x;
    string private y;
}
```
那么你就不能改变变量的类型。
```
contract MyContract {
    string private x;
    string private y;
}
```
或者改变它们被声明的顺序。

```
contract MyContract {
    string private y;
    uint256 private x;
}
```

在现有变量之前引入一个新的变量。
```
contract MyContract {
    bytes private a;
    uint256 private x;
    string private y;
}
```

或者删除一个已存在的变量。
```
contract MyContract {
    string private y;
}
```

如果你需要引入一个新变量，确保你总是在最后引入。
```
contract MyContract {
    uint256 private x;
    string private y;
    bytes private z;
}
```

请记住，如果你重命名一个变量，那么在升级后它将保持与之前相同的值。如果新变量在语义上与旧变量相同，那么这可能是期望的行为。
```
contract MyContract {
    uint256 private x;
    string private z; // starts with the value from `y`
}
```

如果从合约末尾删除一个变量，请注意存储将不会被清除。随后的更新添加一个新变量时，该变量将读取被删除变量的残留值。
```
contract MyContract {
    uint256 private x;
}
```

然后升级到：
```
contract MyContract {
    uint256 private x;
    string private z; // starts with the value from `y`
}
```

请注意，通过更改合约的父合约，您可能会无意中更改合约的存储变量。例如，如果您有以下合约：
```
contract A {
    uint256 a;
}


contract B {
    uint256 b;
}


contract MyContract is A, B {}
```

然后通过交换声明基础合约的顺序，或引入新的基础合约，修改MyContract将改变变量的实际存储方式。
```
contract MyContract is B, A {}
```

在以下情况下，如果子合约有任何自己的变量，您也不能向基本合约添加新变量。
```
contract Base {
    uint256 base1;
}


contract Child is Base {
    uint256 child;
}
```

如果Base被修改以添加额外的变量：
```
contract Base {
    uint256 base1;
    uint256 base2;
}
```

然后变量base2将被分配为前一个版本中child的槽位。解决这个问题的一种方法是在未来可能需要扩展的基础合约中声明未使用的变量或存储空间，作为“保留”这些槽位的手段。请注意，这个技巧不会增加燃气使用量。

### 存储间隙
存储间隙是一种保留基础合约中存储槽的约定，允许该合约的未来版本使用这些槽，而不影响子合约的存储布局。

要创建一个存储间隙，在基础合约中声明一个固定大小的数组，并设置初始槽数量。这可以是一个uint256数组，以便每个元素保留一个32字节的槽。使用名称__gap或以__gap_开头的名称命名数组，以便OpenZeppelin Upgrades可以识别到该间隙。
```
contract Base {
    uint256 base1;
    uint256[49] __gap;
}

contract Child is Base {
    uint256 child;
}
```

如果Base稍后被修改以添加额外的变量，根据[Solidity对连续项目的打包规则](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html#layout-of-state-variables-in-storage)，从存储间隙中减少合适的槽位。例如：
```
contract Base {
    uint256 base1;
    uint256 base2; // 32 bytes
    uint256[48] __gap;
}
```

或者:
```
contract Base {
    uint256 base1;
    address base2; // 20 bytes
    uint256[48] __gap; // array always starts at a new slot
}
```

或者:
```
contract Base {
    uint256 base1;
    address base2; // 20 bytes
    uint256[48] __gap; // array always starts at a new slot
}
```

或者:
```
contract Base {
    uint256 base1;
    uint128 base2a; // 16 bytes
    uint128 base2b; // 16 bytes - continues from the same slot as above
    uint256[48] __gap;
}
```

为了确定新合约版本中的适当存储间隙大小，您可以尝试使用upgradeProxy进行升级，或者只需使用validateUpgrade运行验证（请参阅*Hardhat*或*Truffle*的文档）。如果存储间隙没有被正确地减少，您将看到一个错误信息，指示存储间隙的预期大小。