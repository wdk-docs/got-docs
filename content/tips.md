# 技巧

## 超时

每个请求可以有一个最大允许运行时间。
为了使用此功能，请指定`request`超时选项。

```js
import got from "got";

const body = await got("https://httpbin.org/anything", {
  timeout: {
    request: 30000,
  },
});
```

有关更具体的超时，请访问[超时 API](6-timeout.md).

## 重试

默认情况下，如果可能，Got会对失败的请求进行新的重试。

通过将允许的最大重试次数设置为`0`，可以完全禁用此功能。

```js
import got from "got";

const noRetryGot = got.extend({
  retry: {
    limit: 0,
  },
});
```

要指定可检索的错误，请使用[重试API](7-retry.md).

## Cookies

Got 支持 cookies 开箱即用。
不需要手动解析它们。
为了使用cookie，从[`tough-cookie`](https://github.com/salesforce/tough-cookie)包中传递一个`CookieJar`实例。

```js
import got from "got";
import { CookieJar } from "tough-cookie";

const cookieJar = new CookieJar();

await cookieJar.setCookie("foo=bar", "https://httpbin.org");
await got("https://httpbin.org/anything", { cookieJar });
```

## AWS

对AWS服务的请求需要对其标头进行签名。
这可以通过使用 [`got4aws`](https://github.com/SamVerschueren/got4aws)包来完成。

这是一个使用签名请求查询[`API 网关`](https://docs.aws.amazon.com/apigateway/api-reference/signing-requests/) 的示例。

```js
import got4aws from "got4aws";

const got = got4aws();

const response = await got("https://<api-id>.execute-api.<api-region>.amazonaws.com/<stage>/endpoint/path", {
  // …
});
```

## 分页

在处理大型数据集时，使用分页是非常有效的
默认情况下，Got使用[`Link` 头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Link)来检索下一页。
但是，这种行为可以自定义，参见[分页API](4-pagination.md)。

```js
const countLimit = 50;

const pagination = got.paginate("https://api.github.com/repos/sindresorhus/got/commits", {
  pagination: { countLimit },
});

console.log(`Printing latest ${countLimit} Got commits (newest to oldest):`);

for await (const commitData of pagination) {
  console.log(commitData.commit.message);
}
```

<a name="unix"></a>

## UNIX域套接字

参考[`enableUnixSockets` 选项](./2-options.md#enableunixsockets).

## 测试



Got使用本机[`http`](https://nodejs.org/api/http.html)模块，该模块依赖于本机[`net`](https://nodejs.org/api/net.html)模块。
这意味着有两种可能的测试方法:

1. 使用像[`nock`]这样的mock库(https://github.com/nock/nock),
2. 创建服务器。

第一种方法应该涵盖所有常见的用例
请记住，它覆盖了本地的`http`模块，因此可能会由于差异而出现bug。

最可靠的方法是创建一个服务器
在某些情况下，`nock` 可能不够或缺乏功能。

### Nock

By default `nock` mocks only one request.\
Got will [retry](7-retry.md) on failed requests by default, causing a `No match for request ...` error.\
The solution is to either disable retrying (set `options.retry.limit` to `0`) or call `.persist()` on the mocked request.

默认情况下，`nock`只模拟一个请求。
默认情况下，Got将对失败的请求进行[retry](7-retry.md)，导致`No match for request ...`的错误。
解决方案是禁用重试(将`options.retry.limit`设置为`0`)或在模拟请求上调用`.persist()`。

```js
import got from "got";
import nock from "nock";

const scope = nock("https://sindresorhus.com").get("/").reply(500, "Internal server error").persist();

try {
  await got("https://sindresorhus.com");
} catch (error) {
  console.log(error.response.body);
  //=> 'Internal server error'

  console.log(error.response.retryCount);
  //=> 2
}

scope.persist(false);
```

## 代理

!!! note

    流行的[`tunnel`](https://www.npmjs.com/package/tunnel)包未维护。使用风险自负。  
    [`proxy-agent`](https://www.npmjs.com/package/proxy-agent)家族不遵循最新的Node.js特性，缺乏支持。


虽然没有一个完美的、没有错误的软件包，但[Apify](https://apify.com/)解决方案是一个现代的解决方案
查看[`got-scraping/src/agent/h1-proxy-agent.ts`](https://github.com/apify/got-scraping/blob/2ec7f9148917a6a38d6d1c8c695606767c46cce5/src/agent/h1-proxy-agent.ts)。
它具有与`hpagent`相同的API。

[`hpagent`](https://github.com/delvedor/hpagent) is a modern package as well. In contrast to `tunnel`, it allows keeping the internal sockets alive to be reused.

```js
import got from "got";
import { HttpsProxyAgent } from "hpagent";

await got("https://sindresorhus.com", {
  agent: {
    https: new HttpsProxyAgent({
      keepAlive: true,
      keepAliveMsecs: 1000,
      maxSockets: 256,
      maxFreeSockets: 256,
      scheduling: "lifo",
      proxy: "https://localhost:8080",
    }),
  },
});
```

Alternatively, use [`global-agent`](https://github.com/gajus/global-agent) to configure a global proxy for all HTTP/HTTPS traffic in your program.

If you're using HTTP/2, the [`http2-wrapper`](https://github.com/szmarczak/http2-wrapper/#proxy-support) package provides proxy support out-of-box.\
[Learn more.](https://github.com/szmarczak/http2-wrapper#proxy-support)

## 在没有代理的情况下重试

如果使用代理，可能会遇到连接问题。
一种解决方法是在重试时禁用代理。
流API的解决方案是这样的:

```js
import https from "https";
import fs from "fs";
import got from "got";

class MyAgent extends https.Agent {
  createConnection(port, options, callback) {
    console.log(`Connecting with MyAgent`);
    return https.Agent.prototype.createConnection.call(this, port, options, callback);
  }
}

const proxy = new MyAgent();

let writeStream;

const fn = (retryStream) => {
  const options = {
    agent: {
      https: proxy,
    },
  };

  const stream = retryStream ?? got.stream("https://example.com", options);

  if (writeStream) {
    writeStream.destroy();
  }

  writeStream = fs.createWriteStream("example-com.html");

  stream.pipe(writeStream);
  stream.once("retry", (retryCount, error, createRetryStream) => {
    fn(
      createRetryStream({
        agent: {
          http: undefined,
          https: undefined,
          http2: undefined,
        },
      })
    );
  });
};

fn();
```

## `h2c`

没有直接的[`h2c`](https://datatracker.ietf.org/doc/html/rfc7540#section-3.1)支持。

然而，你可以在`beforeRequest`钩子中提供一个`h2session`选项。
参见[例子](examples/h2c.js)。

## 大写标头

Got总是将标头规范化，因此传递`Uppercase-Header`会将其转换为`uppercase-header`。
要解决这个问题，你需要传递一个包装代理:

```js
class WrappedAgent {
  constructor(agent) {
    this.agent = agent;
  }

  addRequest(request, options) {
    return this.agent.addRequest(request, options);
  }

  get keepAlive() {
    return this.agent.keepAlive;
  }

  get maxSockets() {
    return this.agent.maxSockets;
  }

  get options() {
    return this.agent.options;
  }

  get defaultPort() {
    return this.agent.defaultPort;
  }

  get protocol() {
    return this.agent.protocol;
  }
}

class TransformHeadersAgent extends WrappedAgent {
  addRequest(request, options) {
    const headers = request.getHeaderNames();

    for (const header of headers) {
      request.setHeader(this.transformHeader(header), request.getHeader(header));
    }

    return super.addRequest(request, options);
  }

  transformHeader(header) {
    return header
      .split("-")
      .map((part) => {
        return part[0].toUpperCase() + part.slice(1);
      })
      .join("-");
  }
}

const agent = new http.Agent({
  keepAlive: true,
});

const wrappedAgent = new TransformHeadersAgent(agent);
```

参看[例子](examples/uppercase-headers.js).

## 自定义选项

当一个选项不存在时 Got v12 抛出。因此传递一个顶级选项，如:

```js
import got from "got";

await got("https://example.com", {
  foo: "bar",
});
```

将抛出。为了防止这种情况，你需要在`init`钩子中读取该选项:

```js
import got from "got";

const convertFoo = got.extend({
  hooks: {
    init: [
      (rawOptions, options) => {
        if ("foo" in rawOptions) {
          options.context.foo = rawOptions.foo;
          delete rawOptions.foo;
        }
      },
    ],
  },
});

const instance = got.extend(convertFoo, {
  hooks: {
    beforeRequest: [
      (options) => {
        options.headers.foo = options.context.foo;
      },
    ],
  },
});

const { headers } = await instance("https://httpbin.org/anything", { foo: "bar" }).json();
console.log(headers.Foo); //=> 'bar'
```

最后，您可能想要创建一个捕获所有实例:

```js
import got from "got";

const catchAllOptions = got.extend({
  hooks: {
    init: [
      (raw, options) => {
        for (const key in raw) {
          if (!(key in options)) {
            options.context[key] = raw[key];
            delete raw[key];
          }
        }
      },
    ],
  },
});

const instance = got.extend(catchAllOptions, {
  hooks: {
    beforeRequest: [
      (options) => {
        // All custom options will be visible under `options.context`
        options.headers.foo = options.context.foo;
      },
    ],
  },
});

const { headers } = await instance("https://httpbin.org/anything", { foo: "bar" }).json();
console.log(headers.Foo); //=> 'bar'
```

!!! note

    在`init`钩子中执行验证是一个很好的做法。
    当选项未知时，可以安全地抛出!
    在内部，Got使用 [`@sindresorhus/is`](https://github.com/sindresorhus/is) 包。

## 不支持Electron  `net` 模块

!!! note 
    
    得到了v12和更高版本的ESM包，但是Electron还不支持ESM。所以你需要使用Got v11。

Got doesn't support the `electron.net` module. It's missing crucial APIs that are available in Node.js.\
While Got used to support `electron.net`, it got very unstable and caused many errors.

However, you can use [IPC communication](https://www.electronjs.org/docs/api/ipc-main#ipcmainhandlechannel-listener) to get the Response object:

```js
// Main process
const got = require("got");

const instance = got.extend({
  // ...
});

ipcMain.handle("got", async (event, ...args) => {
  const { statusCode, headers, body } = await instance(...args);
  return { statusCode, headers, body };
});

// Renderer process
async () => {
  const { statusCode, headers, body } = await ipcRenderer.invoke("got", "https://httpbin.org/anything");
  // ...
};
```
