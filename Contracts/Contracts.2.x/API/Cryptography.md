# Cryptography
这个库的集合提供了使用不同加密原语的简单和安全的方法。

## Libraries

### ECDSA
椭圆曲线数字签名算法（ECDSA）操作。

这些函数可用于验证消息是否由给定地址的私钥持有者签名。

**FUNCTIONS**
[recover(hash, signature)](#recoverbytes32-hash-bytes-signature-→-address)

[toEthSignedMessageHash(hash)](#toethsignedmessagehashbytes32-hash-→-bytes32)

#### recover(bytes32 hash, bytes signature) → address
内部#
返回使用签名对哈希消息（hash）进行签名的地址。该地址可以用于验证目的。

ecrecover EVM操作码允许可塑（非唯一）签名：该函数通过要求s值在低半序中，并且v值为27或28来拒绝它们。

> NOTE
如果签名无效或无法检索签名者，此调用不会回滚。在这些情况下，将返回零地址。

> IMPORTANT
hash必须是用于确保验证安全性的哈希操作的结果：可以制作恢复到任意地址的签名，用于非哈希数据。确保这一点的安全方法是接收原始消息的哈希（可能太长），然后对其调用[toEthSignedMessageHash](#toethsignedmessagehashbytes32-hash-→-bytes32)。

#### toEthSignedMessageHash(bytes32 hash) → bytes32
内部#
返回一个由哈希创建的以太坊签名消息。这个函数复制了[eth_sign](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sign) JSON-RPC方法的行为。

参见[recover](#recoverbytes32-hash-bytes-signature-→-address)。

### MerkleProof
这些函数处理Merkle树（哈希树）的验证。

**FUNCTIONS**
[verify(proof, root, leaf)](#verifyvar-typebytes32-proof-bytes32-root-bytes32-leaf-→-bool)

#### verify([.var-type]#bytes32[# proof, bytes32 root, bytes32 leaf) → bool]
内部#
如果可以证明一个叶子节点是由根节点定义的Merkle树的一部分，则返回true。为此，必须提供一个证明，其中包含从叶子节点到树根的分支上的兄弟节点哈希。假设每对叶子节点和每对原始数据都已排序。