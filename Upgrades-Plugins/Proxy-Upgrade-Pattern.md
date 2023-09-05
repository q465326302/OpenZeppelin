# Proxy Upgrade Pattern

本文介绍了“非结构化存储”代理模式，这是OpenZeppelin升级的基本构建模块。

> TIP
如需更详细的阅读，请参阅[我们的代理模式博文](https://blog.openzeppelin.com/proxy-patterns/)，该博文讨论了代理的需求，更详细地介绍了该主题，详述了OpenZeppelin升级考虑的其他可能的代理模式等等。

## 为什么要升级合约？

根据设计，智能合约是不可变的。另一方面，软件的质量严重依赖于能够升级和修补源代码以产生迭代发布。尽管基于区块链的软件从技术的不可变性中获益很多，但仍然需要一定程度的可变性来进行错误修复和潜在的产品改进。OpenZeppelin升级通过为智能合约提供一个易于使用、简单、强大且可选择的升级机制来解决这种明显的矛盾，该机制可以由任何类型的治理控制，无论是多签名钱包、简单地址还是复杂的DAO。

## 通过代理模式进行升级

基本思路是使用代理进行升级。第一个合约是一个简单的包装器或“代理”，用户直接与之交互，并负责将事务转发到第二个包含逻辑的合约。要理解的关键概念是，逻辑合约可以被替换，而代理或访问点永远不会改变。两个合约在代码不能被更改的意义上仍然是不可变的，但是逻辑合约可以简单地被另一个合约替换。因此，包装器可以指向不同的逻辑实现，在这样做的过程中，软件被“升级”。

用户 ---- 交易 ---> 代理 ----------> 实现_v0
                     |
                      ------------> 实现_v1
                     |
                      ------------> 实现_v2


## 代理转发

代理需要解决的最直接的问题是如何在不要求整个逻辑合约接口一对一映射的情况下公开逻辑合约的整个接口。这将很难维护，容易出错，并且会使接口本身无法升级。因此，需要一种动态转发机制。下面的代码介绍了这种机制的基础部分：
```
// This code is for "illustration" purposes. To implement this functionality in production it
// is recommended to use the `Proxy` contract from the `@openzeppelin/contracts` library.
// https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.8.2/contracts/proxy/Proxy.sol

assembly {
  // (1) copy incoming call data
  calldatacopy(0, 0, calldatasize())

  // (2) forward call to logic contract
  let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)

  // (3) retrieve return data
  returndatacopy(0, 0, returndatasize())

  // (4) forward return data back to caller
  switch result
  case 0 {
      revert(0, returndatasize())
  }
  default {
      return(0, returndatasize())
  }
}
```

这段代码可以放在代理的[fallback函数](https://docs.soliditylang.org/en/latest/contracts.html#fallback-function)中，它会将任何调用转发给逻辑合约的任何函数，而不需要逻辑合约对接口的任何特定了解。简而言之，(1) calldata被复制到内存中，(2) 调用被转发到逻辑合约，(3) 获取从逻辑合约调用返回的数据，(4) 将返回的数据转发回调用者。

一个非常重要的事情需要注意的是，该代码使用了EVM的delegatecall操作码，该操作码在调用者的状态上下文中执行被调用者的代码。也就是说，逻辑合约控制着代理的状态，逻辑合约的状态是无意义的。因此，代理不仅将事务转发给逻辑合约，还代表了这对的状态。状态在代理中，逻辑在代理指向的特定实现中。

## 非结构化存储代理
在使用代理时，一个常见的问题与变量在代理合约中的存储方式有关。假设代理在其唯一的变量address public _implementation中存储了逻辑合约的地址。现在，假设逻辑合约是一个基本代币，其第一个变量是address public _owner。这两个变量的大小都是32字节，就EVM而言，它们占据了代理调用执行流的第一个槽位。当逻辑合约写入_owner时，它实际上在代理状态的范围内写入_implementation。这个问题可以称为"存储冲突"。
|Proxy                     |Implementation           | |
|---|---|---|
|--------------------------|-------------------------|
|address _implementation   |address _owner           | <=== Storage collision! |
|...                       |mapping _balances        | |
|                          |uint256 _supply          | |
|                          |...                      | |

有很多方法可以克服这个问题，OpenZeppelin Upgrades 实现的 "非结构化存储" 方法的工作原理如下。它不是将 _implementation 地址存储在代理的第一个存储槽中，而是选择一个伪随机槽位。这个槽位是足够随机的，以至于逻辑合约在相同槽位声明变量的概率可以忽略不计。在代理的存储中，如果有其他变量，比如一个管理员地址（允许更新 _implementation 的值），也会使用相同的随机化槽位位置的原则。
|Proxy                     |Implementation           | |
|---|---|---|
|--------------------------|-------------------------| |
|...                       |address _owner           | |
|...                       |mapping _balances        | |
|...                       |uint256 _supply          | |
|...                       |...                      | |
|...                       |                         | |
|...                       |                         | |
|...                       |                         | |
|...                       |                         | |
|address _implementation   |                         | <=== Randomized slot. |
|...                       |                         | |
|...                       |                         | |

在[EIP 1967](http://eips.ethereum.org/EIPS/eip-1967)之后，如何实现随机存储的示例
```
bytes32 private constant implementationPosition = bytes32(uint256(
  keccak256('eip1967.proxy.implementation')) - 1
));
```

因此，逻辑合约不需要关心重写代理的任何变量。其他遇到这个问题的代理实现通常需要让代理了解逻辑合约的存储结构并进行适应，或者让逻辑合约了解代理的存储结构并进行适应。这就是为什么这种方法被称为"非结构化存储"；两个合约都不需要关心对方的结构。

## 实现版本之间的存储冲突
正如前面所讨论的，非结构化方法避免了逻辑合约和代理之间的存储冲突。但是，不同版本的逻辑合约之间可能会发生存储冲突。在这种情况下，假设逻辑合约的第一个实现在第一个存储位置存储了地址 public _owner，而升级后的逻辑合约在同一个第一个存储位置存储了地址 public _lastContributor。当更新后的逻辑合约尝试写入 _lastContributor 变量时，它将使用与之前 _owner 变量存储的相同存储位置，并重写它！

错误的存储保留：
|Implementation_v0   |Implementation_v1        | | 
|---|---|---|
|--------------------|-------------------------| |
|address _owner      |address _lastContributor | <=== Storage collision! |
|mapping _balances   |address _owner           | |
|uint256 _supply     |mapping _balances        | |
|...                 |uint256 _supply          | |
|                    |...                      | |

正确的储存保存方法:
|Implementation_v0   |Implementation_v1        | |
|---|---|---|
|--------------------|-------------------------| |
|address _owner      |address _owner           | |
|mapping _balances   |mapping _balances        | |
|uint256 _supply     |uint256 _supply          | |
|...                 |address _lastContributor | <=== Storage extension. |
|                    |...                      | |

非结构化存储代理机制无法防止这种情况发生。用户需要确保新版本的逻辑合约扩展了先前的版本，或者以其他方式保证存储层次结构只能追加而不能修改。然而，OpenZeppelin Upgrades会检测到这种冲突并向开发者发出警告。

## 构造函数注意事项
在Solidity中，位于构造函数或全局变量声明中的代码不属于已部署合约的运行时字节码的一部分。这段代码仅在合约实例部署时执行一次。因此，逻辑合约的构造函数中的代码在代理状态下永远不会被执行。换句话说，代理完全不知道构造函数的存在。对于代理来说，就好像构造函数不存在一样。

不过，这个问题很容易解决。逻辑合约应该将构造函数中的代码移动到一个常规的“初始化器”函数中，并在代理链接到该逻辑合约时调用该函数。对于这个初始化器函数，需要特别小心，以确保它只能被调用一次，这是一般编程中构造函数的一个特性。

这就是为什么使用OpenZeppelin Upgrades创建代理时，可以提供初始化器函数的名称并传递参数的原因。

为了确保初始化函数只能被调用一次，使用了一个简单的修饰符。OpenZeppelin Upgrades通过一个可以被扩展的合约提供了这个功能。
```
// contracts/MyContract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract MyContract is Initializable {
    function initialize(
        address arg1,
        uint256 arg2,
        bytes memory arg3
    ) public payable initializer {
        // "constructor" code...
    }
}
```

请注意，该合约扩展了Initializable并实现了由该合约提供的初始化器。

## 透明代理和函数冲突

如前文所述，升级合约实例（或代理）通过将所有调用委托给逻辑合约来工作。然而，代理需要一些自己的函数，比如upgradeTo(address)用于升级到新的实现。这就引出了一个问题：如果逻辑合约也有一个名为upgradeTo(address)的函数，那么当调用该函数时，调用者是要调用代理还是逻辑合约呢？

> CAUTION
函数冲突也可能发生在具有不同名称的函数之间。每个作为合约公共ABI的一部分的函数在字节码级别上都有一个4字节的标识符。该标识符取决于函数的名称和参数个数，但由于只有4个字节，可能会出现两个具有不同名称的不同函数具有相同标识符的情况。Solidity编译器会跟踪此类冲突是否发生在同一个合约中，但不会跟踪发生在不同合约之间（例如代理和其逻辑合约之间）的冲突。请阅读[本文](https://medium.com/nomic-labs-blog/malicious-backdoors-in-ethereum-proxies-62629adf3357)了解更多信息。

OpenZeppelin Upgrades解决这个问题的方法是通过透明代理模式。透明代理将根据调用者地址（即msg.sender）决定将哪些调用委托给底层逻辑合约：

* 如果调用者是代理的管理员（具有升级代理权限的地址），则代理不会委托任何调用，只会回答它所理解的任何消息。
* 如果调用者是任何其他地址，无论是否匹配代理的任何函数，代理都将始终委托调用。

假设一个代理具有owner()和upgradeTo()函数，并将调用委托给具有owner()和transfer()函数的ERC20合约，以下表格涵盖了所有情况：

| msg.sender|	owner()|	upgradeto()|	transfer()|
|---|---|---|---|
|Owner|returns proxy.owner()|returns proxy.upgradeTo()|fails|
|Other|returns erc20.owner()|fails|returns erc20.transfer()|

幸运的是，OpenZeppelin Upgrades针对这种情况进行了处理，并创建了一个中间的ProxyAdmin合约，负责通过Upgrades插件创建的所有代理。即使您从节点的默认账户调用deploy命令，ProxyAdmin合约仍将是所有代理的实际管理员。这意味着您可以从节点的任何账户与代理进行交互，而不必担心透明代理模式的细微差别。只有通过Solidity创建代理的高级用户需要了解透明代理模式。

## 总结
任何使用可升级合约的开发人员都应该熟悉本文所描述的代理方式。最后，这个概念非常简单，OpenZeppelin Upgrades旨在以一种方式封装所有代理机制，以使开发项目时需要记住的事项最少。以下是要记住的几点：

* 基本了解代理的概念
* 总是扩展存储而不是修改它
* 确保您的合约使用初始化函数而不是构造函数

此外，OpenZeppelin Upgrades将在列表中的某一项出现问题时通知您。