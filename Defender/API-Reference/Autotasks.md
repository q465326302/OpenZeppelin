# Autotask API Reference
Autotask API允许您以编程方式列出、创建、检索、更新、删除Autotasks，以及将新代码上传到任何Autotask中。

请求需要使用从Team API Key与相应能力协商的代币进行身份验证。有关如何协商它的信息，请参阅[身份验证](./Authentication.md)部分。

> NOTE
我们建议您使用[defender-autotask-client](https://www.npmjs.com/package/defender-autotask-client) npm包来简化与Autotask API的交互。

> NOTE
不建议在浏览器环境中使用[defender-autotask-client](https://www.npmjs.com/package/defender-autotask-client) npm包，因为密钥将公开暴露。

## 创建端点
通过POST请求使用autotasks端点来创建新的Autotask。该端点接受以下接口：
```
interface CreateAutotaskRequest {
  name: string;
  encodedZippedCode: string;
  Relayer Id?: string;
  trigger: {
    type: 'schedule' | 'webhook';
    frequencyMinutes?: number;
    cron?: string;
  };
  paused: boolean;
}
```

使用defender-autotask-client，您可以这样调用创建端点：
```
const myAutotask = {
  name: "my-autotask",
  encodedZippedCode: await client.getEncodedZippedCodeFromFolder('./code'),
  trigger: {
    type: 'schedule',
    frequencyMinutes: 1500,
  },
  paused: false
};
await client.create(myAutotask);
```

## 列表终端点
autotasks端点用于通过GET请求检索Autotask。

使用defender-autotask-client，您可以调用列表端点，如下所示：
```
await client.list();
```

一个响应示例
```
{
  items: [
    {
      autotaskId: '11ecbea0-7126-4345-a5a5-815898307c9d',
      name: 'Example 1',
      paused: false,
      trigger: [Object],
      Relayer Id: 'f75701bb-d0bd-49d2-bec9-3420a7b645f6'
    },
    {
      autotaskId: '143f7d62-22c1-427e-82e7-20036925c4b3',
      name: 'Example 2',
      paused: false,
      trigger: [Object]
    },
  ],
  keyValueStoreItemsCount: 0,
  runsQuotaUsage: 0
}
```

## 获取端点
使用GET请求，可以通过autotasks/{id}终端点来检索Autotask。该端点接受一个autotask Id作为参数。

使用defender-autotask-client，您可以像这样调用get端点：
```
await client.get("671d1f80-99e3-4829-aa15-f01e3298e428");
```

一个响应示例
```
{
  autotaskId: '671d1f80-99e3-4829-aa15-f01e3298e428',
  name: 'my-autotask',
  paused: false,
  trigger: { type: 'schedule', frequencyMinutes: 1500 },
  encodedZippedCode: 'UEsDBAoAAAAAAAFzWFTA91VHFwAAABcAAAAHAAAAZGVwcy5qc2V4cG9ydHMuZm9vID0gKCkgPT4gNDI7UEsDBAoAAAAAAAFzWFSEyiyCiQAAAIkAAAAIAAAAaW5kZXguanNjb25zdCBkZXBzID0gcmVxdWlyZSgnLi9kZXBzJyk7CgpleHBvcnRzLmhhbmRsZXIgPSBhc3luYyBmdW5jdGlvbigpIHsKICBjb25zdCB2YWx1ZSA9IGRlcHMuZm9vKCk7CiAgY29uc29sZS5sb2codmFsdWUpOwogIHJldHVybiB2YWx1ZTsKfVBLAQIUAAoAAAAAAAFzWFTA91VHFwAAABcAAAAHAAAAAAAAAAAAAAAAAAAAAABkZXBzLmpzUEsBAhQACgAAAAAAAXNYVITKLIKJAAAAiQAAAAgAAAAAAAAAAAAAAAAAPAAAAGluZGV4LmpzUEsFBgAAAAACAAIAawAAAOsAAAAAAA=='
}
```

## 更新终端点
Autotasks终端点用于通过PUT请求更新Autotask。该终端点接受以下接口：
```
interface UpdateAutotaskRequest {
  autotaskId: string;
  name: string;
  encodedZippedCode?: string;
  Relayer Id?: string;
  trigger: {
    type: 'schedule' | 'webhook';
    frequencyMinutes?: number;
    cron?: string;
  };
  paused: boolean;
}
```

使用defender-autotask-client，您可以这样调用更新端点：
```
const myAutotask = {
  autotaskId: "671d1f80-99e3-4829-aa15-f01e3298e428",
  name: "my-autotask",
  trigger: {
    type: 'schedule',
    frequencyMinutes: 1700,
  },
  paused: true
};
await client.update(myAutotask);
```

一个响应示例
```
{
  autotaskId: '671d1f80-99e3-4829-aa15-f01e3298e428',
  name: 'my-autotask',
  paused: true,
  trigger: { type: 'schedule', frequencyMinutes: 1700 },
  encodedZippedCode: 'UEsDBAoAAAAAAAFzWFTA91VHFwAAABcAAAAHAAAAZGVwcy5qc2V4cG9ydHMuZm9vID0gKCkgPT4gNDI7UEsDBAoAAAAAAAFzWFSEyiyCiQAAAIkAAAAIAAAAaW5kZXguanNjb25zdCBkZXBzID0gcmVxdWlyZSgnLi9kZXBzJyk7CgpleHBvcnRzLmhhbmRsZXIgPSBhc3luYyBmdW5jdGlvbigpIHsKICBjb25zdCB2YWx1ZSA9IGRlcHMuZm9vKCk7CiAgY29uc29sZS5sb2codmFsdWUpOwogIHJldHVybiB2YWx1ZTsKfVBLAQIUAAoAAAAAAAFzWFTA91VHFwAAABcAAAAHAAAAAAAAAAAAAAAAAAAAAABkZXBzLmpzUEsBAhQACgAAAAAAAXNYVITKLIKJAAAAiQAAAAgAAAAAAAAAAAAAAAAAPAAAAGluZGV4LmpzUEsFBgAAAAACAAIAawAAAOsAAAAAAA=='
}
```

## 删除终端点
autotasks/{id}终端点用于通过DELETE请求删除Autotask。该终端点接受一个autotask Id。

使用defender-autotask-client，您可以调用删除终端点如下：
```
await client.delete("671d1f80-99e3-4829-aa15-f01e3298e428");
```

一个响应示例
```
  message: '671d1f80-99e3-4829-aa15-f01e3298e428 deleted'
```

## 更新代码端点

autotasks/{id}/code端点用于通过PUT请求上传新的Autotask代码。该端点接受一个JSON对象，其中包含一个encodedZippedCode属性，该属性对应于包含代码包的base64编码的zip文件。

```
zip -r code.zip index.js

curl \
  -X PUT \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "{ \"encodedZippedCode\": \"$(cat code.zip | base64 -w0)\" }" \
    "https://defender-api.openzeppelin.com/autotask/autotasks/${AUTOTASKID}/code"
```

或者通过defender-autotask-client这样做：
```
await client.updateCodeFromFolder("671d1f80-99e3-4829-aa15-f01e3298e428", './code');
```

> NOTE
您可以在捆绑包中包含多个文件，只要捆绑包在压缩和base64编码后不超过5mb，并在zip文件的根目录下包含一个index.js作为入口点。

## Autotask Runs Endpoints
使用autotasks/{id}/runs/manual端点可以手动触发autotask运行：
```
curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://defender-api.openzeppelin.com/autotask/autotasks/${AUTOTASKID}/runs/manual"
```

或通过defender-autotask-client进行如下操作：
```
await client.runAutotask("671d1f80-99e3-4829-aa15-f01e3298e428");
```

可以使用以下命令列出Autotask运行数据：
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/autotask/autotasks/${AUTOTASKID}/runs"
```

或者通过以下方法进行操作：
```
await client.listAutotaskRuns("671d1f80-99e3-4829-aa15-f01e3298e428");
```

可以使用AUTOTASK_RUN_ID（从上面的列表请求中获取）获取特定运行的日志:
```
curl \
  -X GET \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
    "https://defender-api.openzeppelin.com/autotask/autotasks/runs/${AUTOTASK_RUN_ID}"
```

```
// 这个方法的参数是autotask运行ID，而不是autotask ID。
await client.getAutotaskRun("ae729f92-11e2-0012-bb16-c98c3298e112");
```

##  secrets 终端点
autotasks / secrets终端点可用于创建和删除（但不能获取）secrets：
```
curl \
  -X POST \
  -H 'Accept: application/json' \
  -H 'Content-Type: application/json' \
  -H "X-Api-Key: $KEY" \
  -H "Authorization: Bearer $TOKEN" \
  -d "$DATA" \
    "https://defender-api.openzeppelin.com/autotask/secrets"
```

或者通过defender-autotask-client，如下所示。删除数组和 secrets 对象都必须出现在有效载荷中，但可以包含空值。例如，以下调用都是有效的：
```
await client.createSecrets({ deletes: [], secrets: { foo: 'bar' } });

await client.createSecrets({ deletes: ['foo'], secrets: { baz: 'test' } });

await client.createSecrets({ deletes: ['baz'], secrets: { } });
```

