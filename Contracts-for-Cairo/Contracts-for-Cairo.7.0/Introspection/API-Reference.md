# Introspection
关于[类型自省](https://en.wikipedia.org/wiki/Type_introspection)的接口和实用程序的参考。

## 核心

### [ISRC5](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/introspection/interface.cairo#L7)
```
use openzeppelin::introspection::interface::ISRC5;
```

[SNIP-5](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md)中定义的SRC5自省标准的接口。

**SRC5 ID**
0x3f918d17e5ee77373b56385708f855659a07f75997f365cf87748628532a055

**FUNCTIONS**
supports_interface(interface_id)

#### Functions

##### supports_interface(interface_id: felt252) → bool
external#
检查合约是否实现了给定的接口。

> TIP
查看*计算接口ID*以获取更多关于如何计算此ID的信息。

### [SRC5](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/introspection/src5.cairo)
```
use openzeppelin::introspection::src5::SRC5;
```

**EXTERNAL FUNCTIONS**
SRC5IMPL
supports_interface(self, interface_id)

**INTERNAL FUNCTIONS**
INTERNALIMPL
register_interface(self, interface_id)

deregister_interface(self, interface_id)

#### External Functions

##### supports_interface(self: @ContractState, interface_id: felt252) → bool
external#
请查阅 *ISRC5::supports_interface*.

#### Internal Functions

##### register_interface(ref self: ContractState, interface_id: felt252)
internal#
支持给定接口ID的寄存器。

##### deregister_interface(ref self: ContractState, interface_id: felt252)
internal#
取消对给定接口ID的支持。