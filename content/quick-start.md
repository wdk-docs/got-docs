# 快速入门指南

## 获得和发布数据

最简单的`GET`请求:

```js
import got from "got";

const url = "https://httpbin.org/anything";
const response = await got(url);
```

调用返回<code>Promise<[Response](3-streams.md#response-1)></code>。
如果主体包含 JSON，则可以直接检索:

```js
import got from "got";

const url = "https://httpbin.org/anything";
const data = await got(url).json();
```

类似的<code>[got.text](1-promise.md#promisetext)</code>方法返回纯文本。

所有`got`方法都接受一个 options 对象来传递额外的配置，比如头信息:

```js
import got from "got";

const url = "https://httpbin.org/anything";

const options = {
  headers: {
    "Custom-Header": "Quick start",
  },
  timeout: {
    send: 3500,
  },
};

const data = await got(url, options).json();
```

一个 `POST` 请求非常类似:

```js
import got from "got";

const url = "https://httpbin.org/anything";

const options = {
  json: {
    documentName: "Quick Start",
  },
};

const data = await got.post(url, options);
```

请求体在 options 对象中传递。
`json`属性将自动相应地设置标题。
可以像上面一样添加自定义标头。

## 使用流

[Stream API](3-streams.md)允许利用[Node.js Streams](https://nodejs.dev/learn/nodejs-streams)功能:

```js
import fs from "node:fs";
import { pipeline } from "node:stream/promises";
import got from "got";

const url = "https://httpbin.org/anything";

const options = {
  json: {
    documentName: "Quick Start",
  },
};

const gotStream = got.stream.post(url, options);

const outStream = fs.createWriteStream("anything.json");

try {
  await pipeline(gotStream, outStream);
} catch (error) {
  console.error(error);
}
```

## 选项

选项可以在客户端级别设置，并在后续查询中重用:

```js
import got from "got";

const options = {
  prefixUrl: "https://httpbin.org",
  headers: {
    Authorization: getTokenFromVault(),
  },
};

const client = got.extend(options);

export default client;
```

一些常见的选项是:

- [`searchParams`](2-options.md#searchparams): 查询字符串对象。
- [`prefixUrl`](2-options.md#prefixurl): 前置查询路径。路径必须相对于前缀，即不能以`/`开头。
- [`method`](2-options.md#method): HTTP 方法名。
- [`headers`](2-options.md#headers): 查询头。
- [`json`](2-options.md#json): JSON body.
- [`form`](2-options.md#form): 一个表单查询字符串对象。

有关其他[选项](2-options.md#options)，请参阅文档.

## 错误

Promise 和 Stream api 都使用元数据抛出错误。

```js
import got from "got";

try {
  const data = await got.get("https://httpbin.org/status/404");
} catch (error) {
  console.error(error.response.statusCode);
}
```

```js
import got from "got";

const stream = got.stream
  .get("https://httpbin.org/status/404")
  .once("error", (error) => {
    console.error(error.response.statusCode);
  });
```

## 杂项

HTTP 方法名也可以作为一个选项给出，当它只在运行时才知道时，这可能会很方便:

```js
import got from "got";

const url = "https://httpbin.org/anything";

const method = "POST";

const options = {
  method,
  json: {
    documentName: "Quick Start",
  },
};

const data = await got(url, options);
```

对于大多数应用程序，HTTP 客户端只做`GET`和`POST`查询(`PUT`，`PATCH`或`DELETE`方法工作类似)。
下面的部分将提供一些更高级的用法。

### 超时

默认情况下，请求没有超时。一个好的做法是设置一个:

```js
import got from "got";

const options = {
  timeout: {
    request: 10000,
  },
};

const client = got.extend(options);

export default client;
```

上面为导出的`client`发出的所有请求设置了 10000 毫秒的全局超时。
与所有选项一样，超时也可以设置在请求级别。
参见[`timeout` 选项](6-timeout.md#timeout-options)。

### 重试

失败的请求将重试两次。
重试策略可以通过[`retry`](7-retry.md#retry)选项对象进行调优。

```js
import got from "got";

const options = {
  retry: {
    limit: 5,
    errorCodes: ["ETIMEDOUT"],
  },
};
```

stream 的重试就有点棘手了[`stream.on('retry', …)`](3-streams.md#streamonretry-).

### 钩子

钩子是在一些请求事件上调用的自定义函数:

```js
import got from "got";

const logRetry = (error, retryCount) => {
  console.error(`Retrying after error ${error.code}, retry #: ${retryCount}`);
};

const options = {
  hooks: {
    beforeRetry: [logRetry],
  },
};

const client = got.extend(options);

export default client;
```

_注意，钩子以数组的形式给出_, 因此可以给出多个钩子。参见文档了解其他可能的[钩子](9-hooks.md#hooks-api).

### 走得更远

在[文档](../README.md#documentation)和[技巧](tips.md#tips)中还有很多需要发现的地方。
其中，`Got`可以处理[cookies](tips.md#cookies)， [分页](4-pagination.md#pagination-api)， [缓存](cache.md#cache)。
在实现 `Got` :innocent: 已经完成的操作之前，请阅读文档。
