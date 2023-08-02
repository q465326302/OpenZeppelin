# OpenZeppelin Hardhat Upgrades API

deployProxy和upgradeProxy函数都会返回[ethers.js合约](https://docs.ethers.io/v5/api/contract/contract)的实例，并且需要[ethers.js合约工厂](https://docs.ethers.io/v5/api/contract/contract-factory)作为参数。对于beacon，deployBeacon和upgradeBeacon都将返回一个可用于*beacon*代理的可升级beacon实例。所有的deploy和upgrade函数都会验证实现合约是否可以进行升级，否则将失败。

## 常见选项
以下选项适用于某些函数。

* kind: ("uups" | "transparent" | "beacon") 部署、升级或导入的代理类型，或者实现将使用的代理类型。deployProxy()和upgradeProxy()只支持值"uups" | "transparent"。默认为"transparent"。参见*Transparent vs UUPS*。

* unsafeAllow: (ValidationError[]) 选择性地禁用一个或多个验证错误：

    * "external-library-linking"：允许部署与实现合约链接的外部库。 （否则尚*不支持外部库*。）

    * "struct-definition"、"enum-definition"：以前需要部署带有结构体或枚举的合约。不再需要。

    * "state-variable-assignment"：允许在合约中分配状态变量，即使它们将存储在实现中。

    * "state-variable-immutable"：允许使用不安全的不可变变量

    * "constructor"：允许定义构造函数。参见constructorArgs。

    * "delegatecall"、"selfdestruct"：允许使用这些操作。不正确使用此选项可能会导致资金永久丢失。参见*Can I safely use delegatecall and selfdestruct?*

    * "missing-public-upgradeto"：允许不包含公共upgradeTo函数的UUPS实现。启用此选项可能会导致回滚，因为内置的UUPS安全机制。

* unsafeAllowRenames: (boolean) 配置存储布局检查以允许变量重命名。

* unsafeSkipStorageCheck: (boolean) 在未首先检查存储布局兼容性错误的情况下升级代理或beacon。这是一个危险的选项，应作为最后的手段使用。

* constructorArgs: (unknown[]) 为实现合约的构造函数提供参数。注意，这些参数与初始化器参数不同，并将用于实现合约本身的部署。可用于初始化不可变变量。

* timeout: (number) 部署实现合约或代理管理员合约时等待事务确认的超时时间（以毫秒为单位）。默认为60000。使用0表示无限等待。

* pollingInterval: (number) 部署实现合约或代理管理员合约时检查事务确认之间的轮询间隔（以毫秒为单位）。默认为5000。

* redeployImplementation: ("always" | "never" | "onchange") 确定是否重新部署实现合约。默认为"onchange"。

    * 如果设置为"always"，即使以前使用相同的字节码部署了实现合约，也始终重新部署实现合约。当通过OpenZeppelin Platform部署代理时，可以与salt选项一起使用，以确保实现合约与代理使用相同的salt部署。

    * 如果设置为"never"，则永远不会重新部署实现合约。如果以前未部署实现合约或在网络文件中找不到实现合约，则会抛出错误。

    * 如果设置为"onchange"，则仅当字节码与先前部署的实现合约不同时，才重新部署实现合约。

* usePlatformDeploy: (boolean) 使用OpenZeppelin Platform部署合约，而不是ethers.js。参见*Using with OpenZeppelin Platform*。注意：OpenZeppelin Platform处于测试版，与Platform相关的功能可能会发生变化。

* verifySourceCode: (boolean) 在使用Platform时，是否在区块浏览器上验证源代码。默认为true。

* relayerId: (string) 在使用Platform时，要用于部署的Relayer器的ID。默认为在Platform上配置的部署环境的Relayer器。

* salt: (string) 在使用Platform时，使用CREATE2操作码执行部署。使用此选项提供部署的salt。默认为随机salt。

注意，选项unsafeAllow也可以在Solidity >=0.8.2中直接在源代码中以更细粒度的方式指定。参见*How can I disable some of the checks?*

以下选项已弃用。

* unsafeAllowLinkedLibraries: 相当于在unsafeAllow中包含"external-library-linking"。

* unsafeAllowCustomTypes: 相当于在unsafeAllow中包含"struct-definition"和"enum-definition"。不再需要。

* useDeployedImplementation: 相当于将redeployImplementation设置为"never"。

## 部署代理
```
async function deployProxy(
  Contract: ethers.ContractFactory,
  args: unknown[] = [],
  opts?: {
    initializer?: string | false,
    unsafeAllow?: ValidationError[],
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    kind?: 'uups' | 'transparent',
    usePlatformDeploy?: boolean,
  },
): Promise<ethers.Contract>
```
创建一个UUPS或透明代理，给定一个以太坊合约工厂作为实现，返回一个具有代理地址和实现接口的合约实例。如果设置了args参数，则在代理部署期间调用一个带有提供的args的初始化函数initialize。

如果多次调用deployProxy来使用相同的实现合约，将会部署多个代理，但只使用一个实现合约。

**参数：**

* Contract - 以太坊合约工厂，用作实现。

* args - 初始化函数的参数。

* opts - 一个带有选项的对象：

    * initializer: 设置不同的初始化函数来调用（参见[指定片段](https://docs.ethers.io/v5/api/utils/abi/interface/#Interface--specifying-fragments)），或者指定false来禁用初始化。

    * 如*Common Options*中所述的附加选项。

**返回：**

* 一个具有代理地址和实现接口的合约实例。

## upgradeProxy
```
async function deployBeacon(
  Contract: ethers.ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
  },
): Promise<ethers.Contract>
```

升级指定地址的UUPS或透明代理到一个新的实现合约，并返回一个具有代理地址和新实现接口的合约实例。

*参数：*

* proxy - 代理地址或代理合约实例。

* Contract - 用作新实现的ethers合约工厂。

* opts - 一个带有选项的对象：

    * call: 在升级过程中允许执行任意函数调用。这个调用可以使用函数名、签名或选择器（参见指定片段）来描述，并且可以包含可选参数。它被批处理到升级事务中，可以安全地调用迁移初始化函数。

    * 根据Common Options中的描述，额外的选项包括：

**返回：**

一个具有代理地址和新实现接口的合约实例。

## deployBeacon
```
async function deployBeacon(
  Contract: ethers.ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
  },
): Promise<ethers.Contract>
```
创建一个*可升级的信标（beacon）*，给定一个以太坊合约工厂作为实现，并返回信标合约实例。

**参数：**

* Contract - 一个以太坊合约工厂，用作实现。

* opts - 一个包含选项的对象：

    * 其他选项可以参考*Common Options*。

**返回：**

* 信标合约实例。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.13.0

## upgradeBeacon
```
async function upgradeBeacon(
  beacon: string | ethers.Contract,
  Contract: ethers.ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
  },
): Promise<ethers.Contract>
```
升级指定地址的*可升级信标*到一个新的实现合约，并返回信标合约实例。

**参数：**

* beacon - 信标地址或信标合约实例。
* Contract - 用作新实现的ethers合约工厂。
* opts - 一个包含选项的对象：
    * 其他选项，如Common Options中所述。

**返回值：**

* 信标合约实例。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.13.0

## deployBeaconProxy
```
async function deployBeaconProxy(
  beacon: string | ethers.Contract,
  attachTo: ethers.ContractFactory,
  args: unknown[] = [],
  opts?: {
    initializer?: string | false,
    usePlatformDeploy?: boolean,
  },
): Promise<ethers.Contract>
```
给定一个现有的*beacon合约*地址和与beacon当前实现合约对应的ethers合约工厂，创建一个beacon代理，并返回一个包含beacon代理地址和实现接口的合约实例。如果设置了args，将在代理部署期间使用提供的args调用一个初始化函数initialize。

**参数:**

* beacon - beacon地址或beacon合约实例。

* attachTo - 与beacon当前实现合约对应的ethers合约工厂。

* args - 初始化函数的参数。

* opts - 一个带有选项的对象:

    * initializer: 设置一个不同的初始化函数来调用(参见[指定片段](https://docs.ethers.io/v5/api/utils/abi/interface/#Interface%E2%80%94%E2%80%8Bspecifying-fragments))，或者指定false来禁用初始化。

附加选项是指在“常见选项”中所描述的*其他选项*。

**返回:**

* 一个包含beacon代理地址和实现接口的合约实例。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.13.0

## forceImport
```
async function forceImport(
  address: string,
  deployedImpl: ethers.ContractFactory,
  opts?: {
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<ethers.Contract>
```
强制导入现有的代理、信标或实现合约部署，用于与此插件一起使用。提供现有代理、信标或实现的地址，以及已部署的实现合约的 ethers 合约工厂。

> CATION
当导入代理或信标时，deployedImpl 参数必须是**当前**正在使用的实现合约版本的合约工厂，而不是您计划升级到的版本。

使用此函数通过导入先前的部署来重新创建丢失的*网络文件*，或者即使这些代理或信标最初不是由此插件部署的，也可以注册代理或信标以进行升级。支持 UUPS、Transparent 和 Beacon 代理，以及信标和实现合约。

**参数：**

* ddress - 现有代理、信标或实现的地址。

* deployedImpl - 已部署的实现合约的 ethers 合约工厂。

* opts - 一个带有选项的对象：

    * kind: ("uups" | "transparent" | "beacon") 强制将代理视为 UUPS、Transparent 或 Beacon 代理。如果未提供，将自动检测代理的类型。

**返回：**

* 表示导入的代理、信标或实现的合约实例。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.15.0

## validateImplementation
```
async function validateImplementation(
  Contract: ethers.ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<void>
```
在不部署实现合约的情况下验证一个实现合约的实施。

**参数:**

* Contract - 实现合约的以太坊合约工厂。

* opts - 一个包含选项的对象:

    * 其他选项如Common Options中所述。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.20.0

## deployImplementation
```
async function deployImplementation(
  Contract: ethers.ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    getTxResponse?: boolean,
    kind?: 'uups' | 'transparent' | 'beacon',
    usePlatformDeploy?: boolean,
  },
): Promise<string | ethers.providers.TransactionResponse>
```
验证和部署一个实现合约，并返回其地址。

**参数：**

* Contract - 用作实现的以太坊合约工厂。

* opts - 一个包含选项的对象：
    * getTxResponse：如果设置为true，则导致此函数返回与部署新实现合约相对应的以太坊交易响应，而不是其地址。请注意，如果新实现合约最初是作为forceImport的结果导入的，只会返回地址。
    * 其他选项如*Common Options*中所述。

**返回：**

*  实现合约的地址或与部署实现合约相对应的以太坊交易响应。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.20.0

## validateUpgrade
```
async function validateUpgrade(
  referenceAddressOrContract: string | ethers.ContractFactory,
  newContract: ethers.ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    kind?: 'uups' | 'transparent' | 'beacon',
  },
): Promise<void>
```
验证一个新的实现合约，而不部署它，也不实际升级到它。将当前的实现合约与新的实现合约进行比较，以检查存储布局兼容性错误。如果referenceAddressOrContract是当前实现地址，则需要kind选项。

**参数：**

* referenceAddressOrContract - 使用当前实现的代理或信标地址，或与当前实现相对应的地址或以太合约工厂。

* newContract - 新的实现合约。

* opts - 一个带有选项的对象：
    * 如常见选项中所述的其他选项。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.20.0

**示例：**

验证将现有代理升级到新合约（将PROXY_ADDRESS替换为您的代理地址）：
```
const { ethers, upgrades } = require('hardhat');

const BoxV2 = await ethers.getContractFactory('BoxV2');
await upgrades.validateUpgrade(PROXY_ADDRESS, BoxV2);
```

验证两个契约实现之间的升级
```
const { ethers, upgrades } = require('hardhat');

const Box = await ethers.getContractFactory('Box');
const BoxV2 = await ethers.getContractFactory('BoxV2');
await upgrades.validateUpgrade(Box, BoxV2);
```

## prepareUpgrade
```
async function prepareUpgrade(
  referenceAddressOrContract: string | ethers.Contract,
  Contract: ethers.ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    getTxResponse?: boolean,
    kind?: 'uups' | 'transparent' | 'beacon',
    usePlatformDeploy?: boolean,
  },
): Promise<string | ethers.providers.TransactionResponse>
```
验证和部署一个新的实现合约，并返回其地址。如果referenceAddressOrContract是当前实现地址，则需要使用kind选项。使用此方法可以准备从您无法直接控制或无法在Hardhat中使用的管理员地址运行的升级。

**参数：**

* referenceAddressOrContract - 代理或信标或实现地址或合约实例。

* Contract - 新的实现合约。

* opts - 一个带有选项的对象：

    * getTxResponse：如果设置为true，则导致此函数返回与部署新实现合约相对应的ethers事务响应，而不是其地址。请注意，如果新的实现合约最初是由于forceImport而导入的，只会返回地址。

    * 其他选项如Common Options中所述。

**返回：**

* 地址或与部署新实现合约相对应的ethers事务响应。

## platform.deployContract
```
async function deployContract(
  Contract: ethers.ContractFactory,
  args: unknown[] = [],
  opts?: {
    unsafeAllowDeployContract?: boolean,
    pollingInterval?: number,
  },
): Promise<ethers.Contract>
```
**注意：**OpenZeppelin Platform处于测试阶段，该函数可能会发生变化。

使用OpenZeppelin Platform部署一个不可升级的合约，并返回一个合约实例。如果合约看起来像一个实现合约，则会抛出错误。

> CAUTION
请不要使用此函数部署可升级合约的实现，因为此函数不执行升级安全性验证。对于实现合约，请使用*deployImplementation*。

**参数：**

* Contract - 用作要部署的合约的ethers合约工厂。

* opts - 一个带有选项的对象：

    * unsafeAllowDeployContract：如果设置为true，即使合约看起来像实现合约，也允许部署合约。默认为false。

    * pollingInterval：在调用结果合约实例的.deployed()时，检查事务确认之间的轮询间隔（以毫秒为单位）。默认为5000。

**返回：**

* 合约实例。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.25.0

## platform.proposeUpgrade
```
async function proposeUpgrade(
  proxyAddress: string,
  ImplFactory: ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    kind?: 'uups' | 'transparent' | 'beacon',
    usePlatformDeploy?: boolean,
    approvalProcessId?: string,
  },
): Promise<{
    proposalId: string,
    url: string,
    txResponse?: ethers.providers.TransactionResponse,
  }>
```
**注意：**OpenZeppelin Platform处于测试阶段，该函数可能会发生变化。

类似于prepareUpgrade。该方法验证并部署新的实现合约，还通过OpenZeppelin Platform创建升级提案，供升级管理员审查和批准。支持UUPS或透明代理。目前不支持信标代理或信标。对于信标，请使用prepareUpgrade以及在Platform上的事务提案来升级信标到部署的实现。

**参数：**

* proxyAddress - 代理地址。

* ImplFactory - 新的实现合约。

* opts - 一个带有选项的对象：

    * approvalProcessId：升级批准流程的ID。默认为在Platform上为您的部署环境配置的升级批准流程。

    * 其他如*Common Options*中所述的选项。

**返回：**

* 一个带有Platform提案ID、适用的Safe App提案的URL以及对应于部署新实现合约的ethers事务响应的对象。请注意，如果新的实现合约最初是通过forceImport导入的，ethers事务响应将为undefined。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.26.0

## platform.getDefaultApprovalProcess

```
async function getDefaultApprovalProcess(
): Promise<{
    approvalProcessId: string,
    address?: string,
  }>
```
**注意：**OpenZeppelin Platform处于beta版本，并且此函数可能会发生变化。

获取在Platform上配置的部署环境的默认升级批准流程。例如，这对于确定您可以在脚本中使用的默认多签钱包非常有用，以将其分配为代理的所有者。

**返回：**

* 一个包含默认升级批准流程ID和相关地址（如多签或治理合约地址）的对象。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.26.0

## defender.proposeUpgrade
```
async function proposeUpgrade(
  proxyAddress: string,
  ImplFactory: ContractFactory,
  opts?: {
    unsafeAllow?: ValidationError[],
    unsafeAllowRenames?: boolean,
    unsafeSkipStorageCheck?: boolean,
    constructorArgs?: unknown[],
    timeout?: number,
    pollingInterval?: number,
    redeployImplementation?: 'always' | 'never' | 'onchange',
    kind?: 'uups' | 'transparent' | 'beacon',
    title?: string,
    description?: string,
    multisig?: string,
    proxyAdmin?: string,
  },
): Promise<{
    url: string,
    txResponse?: ethers.providers.TransactionResponse,
  }>
```

> NOTE
该方法需要[@openzeppelin/hardhat-defender](https://www.npmjs.com/package/@openzeppelin/hardhat-defender)包，并配置Defender Team API密钥。

与prepareUpgrade类似。此方法验证并部署新的实现合约，还在*Defender Admin*中创建了一个升级提案，供升级管理员审查和批准。支持UUPS或Transparent代理。目前不支持beacon代理或beacons。对于beacons，请使用prepareUpgrade以及Defender Admin中的自定义操作来将beacon升级到部署的实现。

**参数：**

* proxyAddress - 代理地址。

* ImplFactory - 新的实现合约。

* opts - 一个带有选项的对象：

    * title：升级提案的标题，在Defender Admin中可见，默认为Upgrade to 0x12345678（使用新实现地址的前8位数字）

    * description：升级提案的描述，在Defender Admin中可见，默认为完整的实现地址。

    * multisig：具有执行升级权限的多签名钱包合约的地址。这在*Transparent代理*中是自动检测的，但在*UUPS代理*中是必需的（在*此处*阅读更多）。支持Gnosis Safe和Gnosis MultisigWallet多签名钱包。

    * proxyAdmin：管理代理的*ProxyAdmin*合约的地址（如果存在）。这在*Transparent代理*中是自动检测的，但在*UUPS代理*中是必需的（在*此处*阅读更多），尽管通常不需要使用ProxyAdmin进行UUPS代理。

    * 其他选项如*常见选项*中所述。

**返回：**

* 一个对象，其中包含Defender提案的URL，以及与部署新实现合约对应的ethers事务响应。请注意，如果新的实现合约最初是通过forceImport导入的，则ethers事务响应将为undefined。

## deployProxyAdmin
```
async function deployProxyAdmin(
  signer?: ethers.Signer,
  opts?: {
    timeout?: number,
    pollingInterval?: number,
  },
): Promise<string>
```
部署一个*代理管理员*合约，并返回其地址，如果当前网络上尚未部署，则返回代理管理员的地址。请注意，该插件目前仅支持每个网络使用一个代理管理员。

**参数：**

* signer - 用于部署的签名者。

* opts - 一个带有选项的对象：
    * 其他选项如Common Options中所述。

**返回：**

* 代理管理员的地址。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.20.0

## admin.changeProxyAdmin
```
async function changeProxyAdmin(
  proxyAddress: string,
  newAdmin: string,
  signer?: ethers.Signer,
): Promise<void>
```
更改特定代理的管理员。

**参数：**

* proxyAddress - 要更改的代理的地址。

* newAdmin - 新的管理员地址。

* signer - 用于交易的签名者。

## admin.transferProxyAdminOwnership
```
async function transferProxyAdminOwnership(
  newAdmin: string,
  signer?: ethers.Signer,
): Promise<void>
```
更改代理管理员合约的所有者，该合约是所有代理的升级权限的默认管理员。

**参数：**

* newAdmin - 新的管理员地址。

* signer - 用于交易的签名者。

## erc1967
```
async function erc1967.getImplementationAddress(proxyAddress: string): Promise<string>;
async function erc1967.getBeaconAddress(proxyAddress: string): Promise<string>;
async function erc1967.getAdminAddress(proxyAddress: string): Promise<string>;
```
该模块中的函数提供了对代理合约的[ERC1967](https://eips.ethereum.org/EIPS/eip-1967)变量的访问。

**参数：**

* proxyAddress - 代理地址。

**返回值：**

* 根据调用的函数，返回实现、信标或管理员地址。

## beacon
```
async function beacon.getImplementationAddress(beaconAddress: string): Promise<string>;
```
这个模块提供了一个方便的函数，用于从一个beacon合约中获取实现地址。

**参数：**

* beaconAddress - beacon地址。

**返回值：**

* 实现地址。

**自版本：**

* @openzeppelin/hardhat-upgrades@1.13.0

## silenceWarnings
```
function silenceWarnings()
```
> NOTE
这个函数在测试中很有用，但在生产部署脚本中使用它是不鼓励的。

禁止所有后续关于使用不安全标志的警告。在执行之前打印最后一个警告。

## verify
扩展[hardhat-etherscan](https://hardhat.org/hardhat-runner/plugins/nomiclabs-hardhat-etherscan)的验证任务，以完全验证Etherscan上的代理。这支持验证由Hardhat Upgrades或Truffle Upgrades插件部署的代理合约。

参数与hardhat-etherscan的验证任务相同。如果提供的地址是一个代理，此任务将验证代理的实现合约、代理本身和任何与代理相关的合约，并将代理链接到Etherscan上的实现合约ABI。如果提供的地址不是代理，则会在该地址上运行hardhat-etherscan的常规验证任务。

当您在代理地址上运行此任务时，将验证以下合约：

* 您的实现合约

* *ERC1967Proxy*或*TransparentUpgradeableProxy*或*BeaconProxy*（用于UUPS、透明或信标代理）

* *ProxyAdmin*（透明代理）

* *UpgradeableBeacon*（信标代理）

**自版本：**

* @openzeppelin/hardhat-upgrades@1.18.0

**用法：**

要使用此任务，请确保已安装hardhat-etherscan：
```
npm install --save-dev @nomiclabs/hardhat-etherscan
```
然后在您的Hardhat配置中导入@nomiclabs/hardhat-etherscan插件以及@openzeppelin/hardhat-upgrades插件。例如，如果您正在使用JavaScript，请在hardhat.config.js中导入这些插件：
```
require("@nomiclabs/hardhat-etherscan");
require("@openzeppelin/hardhat-upgrades");
```

或者如果你使用TypeScript，在hardhat.config.ts中导入插件：
```
import "@nomiclabs/hardhat-etherscan";
import "@openzeppelin/hardhat-upgrades";
```
最后，按照[hardhat-etherscan的使用文档](https://hardhat.org/hardhat-runner/plugins/nomiclabs-hardhat-etherscan#usage)配置您的Etherscan API密钥，并在命令行中使用代理地址运行verify任务。
```
npx hardhat verify --network mainnet PROXY_ADDRESS
```
或者可以通过编程方式使用*verify:verify子任务*：

await hre.run("verify:verify", {
  address: PROXY_ADDRESS,
});
请注意，如果您的实现合约只使用初始化函数，那么在验证时不需要包含构造函数参数。然而，如果您的实现合约具有带有参数的实际构造函数（例如用于设置不可变变量），则根据[任务](https://hardhat.org/hardhat-runner/plugins/nomiclabs-hardhat-etherscan#usage)或[子任务](https://hardhat.org/hardhat-runner/plugins/nomiclabs-hardhat-etherscan#using-programmatically)的使用信息包含构造函数参数。