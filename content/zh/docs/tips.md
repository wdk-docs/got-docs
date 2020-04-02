---
title: "提示"
linkTitle: ""
weight: 14
type: "docs"
---

## JSON 模式

传递对象为体，你需要使用`json`选项。 它将使用`JSON.stringify`进行字符串化。 例:

```js
const got = require("got");

(async () => {
	const { body } = await got.post("https://httpbin.org/anything", {
		json: {
			hello: "world"
		},
		responseType: "json"
	});

	console.log(body.data);
	//=> '{"hello":"world"}'
})();
```

要接收 JSON 体您可以将`responseType`选项设置为`json`或使用`promise.json()`. 例:

```js
const got = require("got");

(async () => {
	const body = await got
		.post("https://httpbin.org/anything", {
			json: {
				hello: "world"
			}
		})
		.json();

	console.log(body);
	//=> {…}
})();
```

## 用户代理

It's a good idea to set the `'user-agent'` header so the provider can more easily see how their resource is used. By default, it's the URL to this repo. You can omit this header by setting it to `undefined`.

```js
const got = require("got");
const pkg = require("./package.json");

got("https://sindresorhus.com", {
	headers: {
		"user-agent": `my-package/${pkg.version} (https://github.com/username/my-package)`
	}
});

got("https://sindresorhus.com", {
	headers: {
		"user-agent": undefined
	}
});
```

## 304 Responses

记住; 如果您发送的`if-modified-since`头和接收`304 Not Modified`响应，体会是空的。 这是你的责任，以高速缓存和检索正文内容.

## 自定义端点

使用`got.extend()`使其更好与 REST API 的工作. 特别是如果你使用`prefixUrl`选项.

```js
const got = require("got");
const pkg = require("./package.json");

const custom = got.extend({
	prefixUrl: "example.com",
	responseType: "json",
	headers: {
		"user-agent": `my-package/${pkg.version} (https://github.com/username/my-package)`
	}
});

// Use `custom` exactly how you use `got`
(async () => {
	const list = await custom("v1/users/list");
})();
```

## 实验 HTTP2 支持

GOT 使用[`http2-wrapper`](https://github.com/szmarczak/http2-wrapper)包提供 HTTP2 的实验支持:

```js
const got = require("got");
const { request } = require("http2-wrapper");

const h2got = got.extend({ request });

(async () => {
	const { body } = await h2got("https://nghttp2.org/httpbin/headers");
	console.log(body);
})();
```
