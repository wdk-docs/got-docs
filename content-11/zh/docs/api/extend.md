---
title: "高级创建-扩展实例-extend"
linkTitle: "扩展实例"
weight: 5
description: >
  让调用 REST API 的更容易通过创建利基特定`got`实例。
type: "docs"
---

## 合并实例

`Got`载体构成多个实例在一起。
这是非常强大的。
您可以创建一个客户端，限制下载速度，然后用一个实例标志的要求撰写的。
这就像没有任何插件惹的插件。
你只要创建实例，然后组合在一起。

将它们混合使用 `instanceA.extend(instanceB, instanceC, ...)`, 就这样.

该`options`使用[`got.mergeOptions`](#gotmergeoptionsparentoptions-newoptions)与父实例的`defaults.options`合并.

## got.extend(...options)

使用默认`options`配置一个新的`got`实例。 您可以使用实例上的`.defaults`属性来访问选项。

```js
const client = got.extend({
	prefixUrl: "https://example.com",
	headers: {
		"x-unicorn": "rainbow"
	}
});

client.get("demo");

/* HTTP Request =>
 * GET /demo HTTP/1.1
 * Host: example.com
 * x-unicorn: rainbow
 */
```

```js
(async () => {
	const client = got.extend({
		prefixUrl: "httpbin.org",
		headers: {
			"x-foo": "bar"
		}
	});
	const { headers } = await client.get("headers").json();
	//=> headers['x-foo'] === 'bar'

	const jsonClient = client.extend({
		responseType: "json",
		resolveBodyOnly: true,
		headers: {
			"x-baz": "qux"
		}
	});
	const { headers: headers2 } = await jsonClient.get("headers");
	//=> headers2['x-foo'] === 'bar'
	//=> headers2['x-baz'] === 'qux'
})();
```

此外，`got.extend()`从`defaults`对象接受两个属性: `mutableDefaults` 和 `handlers`. 例:

```js
// You can now modify `mutableGot.defaults.options`.
const mutableGot = got.extend({ mutableDefaults: true });

const mergedHandlers = got.extend({
	handlers: [
		(options, next) => {
			delete options.headers.referer;

			return next(options);
		}
	]
});
```

{{% alert color="primary" %}}
处理程序可以是异步的。推荐的方法是:
{{% /alert %}}

```js
const handler = (options, next) => {
	if (options.stream) {
		// It's a Stream
		return next(options);
	}

	// It's a Promise
	return (async () => {
		try {
			const response = await next(options);
			response.yourOwnProperty = true;
			return response;
		} catch (error) {
			// Every error will be replaced by this one.
			// Before you receive any error here,
			// it will be passed to the `beforeError` hooks first.
			// Note: this one won't be passed to `beforeError` hook. It's final.
			throw new Error("Your very own error.");
		}
	})();
};

const instance = got.extend({ handlers: [handler] });
```

## got.extend(...instances)

合并多实例为一个:

- 选择使用[`got.mergeOptions()`](#gotmergeoptionsparentoptions-newoptions)合并 (+ 挂钩也合并),
- 处理程序被存储在数组中 (你可以通过`instance.defaults.handlers`访问他们).

## got.extend(...options, ...instances, ...)

这是可能的合并选项和实例。\
它给作为`got.extend(...options).extend(...instances)`同样的效果:

```js
const a = { headers: { cat: "meow" } };
const b = got.extend({
	options: {
		headers: {
			cow: "moo"
		}
	}
});

// 同为 `got.extend(a).extend(b)`.
// 注意 `a` 是选项 和 `b` 是一个实例.
got.extend(a, b);
//=> {headers: {cat: 'meow', cow: 'moo'}}
```

## 合并例子

下面的例子描述了你在什么样的情况下可以编辑一些需要合并的例子:

### 比规定的禁止，导致重定向到其他站点

```js
const controlRedirects = got.extend({
	handlers: [
		(options, next) => {
			const promiseOrStream = next(options);
			return promiseOrStream.on("redirect", response => {
				const host = new URL(resp.url).host;
				if (options.allowedHosts && !options.allowedHosts.includes(host)) {
					promiseOrStream.cancel(`Redirection to ${host} is not allowed`);
				}
			});
		}
	]
});
```

### 限制下载和上传大小

当您的机器已经有限的内存容量，可以是有用的。

```js
const limitDownloadUpload = got.extend({
	handlers: [
		(options, next) => {
			let promiseOrStream = next(options);
			if (typeof options.downloadLimit === "number") {
				promiseOrStream.on("downloadProgress", progress => {
					if (
						progress.transferred > options.downloadLimit &&
						progress.percent !== 1
					) {
						promiseOrStream.cancel(
							`Exceeded the download limit of ${options.downloadLimit} bytes`
						);
					}
				});
			}

			if (typeof options.uploadLimit === "number") {
				promiseOrStream.on("uploadProgress", progress => {
					if (
						progress.transferred > options.uploadLimit &&
						progress.percent !== 1
					) {
						promiseOrStream.cancel(
							`Exceeded the upload limit of ${options.uploadLimit} bytes`
						);
					}
				});
			}

			return promiseOrStream;
		}
	]
});
```

### 没有用户代理

```js
const noUserAgent = got.extend({
	headers: {
		"user-agent": null
	}
});
```

### 定义基础网址

```js
const httpbin = got.extend({
	prefixUrl: "https://httpbin.org/"
});
```

### 设置头部签名

```js
const crypto = require("crypto");

const getMessageSignature = (data, secret) =>
	crypto
		.createHmac("sha256", secret)
		.update(data)
		.digest("hex")
		.toUpperCase();
const signRequest = got.extend({
	hooks: {
		beforeRequest: [
			options => {
				options.headers["sign"] = getMessageSignature(
					options.body || "",
					process.env.SECRET
				);
			}
		]
	}
});
```

### 合并所有实例

如果这些实例都是不同的模块，并且你不想重写它们， 用 `got.extend(...instances)`合并.

{{% alert color="primary" %}}
`noUserAgent`实例作为实例必须放在链的末端，以便合并。 否则`user-agent`还会有内容。
{{% /alert %}}

```js
const merged = got.extend(
	controlRedirects,
	limitDownloadUpload,
	httpbin,
	signRequest,
	noUserAgent
);

(async () => {
	// 有没有'user-agent'头 :)
	await merged("/");
	/* HTTP Request =>
	 * GET / HTTP/1.1
	 * accept-encoding: gzip, deflate, br
	 * sign: F9E66E179B6747AE54108F82F8ADE8B3C25D76FD30AFDE6C395822C530196169
	 * Host: httpbin.org
	 * Connection: close
	 */

	const MEGABYTE = 1048576;
	await merged("http://ipv4.download.thinkbroadband.com/5MB.zip", {
		downloadLimit: MEGABYTE,
		prefixUrl: ""
	});
	// CancelError: 超过1048576个字节的下载限制

	await merged("https://jigsaw.w3.org/HTTP/300/301.html", {
		allowedHosts: ["google.com"],
		prefixUrl: ""
	});
	// CancelError: 不允许重定向到jigsaw.w3.org
})();
```
