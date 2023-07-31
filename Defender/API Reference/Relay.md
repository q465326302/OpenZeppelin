# Relay API Reference

Relay API分为两个模块：

1. Relay Client
    * 在帐户中执行创建、读取和更新操作，跨所有中继器
    * 使用使用Team API Key/Secret生成的令牌进行身份验证（在创建Team API Key时可用）

2. Relay Signer

    * 使用特定中继器执行发送、签名和其他操作
    * 使用使用Relayer API Key/Secret生成的令牌进行身份验证（在创建中继器时可用）

有关身份验证的更多信息，请参阅*身份验证*部分。

> NOTE
您可以使用[defender-relay-client](https://www.npmjs.com/package/defender-relay-client) npm包简化与Relay API的交互。

> NOTE
不建议在浏览器环境中使用[defender-relay-client](https://www.npmjs.com/package/defender-relay-client) npm包，因为敏感密钥将被公开暴露。

## Relay客户端模块参考

Relay客户端模块公开以下终端：

* list：列出与帐户关联的所有中继器
* create：创建新的中继器
* retrieve：检索单个中继器的数据
* update：更新现有中继器

### 列出中继器

要列出现有的中继器，您可以在客户端上调用list函数，该函数返回ListRelayerResponse对象：
```
await client.list();
```
relayers/summary端点用于通过GET请求检索现有中继列表。
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/relayer/relayers/summary"
```
一个响应示例
```
[
    {
      relayerId: '201832e2-c612-4283-97c5-31849242c1fe',
      name: 'MyRelayer',
      address: '0x623e258760220127d4856fa24ff126a230560749',
      network: 'rinkeby',
      createdAt: '2022-04-05T16:57:37.154Z',
      paused: false,
      policies: {},
      minBalance: '100000000000000000',
      pendingTxCost: '0'
    }
]
```

### 列出中继器密钥

要列出与现有中继器关联的密钥，您可以在客户端上调用listKeys函数，并使用relayerId返回一个RelayerKey对象数组：
```
await client.listKeys('58b3d255-e357-4b0d-aa16-e86f745e63b9');
```
/relayers/${relayerId}/keys端点用于通过GET请求检索中继器密钥列表。
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/relayer/relayers/58b3d255-e357-4b0d-aa16-e86f745e63b9/keys"
```
一个响应示例
```
[
    {
      apiKey: '5Kk12pMA6LjPSppaRAsfAJRAjYeFJzr6',
      createdAt: '2022-04-26T19:49:10.292Z',
      relayerId: '58b3d255-e357-4b0d-aa16-e86f745e63b9',
      keyId: '898a952b-415b-4d4b-882b-3958f11pbeea'
  }
]
```

### 创建中继器
要创建新的中继器，您需要按照接口CreateRelayerRequest中直接描述的参数提供参数。
```
interface CreateRelayerRequest {
  name: string;
  useAddressFromRelayerId?: string;
  network: Network;
  minBalance: string;
  policies?: UpdateRelayerPoliciesRequest;
}

interface UpdateRelayerPoliciesRequest {
  gasPriceCap?: string;
  whitelistReceivers?: string[];
  EIP1559Pricing?: boolean;
  privateTransactions?: boolean;
}

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
```
使用defender-relay-client包：
```
const requestParameters = {
  name: 'MyNewRelayer',
  network: 'rinkeby',
  minBalance: BigInt(1e17).toString(),
  policies: {
    whitelistReceivers: ['0xAb5801a7D398351b8bE11C439e05C5B3259aeC9B'],
  },
};

await client.create(requestParameters);
```
您还可以通过在useAddressFromRelayerId参数中引用现有的relayerId来将现有的中继克隆到不同的网络中：
```
const requestParameters = {
  name: 'MyClonedRelayer',
  useAddressFromRelayerId: '201832e2-c612-4283-97c5-31849242c1fe',
  network: 'mainnet',
  minBalance: BigInt(1e17).toString()
};

await client.create(requestParameters);
```
向/relayers端点发出POST请求：
```
curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{...}' \
    "https://defender-api.openzeppelin.com/relayer/relayers"
```

#### 中继策略
以下中继策略可以选择性地包含在创建和更新调用中：

* gasPriceCap：以 wei 为单位的最大 gasPrice 或 maxFeePerGas
* whitelistReceivers：一个包含此中继器可以与之交互的所有有效地址的数组；如果未定义此策略，则中继器可以向任何地址发送交易。
* EIP1559Pricing：一个布尔值，指示默认定价方式。如果为 true，则交易将按照 EIP1559 定价，否则按照传统交易定价。
* privateTransactions：一个布尔值，指示默认的交易内存池可见性。如果为 true，则交易将通过私有内存池发送。

> NOTE
EIP1559Pricing 标志仅适用于 EIP1559 网络上的中继器。

> NOTE
仅通过使用 [Flashbots Protect RPC](https://docs.flashbots.net/flashbots-protect/rpc/quick-start) 在 goerli 和 mainnet 上启用[私有交易](https://docs.flashbots.net/flashbots-protect/rpc/quick-start#key-considerations)。因此，发送私有交易时可能需要考虑相同的密钥问题（例如，[叔叔强盗风险](https://docs.flashbots.net/flashbots-protect/rpc/uncle-bandits)）。

## 创建中继器密钥
通过 API 创建的中继器默认不会有任何关联的中继器密钥。要创建一个，可以在客户端上调用 createKey 方法，该方法返回一个 RelayerKey（仅在创建时可用的 secretKey）：
```
await client.createKey('58b3d255-e357-4b0d-aa16-e86f745e63b9');
```
向/relayers/${relayerId}/keys端点发出POST请求：
```
curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/relayer/relayers/58b3d255-e357-4b0d-aa16-e86f745e63b9/keys"
```
一个响应示例
```
[
    {
      apiKey: '5Kk12pMA6LjPSppaRAsfAJRAjYeFJzr6',
      createdAt: '2022-04-26T19:49:10.292Z',
      relayerId: '58b3d255-e357-4b0d-aa16-e86f745e63b9',
      keyId: '898a952b-415b-4d4b-882b-3958f11pbeea',
      secretKey: '1pDxUoWYtrAK4xMsKmOITpd1PQofWDFRRTNp2jshDhZ1XhYxDCfAvQ5vyKprFhXQ'
  }
]
```

### 更新中继器
要更新中继器，您可以在客户端上调用更新函数。这将需要中继器ID和UpdateRelayerRequest对象作为参数：
```
interface UpdateRelayerRequest {
  name?: string;
  minBalance?: string;
  policies?: UpdateRelayerPoliciesRequest;
}
```
使用defender-relay-client包：
```
await client.update('201832e2-c612-4283-97c5-31849242c1fe', { name: 'My Updated Name' });
```
向/relayers端点发出PUT请求：
```
curl \
  -X PUT \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{...}' \
    "https://defender-api.openzeppelin.com/relayer/relayers"
```

> NOTE
如果直接进行请求（而不是使用defender-relay-client），任何策略更新都需要通过以下端点直接进行请求，请求体类型为UpdateRelayerPoliciesRequest。

```
curl \
  -X PUT \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{...}' \
    "https://defender-api.openzeppelin.com/relayer/relayers/{relayerId}"
```

### 删除中继键
要删除与中继器关联的密钥，可以在客户端上调用deleteKey方法，该方法返回一个指示删除状态的消息：
```
await client.deleteKey('58b3d255-e357-4b0d-aa16-e86f745e63b9', 'j3bru93-k32l-3p1s-pp56-u43f675e92p1');
```

> NOTE
deleteKey的第二个参数是keyId，而不是apiKey。区分两者的一种方法是keyId包含连字符。可以通过上面描述的列表操作获取它，也可以在创建密钥的响应中获取它。

通过向/relayers/${relayerId}/keys/${keyId}端点发出DELETE请求：
```
curl \
  -X DELETE \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/relayer/relayers/58b3d255-e357-4b0d-aa16-e86f745e63b9/keys/j3bru93-k32l-3p1s-pp56-u43f675e92p1"
```
一个响应示例
```
{
  message: 'j3bru93-k32l-3p1s-pp56-u43f675e92p1 deleted'
}
```
删除中继（不仅仅是密钥）仅可通过Defender控制台进行。

## 中继签名模块参考资料
Relayer签名模块公开了以下端点：

* txs：将交易发送到以太坊区块链并查询其状态
* sign：使用中继私钥对任意数据进行签名
* relayer：从中继获取信息
* jsonrpc：对以太坊网络进行json-rpc调用

### 交易端点
/ txs端点用于发送交易和根据交易标识符检索交易信息。

#### 发送交易
要将交易发送到以太坊区块链，请提交带有所需有效负载的POST请求。有效负载格式如下：
```
type Address = string;
type BigUInt = string | number;
type Hex = string;
type Speed = 'safeLow' | 'average' | 'fast' | 'fastest';

interface SendBaseTransactionRequest {
  to?: Address;
  value?: BigUInt;
  data?: Hex;
  gasLimit: BigUInt;
  validUntil?: string;
  isPrivate?: boolean;
}

interface SendSpeedTransactionRequest extends SendBaseTransactionRequest {
  speed: Speed;
}

interface SendLegacyTransactionRequest extends SendBaseTransactionRequest {
  gasPrice: BigUInt;
}

interface SendEIP1559TransactionRequest extends SendBaseTransactionRequest {
  maxFeePerGas: BigUInt;
  maxPriorityFeePerGas: BigUInt;
}

type RelayerTransactionPayload =
  | SendBaseTransactionRequest
  | SendSpeedTransactionRequest
  | SendLegacyTransactionRequest
  | SendEIP1559TransactionRequest;
```
一个请求的例子
```
DATA='{ "to": "0x179810822f56b0e79469189741a3fa5f2f9a7631", "value": "1", "speed": "fast", "gasLimit": "21000" }'

curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://api.defender.openzeppelin.com/txs"
```
您将收到以下格式的回复
```
type Address = string;
type BigUInt = string | number;
type Hex = string;
type Speed = 'safeLow' | 'average' | 'fast' | 'fastest';
type Status = 'pending' | 'sent' | 'submitted' | 'inmempool' | 'mined' | 'confirmed' | 'failed';

interface RelayerTransactionBase {
  transactionId: string;
  hash: string;
  to: Address;
  from: Address;
  value?: string;
  data?: string;
  speed: Speed;
  gasLimit: number;
  nonce: number;
  status: Status;
  chainId: number;
  validUntil: string;
  createdAt: string;
  sentAt?: string;
  pricedAt?: string;
  isPrivate?: boolean;
}

interface RelayerLegacyTransaction extends RelayerTransactionBase {
  gasPrice: number;
}

interface RelayerEIP1559Transaction extends RelayerTransactionBase {
  maxPriorityFeePerGas: number;
  maxFeePerGas: number;
}

export type TransactionResponse = RelayerLegacyTransaction | RelayerEIP1559Transaction;
```
注意额外的transactionId字段，它是一个Defender内部的标识符，用于查询和替换。

#### 交易状态
每个交易响应都将分配一个状态，具体取决于后端最后一次看到它的时间。每次Defender监控交易时，状态可能会更新。

可用的状态及其对应的描述如下：

> WARNING
状态挂起，已发送和已提交可能在未来合并。避免依赖它们，因为向后兼容性不能保证。
```
type Status =
  /**
   * Temporary pre-sent status.
   * The transaction has been received but not yet signed.
   */
  | 'pending'
  /**
   * Sent to queue.
   * The transaction is already signed, enqueued and waiting to be broadcasted to the network.
   */
  | 'sent'
  /**
   * Transaction broadcasted.
   * The transaction has been broadcasted to the network providers.
   */
  | 'submitted'
  /**
   * Seen on mempool.
   * It's confirmed that the transaction is in the mempool of at least one of the nodes of the network.
   */
  | 'inmempool'
  /**
   * Transaction mined.
   * A mined block has included the transaction.
   */
  | 'mined'
  /**
   * Enough confirmations passed.
   * The transaction has being kept in a mined block after N new mined blocks, where N depends on the chain.
   */
  | 'confirmed'
  /**
   * Terminal failure.
   * The transaction couldn't get processed for unexpected reasons.
   */
  | 'failed';
```

#### 替换交易
要替换待处理的交易，请向txs端点发出PUT请求，使用Defender transactionId或nonce，而不是交易哈希。

请求的示例：
```
curl \
  -X PUT \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://api.defender.openzeppelin.com/txs/$IDorNONCE"
```
请求负载和响应与发送交易的相同。

#### 查询交易
要检索交易状态和数据，请使用 Defender transactionId 而不是交易哈希值向 txs 端点发出 GET 请求。

请求的示例：
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://api.defender.openzeppelin.com/txs/$ID"
```
一个响应示例
```
{
   "chainId":4,
   "hash":"0xcef95469a9f02757f0968ec8c11449ae5e7486073075381dcd62bacec9e5d627",
   "transactionId":"affba150-e563-441e-ae49-04bd6050979a",
   "value":"0x1",
   "maxFeePerGas":1000000000,
   "maxPriorityFeePerGas":150000000,
   "gasLimit":21000,
   "to":"0x179810822f56b0e79469189741a3fa5f2f9a7631",
   "from":"0xbce0b5b71668e42d908e387b68dba91789c932b8",
   "data":"0x",
   "nonce":160,
   "status":"mined",
   "speed":"fast"
   "validUntil": "2022-10-27T03:22:09.566Z",
   "sentAt": "2022-10-26T19:22:10.067Z",
   "createdAt": "2022-10-26T19:22:10.067Z",
   "isPrivate": false
}
```

#### 列出交易记录
要检索最近从您的中继发送的交易记录列表，请向txs发出GET请求。您可以选择设置since、limit和status（已挖掘、待处理或失败）作为查询参数。

请求示例：
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://api.defender.openzeppelin.com/txs?status=pending&limit=5"
```
一个响应示例
```
[{
   "chainId":4,
   "hash":"0xcef95469a9f02757f0968ec8c11449ae5e7486073075381dcd62bacec9e5d627",
   "transactionId":"affba150-e563-441e-ae49-04bd6050979a",
   "value":"0x1",
   "maxFeePerGas":1000000000,
   "maxPriorityFeePerGas":150000000,
   "gasLimit":21000,
   "to":"0x179810822f56b0e79469189741a3fa5f2f9a7631",
   "from":"0xbce0b5b71668e42d908e387b68dba91789c932b8",
   "data":"0x",
   "nonce":160,
   "status":"mined",
   "speed":"fast"
   "validUntil": "2022-10-27T03:22:09.566Z",
   "sentAt": "2022-10-26T19:22:10.067Z",
   "createdAt": "2022-10-26T19:22:10.067Z",
   "isPrivate": false
}]
```

### 签名终端
要根据[EIP-191标准](https://eips.ethereum.org/EIPS/eip-191)（以\x19Ethereum Signed Message：\n为前缀）使用您的Relay私钥签署任意消息，请向/sign发出POST请求，其中包含要签署的十六进制字符串的有效载荷。有效载荷格式如下：
```
interface SignMessagePayload {
  message: Hex;
}
```
一个响应示例
```
DATA='{ "message": "0x0123456789abcdef" }'

curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://api.defender.openzeppelin.com/sign"
```
您将收到以下格式的回复：
```
interface SignedMessagePayload {
  sig: Hex;
  r: Hex;
  s: Hex;
  v: number;
}
```
一个响应示例
```
{
   "r":"0x819b2645a0b73494724dac355e6ecfc983d94597b533d34fe3ecd0277046a1eb",
   "s":"0x3b73c695b47dd275d17246d86bbfe35f112a7bdb5bf4a5a1a8e22fe37dfd005a",
   "v":44,
   "sig":"0x819b2645a0b73494724dac355e6ecfc983d94597b533d34fe3ecd0277046a1eb3b73c695b47dd275d17246d86bbfe35f112a7bdb5bf4a5a1a8e22fe37dfd005a2c"
}
```

### 签名数据终端
要按照[EIP-712规范](https://eips.ethereum.org/EIPS/eip-712)使用Relay私钥签署输入的数据，需要向/sign发出POST请求，请求载荷包含域分隔符和哈希结构（message）的哈希值。它们都应该是32字节长，因为它们本身是哈希值。请求载荷格式如下：
```
interface SignTypedDataPayload {
  domainSeparator: Hex;
  hashStructMessage: Hex;
}
```
一个响应示例
```
DATA='{ "domainSeparator": "0x0123456789abcdef...", "hashStructMessage": "0x0123456789abcdef..." }'

curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://api.defender.openzeppelin.com/sign-typed-data"
```
您将收到以下格式的回复:
```
interface SignedMessagePayload {
  sig: Hex;
  r: Hex;
  s: Hex;
  v: number;
}
```
一个响应示例
```
{
   "r":"0x819b2645a0b73494724dac355e6ecfc983d94597b533d34fe3ecd0277046a1eb",
   "s":"0x3b73c695b47dd275d17246d86bbfe35f112a7bdb5bf4a5a1a8e22fe37dfd005a",
   "v":44,
   "sig":"0x819b2645a0b73494724dac355e6ecfc983d94597b533d34fe3ecd0277046a1eb3b73c695b47dd275d17246d86bbfe35f112a7bdb5bf4a5a1a8e22fe37dfd005a2c"
}
```

### Relayer Endpoint（中继端点）

要使用Relay API检索中继器的数据，请向/relayer端点发出GET请求。

请求示例：
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://api.defender.openzeppelin.com/relayer"
```
您将收到以下格式的回复:
```
interface RelayerModel {
  relayerId: string;
  name: string;
  address: string;
  network: string;
  paused: boolean;
  createdAt: string;
  pendingTxCost: string;
}
```
一个响应示例
```
{
   "relayerId":"d5484fb1-df83-4659-9903-16d57d41f188",
   "name":"Rinkeby",
   "address":"0x71764d6450c2b710fc3e4ee5b7a038d1e7e4fc29",
   "network":"rinkeby",
   "createdAt":"2020-11-02T18:00:00.212Z",
   "paused":false,
   "pendingTxCost":"0"
}
```

### JSON RPC 端点
要向您的中继网络发出JSON RPC调用，请向/relayer/jsonrpc端点发出POST请求，并提供方法名称和参数。请注意，不支持事件过滤方法和WebSocket订阅。

请求的示例：
```
DATA='{ "jsonrpc":"2.0","method":"eth_getBalance","params":["0x407d73d8a49eeb85d32cf465507dd71d507100c1","latest"],"id":1 }'

curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://api.defender.openzeppelin.com/relayer/jsonrpc"
```
一个响应示例
```
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0x0234c8a3397aab58"
}
```
