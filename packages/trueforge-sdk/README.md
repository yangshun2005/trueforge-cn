# TrueForge TypeScript 库

[![fern shield](https://img.shields.io/badge/%F0%9F%8C%BF-Built%20with%20Fern-brightgreen)](https://buildwithfern.com?utm_source=github&utm_medium=github&utm_campaign=readme&utm_source=Truefoundry%2FTypeScript)
[![npm shield](https://img.shields.io/npm/v/@truefoundry/trueforge-sdk)](https://www.npmjs.com/package/@truefoundry/trueforge-sdk)

TrueForge Agent Harness Server 的 TypeScript 客户端：一个自托管运行时，用于执行 Agent 回合并通过 Server-Sent Events 进行流式传输。

当服务器启用认证时，可通过可选的 `token` 客户端选项传递 ID 令牌（`Authorization: Bearer`）。

## 目录

- [文档](#文档)
- [安装](#安装)
- [参考](#参考)
- [用法](#用法)
- [请求和响应类型](#请求和响应类型)
- [异常处理](#异常处理)
- [流式响应](#流式响应)
- [二进制响应](#二进制响应)
- [高级](#高级)
  - [子包导出](#子包导出)
  - [附加请求头](#附加请求头)
  - [附加查询字符串参数](#附加查询字符串参数)
  - [重试](#重试)
  - [超时](#超时)
  - [中止请求](#中止请求)
  - [访问原始响应数据](#访问原始响应数据)
  - [日志记录](#日志记录)
  - [自定义 Fetch](#自定义-fetch)
  - [自定义 Fetcher](#自定义-fetcher)
  - [运行时兼容性](#运行时兼容性)
- [贡献](#贡献)

## 文档

API 参考文档可在[此处](https://trueforge.dev/api/use-agent)获取。

## 安装

```sh
npm i -s @truefoundry/trueforge-sdk
```

## 参考

该库的完整参考可在[此处](./reference.md)获取。

## 用法

使用以下方式实例化并使用客户端：

```typescript
import { TrueForge } from "@truefoundry/trueforge-sdk";

const client = new TrueForge({ baseUrl: "YOUR_BASE_URL", token: "YOUR_TOKEN" });
const response = await client.sessions.createTurnStream("session_id", {});
for await (const item of response) {
    console.log(item);
}
```

## 请求和响应类型

SDK 将所有请求和响应类型导出为 TypeScript 接口。只需使用以下命名空间导入它们：

```typescript
import { TrueForge } from "@truefoundry/trueforge-sdk";

const request: TrueForge.CreateAgentRequest = {
    ...
};
```

## 异常处理

当 API 返回非成功状态码（4xx 或 5xx 响应）时，将抛出以下错误类的子类。

```typescript
import { TrueForgeError } from "@truefoundry/trueforge-sdk";

try {
    await client.sessions.createTurnStream(...);
} catch (err) {
    if (err instanceof TrueForgeError) {
        console.log(err.statusCode);
        console.log(err.message);
        console.log(err.body);
        console.log(err.rawResponse);
    }
}
```

## 流式响应

某些端点返回流式响应，而不是一次性返回完整响应。SDK 使用异步迭代器，因此您可以使用 `for await...of` 循环来消费响应。

```typescript
import { TrueForge } from "@truefoundry/trueforge-sdk";

const client = new TrueForge({ baseUrl: "YOUR_BASE_URL", token: "YOUR_TOKEN" });
const response = await client.sessions.createTurnStream("session_id", {});
for await (const item of response) {
    console.log(item);
}
```

## 二进制响应

您可以使用 `BinaryResponse` 类型从端点消费二进制数据，该类型允许您选择如何消费数据：

```typescript
const response = await client.sessions.downloadSandboxFile(...);
const stream: ReadableStream<Uint8Array> = response.stream();
// const arrayBuffer: ArrayBuffer = await response.arrayBuffer();
// const blob: Blob = response.blob();
// const bytes: Uint8Array = response.bytes();
// 响应体只能使用一次，因此您必须选择上述方法之一。
// 如果您想检查响应体是否已被使用，可以使用以下属性。
const bodyUsed = response.bodyUsed;
```
<details>
<summary>将二进制响应保存到文件</summary>

<blockquote>
<details>
<summary>Node.js</summary>

<blockquote>
<details>
<summary>ReadableStream（最高效）</summary>

```ts
import { createWriteStream } from 'fs';
import { Readable } from 'stream';
import { pipeline } from 'stream/promises';

const response = await client.sessions.downloadSandboxFile(...);

const stream = response.stream();
const nodeStream = Readable.fromWeb(stream);
const writeStream = createWriteStream('path/to/file');

await pipeline(nodeStream, writeStream);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>ArrayBuffer</summary>

```ts
import { writeFile } from 'fs/promises';

const response = await client.sessions.downloadSandboxFile(...);

const arrayBuffer = await response.arrayBuffer();
await writeFile('path/to/file', Buffer.from(arrayBuffer));
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Blob</summary>

```ts
import { writeFile } from 'fs/promises';

const response = await client.sessions.downloadSandboxFile(...);

const blob = await response.blob();
const arrayBuffer = await blob.arrayBuffer();
await writeFile('output.bin', Buffer.from(arrayBuffer));
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Bytes（UIntArray8）</summary>

```ts
import { writeFile } from 'fs/promises';

const response = await client.sessions.downloadSandboxFile(...);

const bytes = await response.bytes();
await writeFile('path/to/file', bytes);
```

</details>
</blockquote>

</details>
</blockquote>

<blockquote>
<details>
<summary>Bun</summary>

<blockquote>
<details>
<summary>ReadableStream（最高效）</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const stream = response.stream();
await Bun.write('path/to/file', stream);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>ArrayBuffer</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const arrayBuffer = await response.arrayBuffer();
await Bun.write('path/to/file', arrayBuffer);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Blob</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const blob = await response.blob();
await Bun.write('path/to/file', blob);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Bytes（UIntArray8）</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const bytes = await response.bytes();
await Bun.write('path/to/file', bytes);
```

</details>
</blockquote>

</details>
</blockquote>

<blockquote>
<details>
<summary>Deno</summary>

<blockquote>
<details>
<summary>ReadableStream（最高效）</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const stream = response.stream();
const file = await Deno.open('path/to/file', { write: true, create: true });
await stream.pipeTo(file.writable);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>ArrayBuffer</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const arrayBuffer = await response.arrayBuffer();
await Deno.writeFile('path/to/file', new Uint8Array(arrayBuffer));
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Blob</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const blob = await response.blob();
const arrayBuffer = await blob.arrayBuffer();
await Deno.writeFile('path/to/file', new Uint8Array(arrayBuffer));
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Bytes（UIntArray8）</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const bytes = await response.bytes();
await Deno.writeFile('path/to/file', bytes);
```

</details>
</blockquote>

</details>
</blockquote>

<blockquote>
<details>
<summary>浏览器</summary>

<blockquote>
<details>
<summary>Blob（最高效）</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const blob = await response.blob();
const url = URL.createObjectURL(blob);

// 触发下载
const a = document.createElement('a');
a.href = url;
a.download = 'filename';
a.click();
URL.revokeObjectURL(url);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>ReadableStream</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const stream = response.stream();
const reader = stream.getReader();
const chunks = [];

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  chunks.push(value);
}

const blob = new Blob(chunks);
const url = URL.createObjectURL(blob);

// 触发下载
const a = document.createElement('a');
a.href = url;
a.download = 'filename';
a.click();
URL.revokeObjectURL(url);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>ArrayBuffer</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const arrayBuffer = await response.arrayBuffer();
const blob = new Blob([arrayBuffer]);
const url = URL.createObjectURL(blob);

// 触发下载
const a = document.createElement('a');
a.href = url;
a.download = 'filename';
a.click();
URL.revokeObjectURL(url);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Bytes（UIntArray8）</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const bytes = await response.bytes();
const blob = new Blob([bytes]);
const url = URL.createObjectURL(blob);

// 触发下载
const a = document.createElement('a');
a.href = url;
a.download = 'filename';
a.click();
URL.revokeObjectURL(url);
```

</details>
</blockquote>

</details>
</blockquote>

</details>
</blockquote>

<details>
<summary>将二进制响应转换为文本</summary>

<blockquote>
<details>
<summary>ReadableStream</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const stream = response.stream();
const text = await new Response(stream).text();
```

</details>
</blockquote>

<blockquote>
<details>
<summary>ArrayBuffer</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const arrayBuffer = await response.arrayBuffer();
const text = new TextDecoder().decode(arrayBuffer);
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Blob</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const blob = await response.blob();
const text = await blob.text();
```

</details>
</blockquote>

<blockquote>
<details>
<summary>Bytes（UIntArray8）</summary>

```ts
const response = await client.sessions.downloadSandboxFile(...);

const bytes = await response.bytes();
const text = new TextDecoder().decode(bytes);
```

</details>
</blockquote>

</details>

## 高级

### 子包导出

此 SDK 支持直接导入子包客户端，这允许 JavaScript 打包器进行 tree-shaking，仅包含导入的子包代码。这样可以显著减小打包体积。

```typescript
import { AgentsClient } from '@truefoundry/trueforge-sdk/agents';

const client = new AgentsClient({...});
```

### 附加请求头

如果您希望在请求中发送附加请求头，请使用 `headers` 请求选项。

```typescript
import { TrueForge } from "@truefoundry/trueforge-sdk";

const client = new TrueForge({
    ...
    headers: {
        'X-Custom-Header': 'custom value'
    }
});

const response = await client.sessions.createTurnStream(..., {
    headers: {
        'X-Custom-Header': 'custom value'
    }
});
```

### 附加查询字符串参数

如果您希望在请求中发送附加查询字符串参数，请使用 `queryParams` 请求选项。

```typescript
const response = await client.sessions.createTurnStream(..., {
    queryParams: {
        'customQueryParamKey': 'custom query param value'
    }
});
```

### 重试

SDK 配备了带指数退避的自动重试机制。只要请求被视为可重试且重试次数未超过配置的重试限制（默认：2），请求就会被重试。

哪些状态码会被重试取决于 `retryStatusCodes` 生成器配置：

**`legacy`**（当前默认）：重试以下状态码
- [408](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408)（超时）
- [429](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)（请求过多）
- [5XX](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses)（所有服务器错误，包括 500）

**`recommended`**：重试以下状态码
- [408](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408)（超时）
- [429](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)（请求过多）
- [502](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/502)（错误网关）
- [503](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/503)（服务不可用）
- [504](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/504)（网关超时）

使用 `maxRetries` 请求选项来配置此行为。

```typescript
const response = await client.sessions.createTurnStream(..., {
    maxRetries: 0 // 在请求级别覆盖 maxRetries
});
```

### 超时

SDK 默认超时时间为 60 秒。使用 `timeoutInSeconds` 选项来配置此行为。

```typescript
const response = await client.sessions.createTurnStream(..., {
    timeoutInSeconds: 30 // 将超时时间覆盖为 30 秒
});
```

### 中止请求

SDK 允许用户通过传入中止信号（abort signal）在任何时候中止请求。

```typescript
const controller = new AbortController();
const response = await client.sessions.createTurnStream(..., {
    abortSignal: controller.signal
});
controller.abort(); // 中止请求
```

### 访问原始响应数据

SDK 通过 `.withRawResponse()` 方法提供对原始响应数据（包括请求头）的访问。`.withRawResponse()` 方法返回一个 Promise，解析为一个包含 `data` 和 `rawResponse` 属性的对象。

```typescript
const { data, rawResponse } = await client.sessions.createTurnStream(...).withRawResponse();

console.log(data);
console.log(rawResponse.headers['X-My-Header']);
```

### 日志记录

SDK 支持日志记录。您可以通过向客户端选项传入 `logging` 对象来配置日志记录器。

```typescript
import { TrueForge, logging } from "@truefoundry/trueforge-sdk";

const client = new TrueForge({
    ...
    logging: {
        level: logging.LogLevel.Debug, // 默认为 logging.LogLevel.Info
        logger: new logging.ConsoleLogger(), // 默认为 ConsoleLogger
        silent: false, // 默认为 true，设置为 false 以启用日志记录
    }
});
```
`logging` 对象可以包含以下属性：
- `level`：要使用的日志级别。默认为 `logging.LogLevel.Info`。
- `logger`：要使用的日志记录器。默认为 `logging.ConsoleLogger`。
- `silent`：是否静默日志记录器。默认为 `true`。

`level` 属性可以是以下值之一：
- `logging.LogLevel.Debug`
- `logging.LogLevel.Info`
- `logging.LogLevel.Warn`
- `logging.LogLevel.Error`

要提供自定义日志记录器，您可以传入一个实现了 `logging.ILogger` 接口的对象。

<details>
<summary>自定义日志记录器示例</summary>

以下是使用流行的 `winston` 日志库的示例。
```ts
import winston from 'winston';

const winstonLogger = winston.createLogger({...});

const logger: logging.ILogger = {
    debug: (msg, ...args) => winstonLogger.debug(msg, ...args),
    info: (msg, ...args) => winstonLogger.info(msg, ...args),
    warn: (msg, ...args) => winstonLogger.warn(msg, ...args),
    error: (msg, ...args) => winstonLogger.error(msg, ...args),
};
```

以下是使用流行的 `pino` 日志库的示例。

```ts
import pino from 'pino';

const pinoLogger = pino({...});

const logger: logging.ILogger = {
  debug: (msg, ...args) => pinoLogger.debug(args, msg),
  info: (msg, ...args) => pinoLogger.info(args, msg),
  warn: (msg, ...args) => pinoLogger.warn(args, msg),
  error: (msg, ...args) => pinoLogger.error(args, msg),
};
```
</details>


### 自定义 Fetch

SDK 提供了一个底层的 `fetch` 方法，用于发起自定义 HTTP 请求，同时仍然受益于 SDK 级别的配置，如认证、重试、超时和日志记录。这对于调用 SDK 中尚未支持的 API 端点非常有用。

```typescript
```