# Sentinel API Referencev
Sentinel API允许您以编程方式创建和管理Sentinels。

请求需要使用从Team API Key协商的代币进行身份验证，该代币具有相应的功能。有关如何协商它的信息，请参阅[身份验证](./Authentication.md)部分。

> NOTE
我们建议您使用[defender-sentinel-client](https://www.npmjs.com/package/defender-sentinel-client) npm包简化与Sentinel API的交互。

> NOTE
不建议在浏览器环境中使用[defender-sentinel-client](https://www.npmjs.com/package/defender-sentinel-client) npm包，因为敏感密钥将公开暴露。
```
const { SentinelClient } = require('defender-sentinel-client');
const creds = { apiKey: ADMIN_API_KEY, apiSecret: ADMIN_API_SECRET };
const client = new SentinelClient(creds);
```

## 通知端点
Sentinels需要一个通知配置来在事件触发时通知正确的渠道。为此，您可以使用现有的通知ID（例如来自另一个Sentinels），或创建一个新的通知ID。

以下通知渠道可用：

* email
* Slack
* Discord
* telegram
* Datadog

### 列出通知
要列出所有通知渠道，您可以在客户端上调用listNotificationChannels函数，该函数返回一个NotificationResponse []对象。
```
// List existing notification channels
// This returns a `NotificationResponse[]` object.
const notificationChannels = await client.listNotificationChannels()
const { notificationId, type } = notificationChannels[0];
```

作为替代方案，可以通过curl请求实现。
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/sentinel/notifications"
```

一个响应示例
```
[
    {
        "notificationId": "14e99e8f-e2be-4eb1-a543-b2d90b68cdd8",
        "config": {
            "emails": [
                "john@example.com"
            ]
        },
        "updatedAt": "2021-11-04T12:21:22.858Z",
        "tenantId": "0c1585a9-1b86-4f9d-8468-28227802ac1d",
        "paused": false,
        "name": "MyEmailNotification",
        "type": "email"
    },
]
```

### 创建通知
要创建新的通知频道，您可以在客户端上调用createNotificationChannel函数。

createNotificationChannel函数需要CreateNotificationRequest对象，并返回一个NotificationResponse对象。
```
// create a new notification
const notification = await client.createNotificationChannel({
  type: 'email',
  name: 'MyEmailNotification',
  config: {
    emails: ['john@example.com'],
  },
  paused: false,
});

const notification = await client.createNotificationChannel({
  type: 'slack',
  name: 'MySlackNotification',
  config: {
    url: 'https://slack.com/url/key',
  },
  paused: false,
});

const notification = await client.createNotificationChannel({
  type: 'telegram',
  name: 'MyTelegramNotification',
  config: {
    botToken: 'abcd',
    chatId: '123',
  },
  paused: false,
});

const notification = await client.createNotificationChannel({
  type: 'discord',
  name: 'MyDiscordNotification',
  config: {
    url: 'https://discord.com/url/key',
  },
  paused: false,
});

const notification = await client.createNotificationChannel({
  type: 'datadog',
  name: 'MyDatadogNotification',
  config: {
    apiKey: 'abcd',
    metricPrefix: 'prefix',
  },
  paused: false,
});
```

### 更新通知
要更新现有的通知通道，您可以在客户端上调用updateNotificationChannel函数，该函数返回一个NotificationResponse对象。该函数将UpdateNotificationRequest对象作为参数传入。
```
const notificationChannel = await client.updateNotificationChannel(
{
  type: 'email',
  notificationId: '14e99e8f-e2be-4eb1-a543-b2d90b68cdd8',
  config: {
    emails: ["john@example.com"]
  },
  paused: false,
  name: "MyUpdatedEmailNotification"
});
```

### 收到通知
要检索通知频道，您可以在客户端上调用getNotificationChannel函数，该函数返回一个NotificationResponse对象。该函数以GetNotificationRequest对象作为参数。
```
const notificationToRetrieve = {type: 'email', notificationId: '14e99e8f-e2be-4eb1-a543-b2d90b68cdd8'}
const notificationChannel = await client.getNotificationChannel(notificationToRetrieve);
```

### 删除通知
要删除通知频道，您可以在客户端上调用deleteNotificationChannel函数，如果成功则返回一个字符串。该函数将DeleteNotificationRequest对象作为参数。
```
const notificationToDelete = {type: 'email', notificationId: '14e99e8f-e2be-4eb1-a543-b2d90b68cdd8'}
const deleted = await client.deleteNotificationChannel(notificationToDelete);
```

## Sentinels终端

### 列出Sentinels
要列出现有的Sentinels，您可以在客户端上调用list函数，该函数返回一个ListSentinelResponse对象：
```
await client.list();
```
订阅者端点用于通过GET请求检索现有Sentinels的列表。
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/sentinel/subscribers"
```

一个响应示例
```
[
    {
        "notifyConfig": {
            "notifications": [
                {
                    "type": "email",
                    "notificationId": "68e494d7-3b5a-4ffe-bd12-d4e483aa4995"
                }
            ],
            "timeoutMs": 0
        },
        "tenantId": "0c1585a9-1b86-4f9d-8468-28227802ac1d",
        "createdAt": "2021-11-15T16:04:13.936Z",
        "addressRules": [
            {
                "conditions": [],
                "abi": "[...]",
                "addresses": ["0xf664FA8aB9AA8021E2c08F45fEeA817D5730A713"]
            }
        ],
        "blockWatcherId": "rinkeby-1",
        "subscriberId": "abebeda6-f670-4e3c-a65b-a34c840e9a5e",
        "paused": false,
        "name": "test",
        "network": "rinkeby"
    }
]
```

### 创建Sentinels
要创建一个新的Sentinels，您需要提供网络、名称、暂停状态、条件、警报阈值和通知配置。该请求被导出为CreateSentinelRequest类型。
```
type CreateSentinelRequest =
  | ExternalCreateBlockSubscriberRequest
  | ExternalCreateFortaSubscriberRequest;

interface ExternalCreateBlockSubscriberRequest {
  type: 'BLOCK';
  name: string;
  addresses: string[];
  paused?: boolean;
  alertThreshold?: Threshold;
  autotaskCondition?: string;
  autotaskTrigger?: string;
  alertTimeoutMs?: number;
  alertMessageBody?: string;
  notificationChannels: string[];
  network: string;
  confirmLevel?: number; // blockWatcherId
  abi?: string;
  eventConditions?: EventCondition[];
  functionConditions?: FunctionCondition[];
  txCondition?: string;
}

interface ExternalCreateFortaSubscriberRequest {
  type: 'FORTA';
  name: string;
  paused?: boolean;
  alertThreshold?: Threshold;
  autotaskCondition?: string;
  autotaskTrigger?: string;
  alertTimeoutMs?: number;
  alertMessageBody?: string;
  notificationChannels: string[];
  network?: string;
  fortaLastProcessedTime?: string;
  addresses?: Address[];
  agentIDs?: string[];
  fortaConditions: FortaConditionSet;
  privateFortaNodeId?: string;
}
```

以下是一个合约（BLOCK）Sentinels的示例。此Sentinels将被命名为“我的新Sentinels”，并将监视Rinkeby网络上0x0f06aB75c7DD497981b75CD82F6566e3a5CAd8f2合约中的renounceOwnership函数。警报阈值设置为1小时内2次，并通过电子邮件通知用户。
```
const requestParameters = {
  type: 'BLOCK',
  network: 'rinkeby',
  // optional
  confirmLevel: 1, // if not set, we pick the blockwatcher for the chosen network with the lowest offset
  name: 'My New Sentinel',
  addresses: ['0x0f06aB75c7DD497981b75CD82F6566e3a5CAd8f2'],
  abi: '[{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{...}]',
  // optional
  paused: false,
  // optional
  eventConditions: [],
  // optional
  functionConditions: [{ functionSignature: 'renounceOwnership()' }],
  // optional
  txCondition: 'gasPrice > 0',
  // optional
  autotaskCondition: '3dcfee82-f5bd-43e3-8480-0676e5c28964',
  // optional
  autotaskTrigger: undefined,
  // optional
  alertThreshold: {
    amount: 2,
    windowSeconds: 3600,
  },
  // optional
  alertTimeoutMs: 0,
  notificationChannels: [notification.notificationId],
};
```

如果您希望基于其他事件触发Sentinels，可以添加另一个EventCondition或FunctionCondition对象，例如：
```
functionConditions: [{ functionSignature: 'renounceOwnership()' }],
eventConditions: [
  {
    eventSignature: "OwnershipTransferred(address,address)",
    expression: "\"0xf5453Ac1b5A978024F0469ea36Be25887EA812b5,0x6B9501462d48F7e78Ba11c98508ee16d29a03412\""
  }
]
```

您还可以通过修改txCondition属性应用交易条件：可能的变量包括：value、gasPrice、maxFeePerGas、maxPriorityFeePerGas、gasLimit、gasUsed、to、from、nonce、status（'success'、'failed'或'any'）、input或transactionIndex。
```
txCondition: 'gasPrice > 0',
```

您也可以按照以下方式构建一个请求Forta（FORTA）Sentinels。
```
const requestParameters = {
  type: 'FORTA',
  name: 'MyNewFortaSentinel',
  // optional
  addresses: ['0x0f06aB75c7DD497981b75CD82F6566e3a5CAd8f2'],
  // optional
  agentIDs: ['0x8fe07f1a4d33b30be2387293f052c273660c829e9a6965cf7e8d485bcb871083'],
  fortaConditions: {
    // optional
    alertIDs: undefined, // string[]
    minimumScannerCount: 1, // default is 1
    // optional
    severity: 2, // (unknown=0, info=1, low=2, medium=3, high=4, critical=5)
  },
  // optional
  paused: false,
  // optional
  autotaskCondition: '3dcfee82-f5bd-43e3-8480-0676e5c28964',
  // optional
  autotaskTrigger: undefined,
  // optional
  alertThreshold: {
    amount: 2,
    windowSeconds: 3600,
  },
  // optional
  alertTimeoutMs: 0,
  notificationChannels: [notification.notificationId],
};
```

要创建Forta本地模式Sentinel，请使用privateFortaNodeId指定扫描仪节点地址。
```
requestParameters.privateFortaNodeId: '0x0f06aB75c7DD497981b75CD82F6566e3a5CAd8f2'
```

一旦填充了所有必需的参数，您可以通过客户端调用 create 函数来创建一个 Sentinel。这将返回一个 CreateSentinelResponse 对象。
```
await client.create(requestParameters);
```

此外，Sentinels还可以调用自动任务进行进一步评估。有关此内容的文档可以在此处找到：https://docs.openzeppelin.com/defender/sentinel#autotask_conditions。
```
// 如果其他条件匹配，sentinel将调用此autotask进行进一步评估。
autotaskCondition: '3dcfee82-f5bd-43e3-8480-0676e5c28964',
// 在通知配置中定义autotask
autotaskTrigger: '1abfee11-a5bc-51e5-1180-0675a5b24c61',
```

通过POST请求使用subscribers端点可以创建新的sentinel。如果您希望直接调用API，则需要构建CreateBlockSubscriberRequest对象。

> CANTION
Defender目前仅支持Sentinels的有限子集（仅支持单个addressRule），我们强烈建议通过JS客户端进行操作，以避免不兼容性。

```
interface CreateBlockSubscriberRequest {
  name: string;
  paused: boolean;
  alertThreshold?: {
    amount: number;
    windowSeconds: number;
  };
  notifyConfig?: {
    notifications: [{
      notificationId: string;
      type: NotificationType;
    }];
    autotaskId?: string;
    messageBody?: string;
    timeoutMs: number;
  };
  addressRules: [{
    conditions: ConditionSet[];
    autotaskCondition?: {
      autotaskId: string;
    };
    addresses: string[];
    abi?: string;
  }];
  blockWatcherId: string;
  network: Network;
  type: 'BLOCK';
}

type NotificationType = 'slack' | 'email' | 'discord' | 'telegram' | 'datadog';

interface ConditionSet {
  eventConditions: EventCondition[];
  txConditions: TxCondition[];
  functionConditions: FunctionCondition[];
}
interface EventCondition {
  eventSignature: string;
  expression?: string | null;
}
interface TxCondition {
  status: 'success' | 'failed' | 'any';
  expression?: string | null;
}
interface FunctionCondition {
  functionSignature: string;
  expression?: string | null;
}
```

```
curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{...}' \
    "https://defender-api.openzeppelin.com/sentinel/subscribers"
```

支持的网络包括
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
```

### 获取Sentinels
您可以通过ID检索Sentinel。这将返回一个CreateSentinelResponse对象。
```
await client.get('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
```

订阅者/{id}端点用于通过GET请求检索Sentinels。
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/sentinel/subscribers/{id}"
```

### 更新Sentinels
要更新Sentinels，您可以在客户端上调用update函数。这将需要SentinelsID和UpdateSentinelRequest对象作为参数：
```
await client.update('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1', {name: 'My Updated Name', paused: true});
```

使用subscribers/{id}端点通过PUT请求来更新现有的Sentinels。

如果您希望直接调用API，则需要构造一个CreateBlockSubscriberRequest对象。
```
curl \
  -X PUT \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{...}' \
    "https://defender-api.openzeppelin.com/sentinel/subscribers/{id}"
```

### 删除Sentinels
您可以通过ID删除一个Sentinels。这将返回一个DeletedSentinelResponse对象。
```
await client.delete('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
```

使用subscribers/{id}端点来通过DELETE请求删除一个Sentinel。
```
curl \
  -X DELETE \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/sentinel/subscribers/{id}"
```

一个响应示例
```
{
    "message": "subscriber deleted"
}
```

### 暂停或恢复Sentinel。
您可以通过ID暂停和取消暂停一个Sentinel。这将返回一个CreateSentinelResponse对象。
```
await client.pause('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
await client.unpause('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
```
如果您希望直接调用API，可以使用更新端点，并根据需要设置pause为true或false。

### 列出网络
要列出启用了租户的网络，您可以在客户端上调用listNetworks函数，它返回一个Network[]对象：
```

await client.listNetworks(); // 列出所有网络
await client.listNetworks({ networkType: 'production' }); // 仅列出生产网络
await client.listNetworks({ networkType: 'test' }); // 仅列出测试网络
```

```
curl
-X GET
-H 'Accept: application/json'
-H 'Content-Type: application/json'
-H "X-Api-Key: $KEY"
-H "Authorization: Bearer $TOKEN"
"https://defender-api.openzeppelin.com/sentinel/networks"
```

您可以通过传递type查询参数来查询特定类型的网络（生产或测试）：
```
curl
-X GET
-H 'Accept: application/json'
-H 'Content-Type: application/json'
-H "X-Api-Key: $KEY"
-H "Authorization: Bearer $TOKEN"
"https://defender-api.openzeppelin.com/sentinel/networks?type=production"
```