# Upgrades
关于可升级性的接口和实用程序的参考。

## 核心

### [IUpgradeable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/upgrades/interface.cairo#L3)

```
use openzeppelin::upgrades::interface::IUpgradeable;
```

可升级合约的接口。

**FUNCTIONS**
[upgrade(new_class_hash)](#upgradenew_class_hash-classhash)

#### Functions
##### upgrade(new_class_hash: ClassHash)
external#
此函数通过更新其[类哈希](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/class-hash/)来升级合约代码。

> NOTE
这个功能通常受到*访问控制机制*的保护。

### [Upgradeable](https://github.com/OpenZeppelin/cairo-contracts/blob/release-v0.7.0/src/upgrades/upgradeable.cairo)
```
use openzeppelin::upgrades::upgradeable::Upgradeable;
```

可升级合约模块。

**INTERNAL FUNCTIONS**
[_upgrade(self, new_class_hash)](#_upgraderef-self-contractstate-new_class_hash-classhash)

**EVENTS**
[Upgraded(class_hash)](#upgradedclass_hash-classhash)

#### Internal Functions
##### _upgrade(ref self: ContractState, new_class_hash: ClassHash)
internal#
通过更新合约[类哈希](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/class-hash/)来升级合约。

要求：
* 新的类哈希必须与零不同。
触发一个Upgraded事件。

#### Events
##### Upgraded(class_hash: ClassHash)
event#
当[类哈希](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/class-hash/)升级时发出。