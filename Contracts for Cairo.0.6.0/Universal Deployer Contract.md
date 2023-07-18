# Universal Deployer Contract
通用部署合约（UDC）是一个单例智能合约，它包装了[部署系统调用](https://www.cairo-lang.org/docs/hello_starknet/deploying_from_contracts.html#the-deploy-system-call)，以将其暴露给任何没有实现该调用的合约，例如**账户合约。您可以将其视为用于StarkNet合约的标准化通用工厂**。

由于StarkNet没有部署交易类型，它通过遵循[标准部署器接口](https://community.starknet.io/t/snip-deployer-contract-interface/2772)并发出ContractDeployed事件，提供了一种规范的智能合约部署方式。

> TIP
有关设计决策的信息，请参阅[通用部署合约提案](https://community.starknet.io/t/universal-deployer-contract-proposal/1864)。

> NOTE
UDC地址在主网、测试网和starknet-devnet中为0x041a78e741e5af2fec34b695679bc6891742439f7afb8484ecd7766661ad02bf。该地址可能在将来发生变化。

# 目录

* 快速入门

* 部署UDC

* 用法

  * 部署者相关

  * 部署者无关

* 计算反事实地址

* API规范

## 快速入门
要部署一个合约，首先需要确保它已被声明，然后发送一个deployContract交易到Universal Deployer Contract。以下是一个Python的示例实现:
```
from nile.common import UNIVERSAL_DEPLOYER_ADDRESS

signer = MockSigner(PRIVATE_KEY)

async def deploy_contract():
    # Declare the contract and get the class hash before deployment
    class_hash, _ = await signer.declare_class(account, "Ownable")

    # Set up deployment parameters
    salt = 123456  # random number
    unique = FALSE
    calldata = [account.contract_address]
    params = [class_hash, salt, unique, len(calldata), *calldata]

    # Send a "deployContract" transaction to the deployer contract
    await account.send_transaction(
        account,
        UNIVERSAL_DEPLOYER_ADDRESS,
        "deployContract",
        params
    )
```

### Deploying the UDC
通用部署器合约已经在大多数网络和开发环境上部署完成。部署UDC需要将deploy_from_zero=TRUE传递给部署系统调用。这将在所有StarkNet实例中产生确定性和可预测的地址，便于SDK集成和部署的可复现性。

## Usage
通用部署器合约提供了两种类型的地址供部署：依赖部署者和独立于部署者。顾名思义，依赖部署者类型在地址计算中包括部署者的地址；而独立于部署者类型则不包括。唯一的布尔参数最终决定了部署的类型。

> CAUTION
当部署一个在构造函数calldata中使用get_caller_address的合约时，请记住是UDC（而不是账户）部署该合约。因此，在合约的构造函数中查询get_caller_address将返回UDC的地址，而不是账户的地址。

### Deployer dependent
通过使部署依赖于部署者地址，用户可以通过其他人占用来“预留”整个地址空间。只有该部署者才能部署到这些地址。

实现这种类型的部署需要在deployContract调用中将unique设置为TRUE。在底层，函数调用利用部署者的地址，并通过使用给定的salt对部署者的地址进行哈希来创建哈希链。

要部署一个唯一的合约地址:
```
from nile.common import UNIVERSAL_DEPLOYER_ADDRESS

signer = MockSigner(PRIVATE_KEY)

async def deploy_contract():
    # Declare the contract and get the class hash before deployment
    class_hash, _ = await signer.declare_class(account, "Ownable")

    # Set up deployment parameters
    salt = 123456  # random number
    unique = TRUE
    calldata = [account.contract_address]
    params = [class_hash, salt, unique, len(calldata), *calldata]

    # Send a "deployContract" transaction to the deployer contract
    await account.send_transaction(
        account,
        UNIVERSAL_DEPLOYER_ADDRESS,
        "deployContract",
        params
    )
```

### Deployer independent
与部署者无关的合约部署是指创建与部署者和UDC实例无关的合约地址。只有类哈希、盐和构造函数参数决定了地址。这种类型的部署使得在多个网络上重新部署账户和已知系统成为可能。要部署一个可复制的部署，请将unique设置为FALSE。
```
from nile.common import UNIVERSAL_DEPLOYER_ADDRESS

signer = MockSigner(PRIVATE_KEY)

async def deploy_contract():
    # Declare the contract and get the class hash before deployment
    class_hash, _ = await signer.declare_class(account, "Ownable")

    # Set up deployment parameters
    salt = 123456  # random number
    unique = FALSE
    calldata = [account.contract_address]
    params = [class_hash, salt, unique, len(calldata), *calldata]

    # Send a "deployContract" transaction to the deployer contract
    await account.send_transaction(
        account,
        UNIVERSAL_DEPLOYER_ADDRESS,
        "deployContract",
        params
    )
```

## Calculating counterfactual addresses
反事实地址是尚未部署的合约地址。计算合约的反事实地址的一个重要用例是部署账户合约。请参阅*反事实部署*。

要预测反事实地址，可以使用StarkWare库的calculate_contract_address_from_hash函数，并传递与实际部署相同的参数。例如：
```
from starkware.starknet.core.os.contract_address.contract_address import (
    calculate_contract_address_from_hash,
)

expected_address = calculate_contract_address_from_hash(
    salt=salt,
    class_hash=class_hash,
    constructor_calldata=calldata,
    deployer_address=deployer_address
)
```

## API specification

### Methods
```
func deployContract(
    classHash: felt,
    salt: felt,
    unique: felt,
    calldata_len: felt,
    calldata: felt*
) -> (address: felt) {
}
```

#### deployContract
部署一个合约通过通用部署合约。

参数:
```
classHash: felt
salt: felt
unique: felt
calldata_len: felt
calldata: felt*
```
返回：
```
address: felt
```

### Events
```
func ContractDeployed(
    address: felt,
    deployer: felt,
    unique: felt,
    classHash: felt,
    calldata_len: felt,
    calldata: felt*,
    salt: felt
) {
}
```

#### ContractDeployed
当部署者通过Universal Deployer Contract部署合约时发出。

参数:
```
address: felt,
deployer: felt,
unique: felt,
classHash: felt,
calldata_len: felt,
calldata: felt*,
salt: felt
```