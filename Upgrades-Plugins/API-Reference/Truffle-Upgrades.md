# OpenZeppelin Truffle Upgrades API
deployProxy函数和upgradeProxy函数都会返回Truffle合约的实例，并且需要使用artifacts.require检索到的[Truffle合约类](https://www.trufflesuite.com/docs/truffle/reference/contract-abstractions)作为参数。对于*beacon*，deployBeacon和upgradeBeacon都将返回一个可用于beacon代理的可升级beacon实例。所有的部署和升级函数都会验证实现合约是否具备升级安全性，否则将失败。

## 常见选项
以下选项适用于一些函数。

* deployer: 在迁移过程中应设置为Truffle迁移部署器。

* kind: ("uups" | "transparent" | "beacon") 要部署、升级或导入的代理的类型，或者要与实现合约一起使用的代理的类型。deployProxy()和upgradeProxy()只支持值"uups" | "transparent"。默认为"transparent"。参见*Transparent vs UUPS*。

* unsafeAllow: (ValidationError[]) 选择性地禁用一个或多个验证错误：
    * "external-library-linking"：允许部署与实现合约链接的外部库。 (否则不支持外部库。)
    * "struct-definition"，"enum-definition"：曾经需要部署带有结构体或枚举的合约。现在不再需要。
    * "state-variable-assignment"：允许在合约中分配状态变量，尽管它们将存储在实现中。
    * "state-variable-immutable"：允许使用不安全的不可变变量
    * "constructor"：允许定义一个构造函数。参见constructorArgs。
    * "delegatecall"，"selfdestruct"：允许使用这些操作。不正确使用此选项可能会导致资金永久丢失。参见*Can I safely use delegatecall and selfdestruct?*

* "missing-public-upgradeto"：允许不包含公共upgradeTo函数的UUPS实现。启用此选项可能会导致回滚，因为内置的UUPS安全机制。

* unsafeAllowRenames: (boolean) 配置存储布局检查以允许变量重命名。

* unsafeSkipStorageCheck: (boolean) 在不先检查存储布局兼容性错误的情况下升级代理或beacon。这是一个危险的选项，应作为最后的手段使用。

* constructorArgs: (unknown[]) 为实现合约的构造函数提供参数。请注意，这些参数与初始化器参数不同，并将用于实现合约本身的部署。可用于初始化不可变变量。

* redeployImplementation: ("always" | "never" | "onchange") 决定是否重新部署实现合约。默认为"onchange"。
    * 如果设置为"always"，即使使用相同的字节码之前已部署了实现合约，也始终重新部署实现合约。
    * 如果设置为"never"，则永不重新部署实现合约。如果之前未部署实现合约或在网络文件中找不到实现合约，将抛出错误。
    * 如果设置为"onchange"，仅当字节码与先前部署的实现合约不同时，才重新部署实现合约。

请注意，选项unsafeAllow也可以在源代码中以更细粒度的方式直接指定（如果使用Solidity >=0.8.2）。参见*How can I disable some of the checks?*

以下选项已被弃用。

* unsafeAllowLinkedLibraries: 等同于在unsafeAllow中包含"external-library-linking"。

* unsafeAllowCustomTypes: 等同于在unsafeAllow中包含"struct-definition"和"enum-definition"。不再需要。

* useDeployedImplementation: (boolean) 等同于将redeployImplementation设置为"never"。

## 代理合约
```
async function deployProxy(
  Contract: ContractClass,
  args: unknown[] = [],
  opts?: {
    deployer?: Deployer,
    initializer?: string | false,
    unsafeAllow?: ValidationError[],
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    kind?: 'uups' | 'transparent',
  },
): Promise<ContractInstance>
```
创建一个 UUPS 或透明代理，给定一个用作实现的 Truffle 合约类，并返回一个带有代理地址和实现接口的合约实例。在迁移过程中，代理地址将存储在实现合约的构件中，因此您可以使用 Truffle 的 [deployed()](https://www.trufflesuite.com/docs/truffle/reference/contract-abstractions#-code-mycontract-deployed-code-) 函数加载它。

如果设置了 args，将在代理部署期间使用提供的 args 调用初始化函数 initialize。

如果对同一个实现合约多次调用 deployProxy，将部署多个代理，但只使用一个实现合约。

**参数：**

* Contract - 用作实现的 Truffle 合约类。

* args - 初始化函数的参数。

* opts - 一个带有选项的对象：
    * initializer：设置一个不同的初始化函数进行调用，或者指定 false 禁用初始化。
    * 参见*常见选项*。

**返回：**

* 一个带有代理地址和实现接口的合约实例。

## 升级代理合约
```
async function upgradeProxy(
  proxy: string | ContractInstance,
  Contract: ContractClass,
  opts?: {
    deployer?: Deployer,
    call?: string | { fn: string; args?: unknown[] },
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    kind?: 'uups' | 'transparent',
  },
): Promise<ContractInstance>
```

将指定地址的UUPS或透明代理升级为新的实现合约，并返回一个带有代理地址和新实现接口的合约实例。

**参数：**

* proxy - 代理地址或代理合约实例。

* Contract - 用作新实现的Truffle合约类。

* opts - 一个带有选项的对象：

    * call：在升级过程中启用执行任意函数调用。该调用使用函数名或签名以及可选参数进行描述。它被批处理到升级事务中，可以安全地调用迁移初始化函数。
    * 参见*常见选项*。

**返回：**

* 一个带有代理地址和新实现接口的合约实例。

## 部署信标
```
async function deployBeacon(
  Contract: ContractClass,
  opts?: {
    deployer?: Deployer,
    unsafeAllow?: ValidationError[],
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
  },
): Promise<ContractInstance>
```
创建一个*可升级的信标*，给定一个用作实现的Truffle合约类，并返回信标合约实例。

**参数：**

* Contract - 用作实现的Truffle合约类。

* opts - 一个包含选项的对象：
    * 参见*常见选项*。

**返回值：**

* 信标合约实例。

**自版本：**

* @openzeppelin/truffle-upgrades@1.12.0

## 升级信标
```
async function upgradeBeacon(
  beacon: string | ContractInstance,
  Contract: ContractClass,
  opts?: {
    deployer?: Deployer,
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
  },
): Promise<ContractInstance>
```
创建一个*可升级的信标（beacon）*，并返回信标合约实例。

**参数：**

* Contract - 用作实现的 Truffle 合约类。

* opts - 一个包含选项的对象：
    * 参见*常见选项*。

**返回：**

* 信标合约实例。

**自版本：**

* @openzeppelin/truffle-upgrades@1.12.0

## 升级信标
```
async function upgradeBeacon(
  beacon: string | ContractInstance,
  Contract: ContractClass,
  opts?: {
    deployer?: Deployer,
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
  },
): Promise<ContractInstance>
```
升级指定地址的*可升级信标*到一个新的实现合约，并返回信标合约实例。

**参数：**

* beacon - 信标地址或信标合约实例。

* Contract - 作为新实现的 Truffle 合约类。

* opts - 一个带有选项的对象：
    * 参见常用选项。

**返回：**

* 信标合约实例。

**自版本：**

* @openzeppelin/truffle-upgrades@1.12.0

## 部署BeaconProxy
```
async function deployBeaconProxy(
  beacon: string | ContractInstance,
  attachTo: ContractClass,
  args: unknown[] = [],
  opts?: {
    deployer?: Deployer,
    initializer?: string | false,
  },
): Promise<ContractInstance>
```
给定一个现有的beacon合约地址和与beacon当前实现合约对应的Truffle合约类，创建一个*Beacon代理*，并返回一个包含beacon代理地址和实现接口的合约实例。如果设置了args，则在代理部署期间使用提供的args调用一个初始化函数initialize。

**参数：**

* beacon - beacon地址或beacon合约实例。

* attachTo - 与beacon当前实现合约对应的Truffle合约类。

* args - 初始化函数的参数。

* opts - 一个带有选项的对象：
    * initializer：设置一个不同的初始化函数进行调用，或者指定false来禁用初始化。

**返回：**

* 一个包含beacon代理地址和实现接口的合约实例。

**自版本：**

* @openzeppelin/truffle-upgrades@1.12.0

## 强制导入
```
async function forceImport(
  address: string,
  deployedImpl: ContractClass,
  opts?: {
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<ContractInstance>
```

强制导入现有代理、beacon或实现合约部署，以便与此插件一起使用。提供现有代理、beacon或实现的地址，以及部署的实现合约的Truffle合约类。

> CAUTION
当导入代理或beacon时，deployedImpl参数必须是**当前正在**使用的实现合约版本的合约类，而不是您计划升级到的版本。

使用此函数通过导入先前的部署来重新创建丢失的*网络文件*，或者即使它们最初不是由此插件部署的，也可以注册代理或beacon进行升级。支持UUPS、Transparent和Beacon代理，以及beacon和实现合约。

**参数：**

* address - 现有代理、beacon或实现的地址。

* deployedImpl - 部署的实现合约的Truffle合约类。

* opts - 一个带有选项的对象：

    * kind: ("uups" | "transparent" | "beacon") 强制将代理视为UUPS、Transparent或Beacon代理。如果未提供，将自动检测代理类型。

**返回：**

* 表示导入的代理、beacon或实现的合约实例。

**自版本：**

* @openzeppelin/truffle-upgrades@1.13.0

## 验证实施
```
async function validateImplementation(
  Contract: ContractClass,
  opts?: {
    unsafeAllow?: ValidationError[],
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<void>
```

验证实施合约的实施合约，而无需部署它。

**参数：**

* Contract - 实施合约的Truffle合约类。

* opts - 一个带有选项的对象：

    * 请参阅*常见选项*。

**自版本：**

* @openzeppelin/truffle-upgrades@1.16.0

## 部署实施
```
async function deployImplementation(
  Contract: ContractClass,
  opts?: {
    deployer?: Deployer,
    unsafeAllow?: ValidationError[],
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<string>
```

验证并部署一个实现合约，并返回其地址。

**参数：**

* Contract - 用作实现的Truffle合约类。

* opts - 一个包含选项的对象：
    * 参见*常见选项*。

**返回：**

* 实现合约的地址。

**自版本：**

* @openzeppelin/truffle-upgrades@1.16.0

## 验证升级
```
async function validateUpgrade(
  referenceAddressOrContract: string | ContractClass,
  newContract: ContractClass,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<void>
```

验证新的实现合约而无需部署它，也无需实际升级到它。将当前的实现合约与新的实现合约进行比较，以检查存储布局兼容性错误。如果referenceAddressOrContract是当前实现地址，则需要kind选项。

**参数：**

* referenceAddressOrContract - 使用当前实现的代理或信标地址，或与当前实现对应的地址或Truffle合约类。

* newContract - 新的实现合约。

* opts - 一个带有选项的对象：
    * 参见*常见选项*。

**自版本：**

* @openzeppelin/truffle-upgrades@1.16.0

**示例：**

验证将现有代理升级到新合约（将PROXY_ADDRESS替换为您的代理地址）：
```
const { validateUpgrade } = require('@openzeppelin/truffle-upgrades');

const BoxV2 = artifacts.require('BoxV2');
await validateUpgrade(PROXY_ADDRESS, BoxV2);
```
验证两个合约实现之间的升级：
```
const { validateUpgrade } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');
const BoxV2 = artifacts.require('BoxV2');
await validateUpgrade(Box, BoxV2);
```

## 准备升级
```
async function prepareUpgrade(
  referenceAddressOrContract: string | ContractInstance,
  Contract: ContractClass,
  opts?: {
    deployer?: Deployer,
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<string>
```

验证并部署一个新的实现合约，并返回其地址。如果referenceAddressOrContract是当前实现的地址，则kind选项是必需的。使用此方法准备从您无法直接控制或无法从Truffle使用的管理地址运行升级。

**参数：**

* referenceAddressOrContract - 代理或信标或实现地址或合约实例。

* Contract - 新的实现合约。

* opts - 一个带有选项的对象：
    * 参见常用选项。

**返回：**

* 新实现合约的地址。

## 部署ProxyAdmin
```
async function deployProxyAdmin(
  opts?: {
    deployer?: Deployer,
    timeout?: number,
    pollingInterval?: number,
  },
): Promise<string>
```
部署一个*代理管理员*合约，并返回其地址，如果当前网络上没有部署代理管理员，则返回代理管理员的地址。请注意，此插件目前只支持每个网络使用一个代理管理员。

**参数：**

* opts - 一个带有选项的对象：
    * 参见常见选项。

**返回：**

* 代理管理员的地址。

**自版本：**

* @openzeppelin/truffle-upgrades@1.16.0

## 管理员更改代理管理员
```
async function changeProxyAdmin(
  proxyAddress: string,
  newAdmin: string,
): Promise<void>
```
更改特定代理的管理员。

**参数：**

* proxyAddress - 要更改的代理的地址。

* newAdmin - 新的管理员地址。

## 管理员转移代理管理员所有权

```
async function transferProxyAdminOwnership(
  newAdmin: string,
): Promise<void>
```

更改代理管理员合约的所有者，该合约是所有代理升级权限的默认管理员。

**参数：**

* newAdmin - 新的管理员地址。

## erc1967
```
async function erc1967.getImplementationAddress(proxyAddress: string): Promise<string>;
async function erc1967.getBeaconAddress(proxyAddress: string): Promise<string>;
async function erc1967.getAdminAddress(proxyAddress: string): Promise<string>;
```

该模块中的函数提供对代理合约的[ERC1967](https://eips.ethereum.org/EIPS/eip-1967)变量的访问。

**参数：**

* proxyAddress - 代理地址。

**返回：**

* 根据调用的函数，返回实现、信标或管理员地址。

## 信标
```
async function beacon.getImplementationAddress(beaconAddress: string): Promise<string>;
```
该模块提供了一个方便的函数，用于从beacon合约中获取实现地址。

**参数：**

* beaconAddress - beacon合约地址。

**返回值：**

* 实现地址。

**自版本：**

* @openzeppelin/truffle-upgrades@1.12.0

## 静默警告
```
function silenceWarnings()
```

> NOTE
这个函数在测试中很有用，但在生产部署脚本中使用是不鼓励的。

它会静默所有后续关于使用不安全标志的警告。在执行之前会打印最后一个警告。