# Adding cross-chain support to contracts
如果你的合约用于多链操作使用，你需要特定的工具来识别和处理这些跨链操作。

OpenZeppelin提供了[CrossChainEnabled](./API/Crosschain.md#crosschainenabledamb)抽象合约，其中包括专用的内部函数。

在本指南中，我们将通过一个示例用例来介绍如何构建一个可升级和可铸造的ERC20代币，由外部链上的治理合约控制。

## Starting point, our ERC20 contract
让我们从一个小的ERC20合约开始，我们使用[OpenZeppelin Contracts Wizard](https://wizard.openzeppelin.com/)引导并通过具有铸造能力的管理者来扩展。请注意，出于演示目的，我们没有使用内置的Ownable合约。

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

contract MyToken is Initializable, ERC20Upgradeable, UUPSUpgradeable {
    address public owner;

    modifier onlyOwner() {
        require(owner == _msgSender(), "Not authorized");
        _;
    }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}

    function initialize(address initialOwner) initializer public {
        __ERC20_init("MyToken", "MTK");
        __UUPSUpgradeable_init();

        owner = initialOwner;
    }

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }

    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {
    }
}
```

这个代币可以由合约的所有者进行铸造和升级。

## Preparing our contract for cross-chain operations.
现在，让我们想象一下，这个合约将在一个链上运行，但我们希望铸造和升级是由另一个链上的[治理合约](./API/Governance.md)执行的。

例如，我们可以在xDai上拥有我们的代币，而我们的治理合约在主网上，或者我们可以在主网上拥有我们的代币，而我们的治理合约在Optimism上。

为了实现这一点，我们将首先向我们的合约添加[CrossChainEnabled](./API/Crosschain.md#cross-chain-awareness)。你会注意到，这合约现在是抽象的。这是因为CrossChainEnabled是一个抽象合约：它不与任何特定的链相绑定，并且以抽象的方式处理跨链交互。这就是使我们能够轻松地将代码重用于不同链的原因。我们以后可以通过继承特定链的实现来进行专门化。
```
 import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
+import "@openzeppelin/contracts-upgradeable/crosschain/CrossChainEnabled.sol";

-contract MyToken is Initializable, ERC20Upgradeable, UUPSUpgradeable {
+abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, UUPSUpgradeable, CrossChainEnabled {
```

完成这一步后，可以使用CrossChainEnabled提供的onlyCrossChainSender修饰符来保护我们的铸造和升级操作。
```
-    function mint(address to, uint256 amount) public onlyOwner {
+    function mint(address to, uint256 amount) public onlyCrossChainSender(owner) {

-    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {
+    function _authorizeUpgrade(address newImplementation) internal override onlyCrossChainSender(owner) {
```

这个改变将有效地限制铸造和升级操作只能由远程链上的所有者执行。

## Specializing for a specific chain
一旦我们的抽象跨链版本的代币准备就绪，我们就可以轻松地为我们想要的链或更准确地说是为我们想要依赖的桥接系统专门化它。

这是使用CrossChainEnabled实现之一来完成的。

例如，如果我们的代币在xDai上，我们的治理合约在主网上，我们可以在xDai上使用可用的[AMB](https://docs.tokenbridge.net/amb-bridge/about-amb-bridge)桥接，地址为[0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59](https://blockscout.com/xdai/mainnet/address/0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59)。

```
[...]

import "@openzeppelin/contracts-upgradeable/crosschain/amb/CrossChainEnabledAMB.sol";

contract MyTokenXDAI is
    MyTokenCrossChain,
    CrossChainEnabledAMB(0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59)
{}
```

如果代币在以太坊主网上，而我们的治理合约在Optimism上，我们将在主网上使用可用的[Optimism CrossDomainMessenger](https://community.optimism.io/docs/protocol/protocol-2.0/#l1crossdomainmessenger)，地址为[0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1](https://etherscan.io/address/0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1)。

```
[...]

import "@openzeppelin/contracts-upgradeable/crosschain/optimismCrossChainEnabledOptimism.sol";

contract MyTokenOptimism is
    MyTokenCrossChain,
    CrossChainEnabledOptimism(0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1)
{}
```

## Mixing cross domain addresses is dangerous

在设计具有跨链支持的合约时，了解可能的回退和正在进行的安全假设非常重要。

在本指南中，我们特别关注限制对特定调用者的访问。通常使用msg.sender或_msgSender()来实现（如上所示）。然而，在进行跨链操作时，情况并不仅仅是这样。即使不考虑可能存在的桥接问题，重要的是要记住，在考虑多链空间时，同一地址能对应于完全不同的实体。EOA钱包只能在钱包的私钥签署交易时执行操作。据我们所知，这在所有EVM链中都是这样的，因此来自此类钱包的跨链消息和非跨链消息等效。然而，对于智能合约来说，情况完全不同。

由于智能合约地址的计算方式和不同链上的智能合约独立运行，你可以在不同链上的相同地址上拥有两个完全不同的合约。你可以两个使用不同签署者的多重签名钱包在不同链上使用相同的地址。也可以在一个链上与治理者相同的地址上使用一个非常基本的智能钱包。因此，你应该小心，每当你授予特定地址许可权时，你要控制该特定地址可以在哪个链执行操作。

## Going further with access control

在前面的例子中，我们既有onlyOwner()修饰符，也有onlyCrossChainSender(owner)机制。我们没有使用[Ownable](./Access-Control.md#ownership-and-ownable)模式，因为其中包含的所有权转移机制不适用于所有者是跨链实体的情况。与[Ownable](./Access-Control.md#ownership-and-ownable)不同，[AccessControl](./Access-Control.md#role-based-access-control)在捕捉细微差别方面更有效，并且可以很好的用于构建跨链感知的合约。

使用[AccessControlCrossChain](./API/Access.md#accesscontrolcrosschain)包括[AccessControl](./API/Access.md#accesscontrol)核心和[CrossChainEnabled](./API/Crosschain.md#crosschainenabled的特殊化)抽象。它还包括一些绑定，使角色管理和跨链操作兼容。

在铸造函数的情况下，当调用源来自同一链时，调用者必须具有MINTER_ROLE。如果调用者不在同一链上，则调用者不应具有MINTER_ROLE，而应该具有“别名”版本（MINTER_ROLE ^ CROSSCHAIN_ALIAS）。通过严格区分来自不同链的本地账户和远程账户来减轻了之前描述的危险。有关更多详细信息，请参见[AccessControlCrossChain](./API/Access.md#accesscontrolcrosschain)文档。
```
 import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
 import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
+import "@openzeppelin/contracts-upgradeable/access/AccessControlCrossChainUpgradeable.sol";

-abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, UUPSUpgradeable, CrossChainEnabled {
+abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, UUPSUpgradeable, AccessControlCrossChainUpgradeable {

-    address public owner;
-    modifier onlyOwner() {
-        require(owner == _msgSender(), "Not authorized");
-        _;
-    }

+    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
+    bytes32 public constant UPGRADER_ROLE = keccak256("UPGRADER_ROLE");

     function initialize(address initialOwner) initializer public {
         __ERC20_init("MyToken", "MTK");
         __UUPSUpgradeable_init();
+        __AccessControl_init();
+        _grantRole(_crossChainRoleAlias(DEFAULT_ADMIN_ROLE), initialOwner); // initialOwner is on a remote chain
-        owner = initialOwner;
     }

-    function mint(address to, uint256 amount) public onlyCrossChainSender(owner) {
+    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {

-    function _authorizeUpgrade(address newImplementation) internal override onlyCrossChainSender(owner) {
+    function _authorizeUpgrade(address newImplementation) internal override onlyRole(UPGRADER_ROLE) {
```

这导致以下的最终代码：
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/AccessControlCrossChainUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

abstract contract MyTokenCrossChain is Initializable, ERC20Upgradeable, AccessControlCrossChainUpgradeable, UUPSUpgradeable {
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant UPGRADER_ROLE = keccak256("UPGRADER_ROLE");

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}

    function initialize(address initialOwner) initializer public {
        __ERC20_init("MyToken", "MTK");
        __AccessControl_init();
        __UUPSUpgradeable_init();

        _grantRole(_crossChainRoleAlias(DEFAULT_ADMIN_ROLE), initialOwner); // initialOwner is on a remote chain
    }

    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }

    function _authorizeUpgrade(address newImplementation) internal onlyRole(UPGRADER_ROLE) override {
    }
}

import "@openzeppelin/contracts-upgradeable/crosschain/amb/CrossChainEnabledAMB.sol";

contract MyTokenXDAI is
    MyTokenCrossChain,
    CrossChainEnabledAMB(0x75Df5AF045d91108662D8080fD1FEFAd6aA0bb59)
{}

import "@openzeppelin/contracts-upgradeable/crosschain/optimismCrossChainEnabledOptimism.sol";

contract MyTokenOptimism is
    MyTokenCrossChain,
    CrossChainEnabledOptimism(0x25ace71c97B33Cc4729CF772ae268934F7ab5fA1)
{}
```
