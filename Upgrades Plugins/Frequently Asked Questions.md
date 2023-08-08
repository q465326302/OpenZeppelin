# Frequently Asked Questions

## 在升级时，我可以更改Solidity编译器版本吗？
是的。Solidity团队保证编译器会在版本之间保持存储布局不变。

## 为什么我会收到“无法从代理管理员调用回退函数”的错误？

这是由于*透明代理模式*造成的。如果您使用OpenZeppelin Upgrades插件，则不应该收到此错误，因为它使用ProxyAdmin合约来管理您的代理。

然而，如果您在编程方式使用OpenZeppelin Contracts代理，可能会遇到此类错误。解决方案是始终使用不是代理管理员的帐户与代理进行交互，除非您想特别调用代理本身的函数。

## 什么是合约的升级安全性？
在部署合约的代理时，合约代码有一些限制。特别是，合约不能有构造函数，并且不应使用selfdestruct或delegatecall操作，出于安全原因。

作为构造函数的替代，通常会设置一个初始化函数来处理合约的初始化。您可以使用*Initializable*基础合约来访问一个initializer修饰符，以确保该函数只被调用一次。
```
import "@openzeppelin/contracts/proxy/utils/Initializable.sol";
// Alternatively, if you are using @openzeppelin/contracts-upgradeable:
// import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract MyContract is Initializable {
  uint256 value;
  function initialize(uint256 initialValue) public initializer {
    value = initialValue;
  }
}
```

这两个插件都会验证您尝试部署的合约是否符合这些规则。您可以在*此处*阅读有关如何编写升级安全合约的更多信息。

## 如何禁用其中一些检查？
部署和升级相关的函数都带有一个可选的opts对象，其中包括一个unsafeAllow选项。可以将其设置为禁用插件执行的任何检查。可以单独禁用的检查列表如下：

* state-variable-assignment

* state-variable-immutable

* external-library-linking

* struct-definition

* enum-definition

* constructor

* delegatecall

* selfdestruct

* missing-public-upgradeto

此函数是原始的unsafeAllowCustomTypes和unsafeAllowLinkedLibraries的通用版本，允许手动禁用任何检查。

例如，为了升级到包含委托调用的实现，您可以调用：
```
await upgradeProxy(proxyAddress, implementationFactory, { unsafeAllow: ['delegatecall'] });
```

此外，还可以通过Solidity源代码中的NatSpec注释精确地禁用检查。这需要Solidity版本大于等于0.8.2。
```
contract SomeContract {
  function some_dangerous_function() public {
    ...
    /// @custom:oz-upgrades-unsafe-allow delegatecall
    (bool success, bytes memory returndata) = msg.sender.delegatecall("");
    ...
  }
}
```

这个语法可以与以下错误一起使用：

* /// @custom:oz-upgrades-unsafe-allow state-variable-immutable

* /// @custom:oz-upgrades-unsafe-allow state-variable-assignment

* /// @custom:oz-upgrades-unsafe-allow external-library-linking

* /// @custom:oz-upgrades-unsafe-allow constructor

* /// @custom:oz-upgrades-unsafe-allow delegatecall

* /// @custom:oz-upgrades-unsafe-allow selfdestruct

在某些情况下，您可能希望在一行中允许多个错误。
```
contract SomeOtherContract {
  /// @custom:oz-upgrades-unsafe-allow state-variable-immutable state-variable-assignment
  uint256 immutable x = 1;
}
```
您还可以使用以下内容来允许可达代码中特定的错误，其中包括任何引用的合约、函数和库：

* /// @custom:oz-upgrades-unsafe-allow-reachable delegatecall

* /// @custom:oz-upgrades-unsafe-allow-reachable selfdestruct

## 我可以安全地使用delegatecall和selfdestruct吗？
> CAUTION
这是一种高级技术，可能会使资金面临永久丢失的风险。

如果通过代理来保护它们，使它们只能通过代理触发而不能在实现合约本身上触发，那么使用delegatecall和selfdestruct可能是安全的。在Solidity中实现这一点的方法如下。
```
abstract contract OnlyDelegateCall {
    /// @custom:oz-upgrades-unsafe-allow state-variable-immutable state-variable-assignment
    address private immutable self = address(this);

    function checkDelegateCall() private view {
        require(address(this) != self);
    }

    modifier onlyDelegateCall() {
        checkDelegateCall();
        _;
    }
}
```
 
```
contract UsesUnsafeOperations is OnlyDelegateCall {
    /// @custom:oz-upgrades-unsafe-allow selfdestruct
    function destroyProxy() onlyDelegateCall {
        selfdestruct(msg.sender);
    }
}
```

## 实现是兼容的意味着什么?
当将一个代理从一种实现升级到另一种实现时，两种实现的存储布局必须兼容。这意味着，即使可以完全更改实现的代码，也不能修改现有的合约状态变量。唯一允许的操作是在已声明的变量之后追加新的状态变量。

两个插件将验证新的实现合约是否与先前的合约兼容。

您可以在*这里*阅读有关如何对实现合约进行存储兼容性更改的更多信息。

## 代理管理员是什么？
ProxyAdmin是一个充当所有代理的所有者的合约。每个网络只部署一个。当您启动项目时，ProxyAdmin由部署者地址拥有，但您可以通过调用*transferOwnership*来转移所有权。

## 什么是实现合约？
可升级的部署需要至少两个合约：代理和实现。代理合约是您和您的用户将交互的实例，而实现是保存代码的合约。如果您对同一个实现合约调用deployProxy多次，则将部署多个代理，但只使用一个实现合约。

当您将代理升级到新版本时，如果需要，将部署一个新的实现合约，并将代理设置为使用新的实现合约。您可以在*这里*阅读有关代理升级模式的更多信息。

## 什么是代理？
代理是一个将其所有调用委托给第二个合约（称为实现合约）的合约。所有状态和资金都保存在代理中，但实际执行的代码是实现的代码。代理管理员可以将代理升级为使用不同的实现合约。

您可以在*这里*阅读有关代理升级模式的更多信息。

## 为什么不能使用不可变变量？
Solidity 0.6.5[引入了不可变关键字](https://github.com/ethereum/solidity/releases/tag/v0.6.5)，用于声明只能在构造函数期间分配一次且在构造函数后只能读取的变量。它通过在合约创建期间计算其值并直接将其值存储到字节码中实现。

请注意，这种行为与可升级合约的工作方式不兼容，原因如下：

1. 可升级合约没有构造函数，而是初始化函数，因此无法处理不可变变量。
2. 由于不可变变量的值存储在字节码中，它的值将在指向给定合约的所有代理之间共享，而不是每个代理的存储中。

> NOTE
在某些情况下，不可变变量是可升级的。插件目前无法自动检测这些情况，因此它们将将其视为错误。您可以使用选项unsafeAllow: ['state-variable-immutable']手动禁用检查，或在Solidity >=0.8.2中，在变量声明之前放置注释/// @custom:oz-upgrades-unsafe-allow state-variable-immutable。

## 为什么我不能使用外部库？
目前，插件对链接到外部库的可升级合约只提供部分支持。这是因为在编译时无法确定要链接的实现，因此很难保证升级操作的安全性。

计划在不久的将来添加此功能，并添加一些限制条件，使问题更容易解决，例如假设外部库的源代码要么存在于代码库中，要么已部署和挖掘，因此可以从区块链中获取以进行分析。

在此期间，您可以通过在deployProxy或upgradeProxy调用中将unsafeAllowLinkedLibraries标志设置为true，或在unsafeAllow数组中包含'external-library-linking'，以部署链接到外部库的可升级合约。请记住，插件将不会验证链接的库是否具有升级安全性。目前必须手动完成此操作，直到完全支持外部库。

您可以在[GitHub上](https://github.com/OpenZeppelin/openzeppelin-upgrades/issues/52)关注或贡献此问题。

## 为什么需要一个公共的upgradeTo函数？
当使用UUPS代理（通过kind: 'uups'选项）时，实现合约必须包含公共函数upgradeTo(address newImplementation)。这是因为在UUPS模式中，代理本身不包含升级函数，整个升级功能机制存在于实现方。因此，在每次部署和升级时，我们必须确保包含该函数，否则可能会永久禁用合约的可升级性。

建议的包含此函数的方法是继承OpenZeppelin Contracts中提供的UUPSUpgradeable合约，如下所示。该合约添加了所需的upgradeTo函数，但还包含一个内置机制，将在升级时在链上检查提议的新实现是否也继承了UUPSUpgradeable或实现了相同的接口。通过这种方式，使用Upgrades插件时，有两层减轻意外禁用可升级性的措施：插件的链外检查和合约本身的链上回退。
```
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

contract MyContract is Initializable, ..., UUPSUpgradeable {
    ...
}
```

阅读有关*Transparent vs UUPS*中Transparent Proxy模式的差异的更多信息。

我可以使用自定义类型，如结构体和枚举吗？
过去版本的插件不支持在其代码或链接库中使用结构体或枚举的可升级合约。对于当前版本的插件，这不再是问题，当升级合约时，会自动检查结构体和枚举的兼容性。

一些已经部署了具有结构体和/或枚举的代理并且需要升级这些代理的用户可能需要在下一次升级时使用override标志unsafeAllowCustomTypes，之后将不再需要。如果项目包含当前代理使用的实现的源代码，插件将在升级之前尝试恢复所需的元数据，如果这不可能，则回退到使用override标志。

## 为什么我必须重新编译所有的Truffle合约？
Truffle的构建文件（build/contracts中的JSON文件）包含每个合约的AST（抽象语法树）。我们的插件使用这些信息来验证您的合约是否可以安全升级。

Truffle有时只会部分重新编译已更改的合约。当发生这种情况时，我们将要求您触发完整的重新编译，可以使用truffle compile --all或删除build/contracts目录。技术原因是，由于Solidity不会生成确定性的AST，如果它们不是来自同一次编译运行，插件将无法正确解析引用。

## 我如何重命名变量或更改其类型？
默认情况下，不允许重命名变量，因为重命名实际上可能是一个意外的重新排序。例如，如果变量uint a; uint b; 升级为uint b; uint a;，如果简单地允许重命名，这不会被视为错误，但这可能是一个意外，特别是当涉及多重继承时。

可以通过传递选项unsafeAllowRenames: true来禁用此检查。更细粒度的方法是在被重命名的变量上方使用文档字符串注释/// @custom:oz-renamed-from <previous name>，例如：
```
contract V1 {
    uint x;
}
contract V2 {
    /// @custom:oz-renamed-from x
    uint y;
}
```
即使类型具有相同的大小和对齐方式，也不允许更改变量的类型，原因与上述类似。只要我们能够保证布局的其余部分不受此类型更改的影响，也可以通过添加文档字符串注释/// @custom:oz-retyped-from <previous type>来重写此检查。
```
contract V1 {
    bool x;
}
contract V2 {
    /// @custom:oz-retyped-from bool
    uint8 x;
}
```
由于当前Solidity的限制，结构成员尚不支持docstring注释。