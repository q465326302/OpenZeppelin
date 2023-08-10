# Utilities
OpenZeppelin Contracts提供了许多实用工具，您可以在项目中使用。以下是一些比较流行的工具。

## Cryptography

### 在链上检查签名
[ECDSA](./API/Utils.md#ecdsa)提供了用于复原和管理以太坊账户ECDSA签名的函数。这些签名通常是通过[web3.eth.sign](https://web3js.readthedocs.io/en/v1.7.3/web3-eth.html#sign)生成的，是一个65字节的数组（在Solidity中是bytes类型），排列方式如下：[[v(1)],[r(32)],[s(32)]]。

数据签名者可以通过[ECDSA.recover](./API/Utils.md#recoverbytes32-hash-bytes-signature-→-address)复原，并将其地址与签名进行比较以验证签名。大多数钱包会对要签名的数据进行哈希处理，并添加前缀'\x19Ethereum Signed Message:\n'，因此在尝试复原以太坊签名消息哈希的签名者时，您需要使用[toEthSignedMessageHash](./API/Utils.md#toethsignedmessagehashbytes32-hash-→-bytes32-message)。
```
using ECDSA for bytes32;

function _verify(bytes32 data, bytes memory signature, address account) internal pure returns (bool) {
    return data
        .toEthSignedMessageHash()
        .recover(signature) == account;
}
```

> WARNING
正确验证签名不容易：确保您完全阅读并理解 [ECDSA](./API/Utils.md#ecdsa) 的文档。

### 验证 Merkle 证明
[MerkleProof](./API/Utils.md#merkleproof) 提供了：

* [verify](./API/Utils.md#verifybytes32-proof-bytes32-root-bytes32-leaf-→-bool) - 可以证明某个值是 [Merkle 树](https://en.wikipedia.org/wiki/Merkle_tree)的一部分。

* [multiProofVerify](./API/Utils.md) - 可以证明多个值是 Merkle 树的一部分。

## Introspection
在 Solidity 中，经常需要知道一个合约是否支持您想要使用的接口。ERC165是一个帮助进行运行时接口检测的标准。合约提供了用于在您的合约中实现ERC165和查询其他合约的帮助函数：

* [IERC165](./API/Utils.md#ierc165) - 这是定义 [supportsInterface](./API/Utils.md#supportsinterfacebytes4-interfaceid-→-bool) 的 ERC165 接口。在实现 ERC165 时，您将符合此接口。

* [ERC165](./API/Utils.md#erc165) - 如果您想使用合约存储中的查找表支持接口检测，请继承此合约。您可以使用 [_registerInterface(bytes4)](./API/Utils.md#_registerinterfacebytes4-interfaceid) 注册接口：请查看作为 ERC721 实现的示例用法。

* [ERC165Checker](./API/Utils.md#erc16checker) - ERC165Checker 简化了检查合约是否支持您所需接口的过程。

* 包括使用 ERC165Checker for address;

* [myAddress._supportsInterface(bytes4)](./API/Utils.md)

* [myAddress._supportsAllInterfaces(bytes4[])](./API/Utils.md)
```
contract MyContract {
    using ERC165Checker for address;

    bytes4 private InterfaceId_ERC721 = 0x80ac58cd;

    /**
     * @dev transfer an ERC721 token from this contract to someone else
     */
    function transferERC721(
        address token,
        address to,
        uint256 tokenId
    )
        public
    {
        require(token.supportsInterface(InterfaceId_ERC721), "IS_NOT_721_TOKEN");
        IERC721(token).transferFrom(address(this), to, tokenId);
    }
}
```

## Math
OpenZeppelin Contracts提供的最受欢迎的与数学相关的库是[SafeMath](./API/Utils.md#safemath)，它提供了数学函数，保护您的合约免受溢出和下溢的影响。

使用SafeMath for uint256;将合约包含进来，然后调用函数：

* myNumber.add(otherNumber)
* myNumber.sub(otherNumber)
* myNumber.div(otherNumber)
* myNumber.mul(otherNumber)
* myNumber.mod(otherNumber)
很容易！

## Payment

想要将一些代币支付分配给多个人吗？也许您有一个应用程序，可以将艺术品收益的30％发送给原始创建者，再将70％的收益发送给当前所有者。您可以使用[PaymentSplitter](./API/Finance.md#paymentsplitter)构建它！

在Solidity中，盲目向帐户发送资金会存在一些安全问题。您可以在[以太坊智能合约最佳实践网站](https://consensys.github.io/smart-contract-best-practices/)上阅读有关这些安全问题的信息。修复重入和停滞问题的一种方法是，不立即将以太币发送到需要的账户，而是使用[PullPayment](./API/Security.md#pullpayment)，提供一个[_asyncTransfer](./API/Security.md#_asynctransferaddress-dest-uint256-amount)函数，用于将资金发送给某个地址，并请求稍后通过[withdrawPayments()](./API/Security.md#withdrawpaymentsaddress-payable-payee)来提取。

如果您想托管一些资金，请查看[Escrow](./API/Utils.md#escrow)和[ConditionalEscrow](./API/Utils.md#conditionalescrow)，用于管理托管的以太币。

## Collections

如果您需要比Solidity的原生数组和映射更强大的集合支持，请查看[EnumerableSet](./API/Utils.md#enumerableset)和[EnumerableMap](./API/Utils.md#enumerablemap)。它们类似于映射，它们以规定的时间存储和删除元素，且不允许重复条目，但还支持枚举，这意味着您可以轻松地在链上和链下查询所有存储的条目。

## Misc
想要检查地址是否是合约吗？使用[Address](./API/Utils.md#address)和[Address.isContract()](./API/Utils.md#iscontractaddress-account-→-bool)。

想要跟踪递增的数字吗？请查看[Counters](./API/Utils.md#counters)。这经常用得上，例如创建递增标识符，如[ERC721指南](./Tokens/ERC721.md)所示。

### Base64
[Base64](./API/Utils.md#base64)工具允许您将bytes32数据转换为其Base64字符串表示形式。

这对于构建URL安全的[ERC721](./API/ERC721.md#ierc721enumerable)或[ERC1155](./API/ERC1155.md#uriuint256-id-→-string)的tokenURI特别有用。该库提供了一种巧妙的方式，提供符合[安全数据的URI](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/Data_URIs/)字符串，在链上数据结构中使用。

以下是使用ERC721通过Base64数据URI发送JSON元数据的示例：
```
// contracts/My721Token.sol
// SPDX-License-Identifier: MIT

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/Base64.sol";

contract My721Token is ERC721 {
    using Strings for uint256;

    constructor() ERC721("My721Token", "MTK") {}

    ...

    function tokenURI(uint256 tokenId)
        public
        pure
        override
        returns (string memory)
    {
        bytes memory dataURI = abi.encodePacked(
            '{',
                '"name": "My721Token #', tokenId.toString(), '"',
                // Replace with extra ERC721 Metadata properties
            '}'
        );

        return string(
            abi.encodePacked(
                "data:application/json;base64,",
                Base64.encode(dataURI)
            )
        );
    }
}
```

### Multicall
Multicall抽象合约有一个multicall函数，它将多个调用捆绑在单个外部调用中。使用它，外部账户可以执行由多个函数调用组成的原子操作。这不仅在EOA在单个交易中进行多个调用很有用，而且还是后续调用失败时回滚之前调用的一种方式。

考虑这个虚拟合约：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Multicall.sol";

contract Box is Multicall {
    function foo() public {
        ...
    }

    function bar() public {
        ...
    }
}
```

这是使用Truffle调用multicall函数，允许在单个交易中调用foo和bar：
```
// scripts/foobar.js

const Box = artifacts.require('Box');
const instance = await Box.new();

await instance.multicall([
    instance.contract.methods.foo().encodeABI(),
    instance.contract.methods.bar().encodeABI()
]);
```
