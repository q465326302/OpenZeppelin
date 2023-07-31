# Admin API Reference
Admin API允许您以编程方式创建新的Admin提案。

请求需要使用从具有相应功能的Team API Key协商的承载令牌进行身份验证。有关如何协商它的信息，请参阅*身份验证*部分。

> NOTE
我们建议您使用[defender-admin-client](https://www.npmjs.com/package/defender-admin-client) npm包来简化与Admin API的交互。

> NOTE
不建议在浏览器环境中使用[defender-admin-client](https://www.npmjs.com/package/defender-admin-client) npm包，因为敏感密钥将被公开显示。

## 提案端点
通过POST请求使用/proposals端点创建新的Admin操作提案。通过此方式创建的任何操作最初都没有批准。如果提案的接收方合约不存在，则将使用提供的参数创建它。
```
type Network =
  | 'mainnet'
  | 'goerli'
  | 'sepolia'
  | 'xdai'
  | 'sokol'
  | 'fuse'
  | 'bsc'
  | 'bsctest'
  | 'fantom'
  | 'fantomtest'
  | 'moonbase'
  | 'moonriver'
  | 'moonbeam'
  | 'matic'
  | 'mumbai'
  | 'avalanche'
  | 'fuji'
  | 'optimism'
  | 'optimism-kovan'
  | 'optimism-goerli'
  | 'arbitrum'
  | 'arbitrum-nova'
  | 'arbitrum-rinkeby'
  | 'arbitrum-goerli'
  | 'celo'
  | 'alfajores'
  | 'harmony-s0'
  | 'harmony-test-s0'
  | 'aurora'
  | 'auroratest'
  | 'zksync'
  | 'zksync-goerli'
  | 'base-goerli'
  | 'linea-goerli';

type Address = string;
type ProposalType = ProposalStepType | ProposalBatchType;
type ProposalStepType = 'upgrade' | 'custom' | 'pause' | 'send-funds' | 'access-control';
type ProposalBatchType = 'batch';
type Hex = string;
type BigUInt = string | number;
type ProposalFunctionInputsArray = (string | boolean)[];
type ProposalFunctionInputs = (
  | string
  | boolean
  | (string | boolean | (string | boolean | (string | boolean | ProposalFunctionInputsArray)[])[])[]
)[];

interface CreateProposalRequest {
  contract: PartialContract | PartialContract[];
  title: string;
  description: string;
  type: ProposalType;
  via?: Address;
  viaType?: 'EOA' | 'Gnosis Safe' | 'Gnosis Multisig';
  functionInterface?: ProposalTargetFunction;
  functionInputs?: ProposalFunctionInputs;
  metadata?: ProposalMetadata;
  steps?: ProposalStep[];
}

interface PartialContract {
  network: Network;
  address: Address;
  name?: string;
  abi?: string;
}

interface ProposalMetadata {
  newImplementationAddress?: Address;
  newImplementationAbi?: string;
  proxyAdminAddress?: Address;
  action?: 'pause' | 'unpause' | 'grantRole' | 'revokeRole';
  operationType?: 'call' | 'delegateCall';
  account?: Address;
  role?: Hex;
  sendTo?: Address;
  sendValue?: BigUInt;
  sendCurrency?: Token | NativeCurrency;
}

interface Token {
  name: string;
  symbol: string;
  address: Address;
  network: Network;
  decimals: number;
  type: 'ERC20';
}
interface NativeCurrency {
  name: string;
  symbol: string;
  decimals: number;
  type: 'native';
}

interface ProposalStep {
  contractId: string;
  targetFunction?: ProposalTargetFunction;
  functionInputs?: ProposalFunctionInputs;
  metadata?: ProposalMetadata;
  type: ProposalStepType;
}

interface ProposalTargetFunction {
  name?: string;
  inputs?: ProposalFunctionInputType[];
}

interface ProposalFunctionInputType {
  name?: string;
  type: string;
  internalType?: string;
  components?: ProposalFunctionInputType[];
}
```
请注意，自定义提案需要提供地址（通过该地址发送多签请求）、viaType（多签类型）、functionInterface（要调用的函数的ABI定义）和functionInputs等字段。另一方面，升级类型的提案只需要提供metadata.newImplementationAddress字段，因为Defender将自动为您计算其余字段。

以下是一个升级请求的示例：
```
DATA='{
	"contract": {
		"address": "0x179810822f56b0e79469189741a3fa5f2f9a7631",
		"network": "rinkeby"
	},
	"title": "Upgrade to v2",
	"description": "Upgrading contract to version 2.0",
	"type": "upgrade",
	"metadata": {
		"newImplementation": "0x3E5e9111Ae8eB78Fe1CC3bb8915d5D461F3Ef9A9",
		"newImplementationAbi": "[{\"inputs \": [],\"name \": \"greet \",\"outputs \": [{\"internalType \": \"string \",\"name \": \"\",\"type \": \"string \"}],\"stateMutability \": \"pure \",\"type \": \"function \"}]"
	}
}'

curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://defender-api.openzeppelin.com/admin/proposals"
```

> WARNING
Defender API仅验证函数输入是否符合签名，但不验证提案是否实际可执行。这意味着您可以创建调用合约上不存在的函数或尝试升级无法升级的合约的提案。但是，您将无法之后批准它们。

### 存档提案
可以通过API调用通过在PUT调用有效负载中传递布尔归档值来存档（和取消存档）提案：
```
DATA='{
	"archived": true
}'

curl \
  -X PUT \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://defender-api.openzeppelin.com/admin/contracts/${CONTRACT_ID}/proposals/${PROPOSAL_ID}/archived"
```
可以使用defender-admin-client进行相同的调用：
```
await client.archiveProposal(contractId, proposalId);
await client.unarchiveProposal(contractId, proposalId);
```

### 检索提案
建议可以通过API检索：
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://defender-api.openzeppelin.com/admin/contracts/${CONTRACT_ID}/proposals/${PROPOSAL_ID}"
```
可以使用defender-admin-client通过传递合约和提案ID进行相同的调用：
```
await client.getProposal(contractId, proposalId);
```

### 模拟一个提案
提案可以通过API进行模拟。模拟的结果（SimulationResponse）将被存储，并可以通过getProposalSimulation端点进行检索。
```
const proposal = await client.getProposal(contractId, proposalId);

// import the ABI and create an ethers interface
const contractInterface = new ethers.utils.Interface(contractABI);

// encode function data
const data = contractInterface.encodeFunctionData(proposal.functionInterface.name, proposal.functionInputs);

const simulation = await client.simulateProposal(
  proposal.contractId, // contractId
  proposal.proposalId, // proposalId
  {
    transactionData: {
      // this is the default hardhat address
      from: '0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266', // change this to impersonate the `from` address
      data,
      to: proposal.contract.address,
      value: proposal.metadata.sendValue,
    },
    // default to latest finalized block,
    // can be up to 100 blocks ahead of current block,
    // does not support previous blocks
    blockNumber: undefined,
  },
);
```

您还可以选择在createProposal请求中设置simulate标志（只要这不是批量提案），以在同一请求中模拟提案。您可以通过设置overrideSimulationOpts属性来覆盖模拟参数，该属性是一个SimulationRequest对象。
```
const proposalWithSimulation = await client.createProposal({
  contract: {
    address: '0xA91382E82fB676d4c935E601305E5253b3829dCD',
    network: 'mainnet',
    // provide abi OR overrideSimulationOpts.transactionData.data
    abi: JSON.stringify(contractABI),
  },
  title: 'Flash',
  description: 'Call the Flash() function',
  type: 'custom',
  metadata: {
    sendTo: '0xA91382E82fB676d4c935E601305E5253b3829dCD',
    sendValue: '10000000000000000',
    sendCurrency: {
      name: 'Ethereum',
      symbol: 'ETH',
      decimals: 18,
      type: 'native',
    },
  },
  functionInterface: { name: 'flash', inputs: [] },
  functionInputs: [],
  via: '0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266',
  viaType: 'EOA',
  // set simulate to true
  simulate: true,
  // optional
  overrideSimulationOpts: {
    transactionData: {
      // or instead of ABI, you can provide data
      data: '0xd336c82d',
    },
  },
});
```

> NOTE
目前不支持在批量提案中启用simulate标志作为createProposal请求的一部分。

> NOTE
模拟可能因多种原因而失败，例如网络拥塞、不稳定的提供商或达到配额限制。我们建议您跟踪响应代码以确保成功返回响应。如果事务因原因字符串而撤消，则可以从响应对象下的response.meta.returnString获取。可以从response.meta.reverted中跟踪事务撤消。

### 检索模拟。
您也可以检索提案的现有模拟。
```
const proposal = await client.getProposal(contractId, proposalId);

const simulation = await client.getProposalSimulation(
  proposal.contractId, // contractId
  proposal.proposalId, // proposalId
);
```

## 合约终端点
/contracts端点可用于管理导入到Defender管理面板中的合约。通过向端点发出PUT请求，您可以创建或更新合约，给出其网络和地址：
```
DATA='{
  "address": "0x179810822f56b0e79469189741a3fa5f2f9a7631",
  "network": "rinkeby",
  "name": "My Contract",
  "abi": "..."
}'

curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://defender-api.openzeppelin.com/admin/contracts"
```
您还可以向相同的端点发出GET请求，以检索导入的所有合约列表。

## 验证端点
/verifications端点可用于验证Defender支持的任何网络中任何合约的部署字节码。通过向端点发出POST请求，您可以发出验证请求，其结果将存储在Defender的地址簿中。这反过来将使任何具有访问您的Defender工作区的权限的人都可以访问这些结果。
```
DATA='
{
  artifactUri: 'https://gist.githubusercontent.com/johndoe/506bff068172583d4d82ef53ba01e26c/raw/5f142de4c33aefddb2625b2cce1e6aba7791ebdc/compile-artifact.json',
  solidityFilePath: 'contracts/Vault.sol',
  contractName: 'VaultV2',
  contractAddress: '0x38e373CC414e90dDec45cf7166d497409902e998',
  contractNetwork: 'rinkeby',
  referenceUri: 'https://ci-run-or-git-commit.example/',
}'

curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://defender-api.openzeppelin.com/admin/verifications"
```
您还可以向 `/verifications/${contractNetwork}/${contractAddress}` 发出 GET 请求，以获取与 contractAddress 相关的最新验证信息。 对于上面的示例，这将是：
```
curl \
  -X GET \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/admin/verifications/rinkeby/0x38e373CC414e90dDec45cf7166d497409902e998"
```
有关Defender中字节码验证的更深入讨论，请参阅本文档的*Defender管理员验证部分*。