---
title: "迁移指南"
linkTitle: "迁移指南"
weight: 24
type: "docs"
---

> :star: 切换其他 HTTP 请求库到 Got :star:

## 从 request 迁移

你可能认为这是太硬开关，但它真的不是。 🦄

让我们从 request 的自述文件中的第一个示例:

```js
const request = require("request");

request("https://google.com", (error, response, body) => {
	console.log("error:", error);
	console.log("statusCode:", response && response.statusCode);
	console.log("body:", body);
});
```

使用 GOT，它是:

```js
const got = require("got");

(async () => {
	try {
		const response = await got("https://google.com");
		console.log("statusCode:", response.statusCode);
		console.log("body:", response.body);
	} catch (error) {
		console.log("error:", error);
	}
})();
```

看起来更好了，对吧？ 😎

### 常用选项

无论`Request`和`Got`都接受 [`http.request` options](https://nodejs.org/api/http.html#http_http_request_options_callback).

这些`Got`选项`Request`一样用:

- [`url`](https://github.com/sindresorhus/got#url) (+ we accept [`URL`](https://developer.mozilla.org/en-US/docs/Web/API/URL) instances too!)
- [`body`](https://github.com/sindresorhus/got#body)
- [`followRedirect`](https://github.com/sindresorhus/got#followRedirect)
- [`encoding`](https://github.com/sindresorhus/got#encoding)
- [`maxRedirects`](https://github.com/sindresorhus/got#maxredirects)

所以，如果你熟悉他们，你是好去。

还有件事儿... 有没有`time`选项。 假设 [它总是 true](https://github.com/sindresorhus/got#timings).

### 重命名选项

可读性是对我们非常重要，所以我们必须对这些选项不同的名字:

- `qs` → [`searchParams`](https://github.com/sindresorhus/got#searchParams)
- `strictSSL` → [`rejectUnauthorized`](https://github.com/sindresorhus/got#rejectUnauthorized)
- `gzip` → [`decompress`](https://github.com/sindresorhus/got#decompress)
- `jar` → [`cookieJar`](https://github.com/sindresorhus/got#cookiejar) (accepts [`tough-cookie`](https://github.com/salesforce/tough-cookie) jar)

它更清晰，不是吗？

### 改变行为

[`timeout` 选项](https://github.com/sindresorhus/got#timeout) 有一些额外的功能。 您可以[设置特定的事件超时](../readme.md#timeout)!

[`searchParams` 选项](https://github.com/sindresorhus/got#searchParams) 使用[`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) 除非它是一个`string`总是序列化。

要使用的流， 只是调用 `got.stream(url, options)` 要么 `got(url, {isStream: true, ...}`).

### 重大更改

- 该`json`选项不是`boolean`，这是一个`Object`。将字符串化并用作体。
- 该`form`选项是`Object`。 它可以是一个普通的对象或[`form-data` instance](https://github.com/sindresorhus/got/#form-data).
- 没有 `oauth`/`hawk`/`aws`/`httpSignature` 选项. 要登录请求，你需要创建一个[自定义实例](advanced-creation.md#signing-requests).
- 没有 `agentClass`/`agentOptions`/`pool` 选项.
- 没有 `forever` 选项. 您需要使用[forever-agent](https://github.com/request/forever-agent).
- 没有 `proxy` 选项. 您需要[通过自定义代理](../readme.md#proxies).
- 没有 `auth` 选项. 您需要使用`username` / `password` 代替.
- 没有 `baseUrl` 选项. 代替, 有`prefixUrl`如果不存在该附加尾部斜杠。 将始终前缀，除非`url`是 URL 的一个实例。
- 没有 `removeRefererHeader` 选项. 您可以在[`beforeRequest` hook](https://github.com/sindresorhus/got#hooksbeforeRequest)里删除的引用者头:

```js
const gotInstance = got.extend({
	hooks: {
		beforeRequest: [
			options => {
				delete options.headers.referer;
			}
		]
	}
});

gotInstance(url, options);
```

- 没有 `jsonReviver`/`jsonReplacer` 选项, 但您可以使用挂钩太：

```js
const gotInstance = got.extend({
	hooks: {
		init: [
			options => {
				if (options.jsonReplacer && options.json) {
					options.body = JSON.stringify(options.json, options.jsonReplacer);
					delete options.json;
				}
			}
		],
		beforeRequest: [
			options => {
				if (options.responseType === "json" && options.jsonReviver) {
					options.responseType = "text";
					options.customJsonResponse = true;
				}
			}
		],
		afterResponse: [
			response => {
				const { options } = response.request;
				if (options.jsonReviver && options.customJsonResponse) {
					response.body = JSON.parse(response.body, options.jsonReviver);
				}

				return response;
			}
		]
	}
});

gotInstance(url, options);
```

钩是强大的，不是吗？ [阅读更多](../readme.md#hooks) 就看你实现用钩子什么。

### 更多关于流

让我们快速浏览一下从`Request`的自述另一个例子:

```js
http.createServer((request, response) => {
	if (request.url === "/doodle.png") {
		request.pipe(request("https://example.com/doodle.png")).pipe(response);
	}
});
```

这里的很酷的功能是`Request`可以与流代理头，但`Got`可以做到这一点:

```js
const stream = require("stream");
const { promisify } = require("util");
const got = require("got");

const pipeline = promisify(stream.pipeline);

http.createServer(async (request, response) => {
	if (request.url === "/doodle.png") {
		// When someone makes a request to our server, we receive a body and some headers.
		// These are passed to Got. Got proxies downloaded data to our server response,
		// so you don't have to do `response.writeHead(statusCode, headers)` and `response.end(body)`.
		// It's done automatically.
		await pipeline(got.stream("https://example.com/doodle.png"), response);
	}
});
```

事情并没有改变。 只要记住使用 `got.stream(url, options)` 要么 `got(url, {isStream: true, …})`. 而已！

### 你是好去！

嗯，你已经走到这一步 :tada:
看看[文档](../readme.md#highlights).
这是值得阅读的时间。
有[一些伟大的秘诀](../readme.md#aborting-the-request).
如果有什么不清楚或不工作，因为它应该不要犹豫，打开的问题。
