# 缓存

Got实现了[RFC 7234](https://httpwg.org/specs/rfc7234.html)兼容的HTTP缓存，它可以在内存中开箱即用，并且可以很容易地插入各种存储适配器。
新的缓存条目直接从缓存中提供，过时的缓存条目使用`If-None-Match` / `If-Modified-Since` 头重新验证。
你可以在[`cacheable-request` 文档](https://github.com/lukechilds/cacheable-request)中阅读更多关于底层缓存行为的信息。

你可以使用JavaScript的`Map`类型作为内存缓存:

```js
import got from "got";

const map = new Map();

let response = await got("https://sindresorhus.com", { cache: map });
console.log(response.isFromCache);
//=> false

response = await got("https://sindresorhus.com", { cache: map });
console.log(response.isFromCache);
//=> true
```

Got在内部使用[Keyv](https://github.com/lukechilds/keyv)来支持各种存储适配器。
对于更可伸缩的东西，您可以使用[官方Keyv存储适配器](https://github.com/lukechilds/keyv#official-storage-adapters):)

```
$ npm install @keyv/redis
```

```js
import got from "got";
import KeyvRedis from "@keyv/redis";

const redis = new KeyvRedis("redis://user:pass@localhost:6379");

await got("https://sindresorhus.com", { cache: redis });
```

Got支持任何遵循Map API的东西，因此编写自己的存储适配器或使用第三方解决方案都很容易。

例如，以下都是有效的存储适配器:

```js
const storageAdapter = new Map();

await got("https://sindresorhus.com", { cache: storageAdapter });
```

```js
import storageAdapter from "./my-storage-adapter";

await got("https://sindresorhus.com", { cache: storageAdapter });
```

```js
import QuickLRU from "quick-lru";

const storageAdapter = new QuickLRU({ maxSize: 1000 });

await got("https://sindresorhus.com", { cache: storageAdapter });
```

View the [Keyv docs](https://github.com/lukechilds/keyv) for more information on how to use storage adapters.

### 高级缓存机制

`request` 函数可能返回一个`IncomingMessage`类的实例。

```js
import https from "node:https";
import { Readable } from "node:stream";
import got from "got";

const getCachedResponse = (url, options) => {
  const response = new Readable({
    read() {
      this.push("Hello, world!");
      this.push(null);
    },
  });

  response.statusCode = 200;
  response.headers = {};
  response.trailers = {};
  response.socket = null;
  response.aborted = false;
  response.complete = true;
  response.httpVersion = "1.1";
  response.httpVersionMinor = 1;
  response.httpVersionMajor = 1;

  return response;
};

const instance = got.extend({
  request: (url, options, callback) => {
    return getCachedResponse(url, options);
  },
});

const body = await instance("https://example.com").text();

console.log(body);
//=> "Hello, world!"
```

If you don't want to alter the `request` function, you can return a cached response in a `beforeRequest` hook:

```js
import https from "node:https";
import { Readable } from "node:stream";
import got from "got";

const getCachedResponse = (url, options) => {
  const response = new Readable({
    read() {
      this.push("Hello, world!");
      this.push(null);
    },
  });

  response.statusCode = 200;
  response.headers = {};
  response.trailers = {};
  response.socket = null;
  response.aborted = false;
  response.complete = true;
  response.httpVersion = "1.1";
  response.httpVersionMinor = 1;
  response.httpVersionMajor = 1;

  return response;
};

const instance = got.extend({
  hooks: {
    beforeRequest: [
      (options) => {
        return getCachedResponse(options.url, options);
      },
    ],
  },
});

const body = await instance("https://example.com").text();

console.log(body);
//=> "Hello, world!"
```

If you want to prevent duplicating the same requests, you can use a handler instead.

```js
import got from "got";

const map = new Map();

const instance = got.extend({
  handlers: [
    (options, next) => {
      if (options.isStream) {
        return next(options);
      }

      const pending = map.get(options.url.href);
      if (pending) {
        return pending;
      }

      const promise = next(options);

      map.set(options.url.href, promise);
      promise.finally(() => {
        map.delete(options.url.href);
      });

      return promise;
    },
  ],
});

const [first, second] = await Promise.all([
  instance("https://httpbin.org/anything"),
  instance("https://httpbin.org/anything"),
]);

console.log(first === second);
//=> true
```
