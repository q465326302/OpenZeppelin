# Cryptography
这个库集合提供了使用不同加密原语的简单和安全的方法。

以下相关的EIP目前处于草案状态，可以在草案目录中找到。

* [EIP712](./Drafts.md#eip712)

## 库

### ECDSA
椭圆曲线数字签名算法（ECDSA）操作。

这些函数可以用于验证一条消息是否由给定地址的私钥持有者签名。

**FUNCTIONS**
[recover(hash, signature)](#recoverbytes32-hash-bytes-signature-→-address)

[recover(hash, v, r, s)](#recoverbytes32-hash-uint8-v-bytes32-r-bytes32-s-→-address)

[toEthSignedMessageHash(hash)](#toethsignedmessagehashbytes32-hash-→-bytes32)

#### recover(bytes32 hash, bytes signature) → address
内部#
返回使用签名对哈希消息（哈希）进行签名的地址。然后可以使用该地址进行验证。

ecrecover EVM操作码允许存在可塑性（非唯一性）签名：该函数通过要求s值在较低的半序中，并且v值为27或28来拒绝它们。

> IMPORTANT
为了确保验证的安全性，哈希必须是用于哈希操作的结果：可以通过为非哈希数据恢复到任意地址的签名。一种安全的方法是接收原始消息的哈希（否则可能太长），然后对其调用[toEthSignedMessageHash](#toethsignedmessagehashbytes32-hash-→-bytes32)。

#### recover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) → address
内部#
接收v、r和s签名字段分别的{ECDSA-recover-bytes32-bytes-}的过载函数。

#### toEthSignedMessageHash(bytes32 hash) → bytes32
内部#
返回一个以哈希值创建的以太坊签名消息。这复制了[eth_sign](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_sign) JSON-RPC方法的行为。

参见[recover](#recoverbytes32-hash-uint8-v-bytes32-r-bytes32-s-→-address。

### MerkleProof

这些函数处理Merkle树（哈希树）的验证。

**FUNCTIONS**
[verify(proof, root, leaf)](#verifybytes32-proof-bytes32-root-bytes32-leaf-→-bool)

#### verify(bytes32[] proof, bytes32 root, bytes32 leaf) → bool
内部#
如果可以证明一个叶子节点是由根节点定义的Merkle树的一部分，则返回true。为此，必须提供一个证明，其中包含从叶子节点到树根的分支上的兄弟节点的哈希值。假设每对叶子节点和每对原始数据都是排序的。
