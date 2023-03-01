---
title: "缓存-cache"
linkTitle: "缓存"
weight: 13
description: >
  `Got`实现[RFC 7234](http://httpwg.org/specs/rfc7234.html)兼容 HTTP 缓存 它的工作原理开箱内存 并且是具有广泛的存储适配器的易插拔.
type: "docs"
---

新鲜缓存条目直接从缓存中, 和陈旧的缓存条目用`If-None-Match`/`If-Modified-Since`头重新验证.
您可以在[`cacheable-request` 文档](https://github.com/lukechilds/cacheable-request)中读取更多关于底层缓存行为 .
对于 DNS 缓存, `Got`使用[`cacheable-lookup`](https://github.com/szmarczak/cacheable-lookup).

您可以使用 JavaScript`Map`类型作为内存缓存:

```js
const got = require("got");

const map = new Map();

(async () => {
	let response = await got("https://sindresorhus.com", { cache: map });
	console.log(response.isFromCache);
	//=> false

	response = await got("https://sindresorhus.com", { cache: map });
	console.log(response.isFromCache);
	//=> true
})();
```

`Got`在内部使用[KEYV](https://github.com/lukechilds/keyv)，以支持广泛的存储适配器。 对于一些更具扩展性，你可以使用[官方 KEYV 存储适配器](https://github.com/lukechilds/keyv#official-storage-adapters):

```
$ npm install @keyv/redis
```

```js
const got = require("got");
const KeyvRedis = require("@keyv/redis");

const redis = new KeyvRedis("redis://user:pass@localhost:6379");

got("https://sindresorhus.com", { cache: redis });
```

`Got`支持任何下面的 Map API，所以很容易写自己的存储适配器或使用第三方解决方案。

例如，下面都是有效的存储适配器:

```js
const storageAdapter = new Map();
// Or
const storageAdapter = require("./my-storage-adapter");
// Or
const QuickLRU = require("quick-lru");
const storageAdapter = new QuickLRU({ maxSize: 1000 });

got("https://sindresorhus.com", { cache: storageAdapter });
```

查看[Keyv 文档](https://github.com/lukechilds/keyv)了解如何使用存储适配器的详细信息。
