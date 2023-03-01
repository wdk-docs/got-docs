---
title: "基础用法-got(url?, options?)"
linkTitle: "基础用法"
weight: 1
description: >
  Got 基础用法以及各选项的解释演示
type: "docs"
---

为[`response` 对象](./response)返回一个`Promise`或如果`options.isStream`被设置为`true`返回[STREAM](./stream)。

## url

**类型:** `string` | `object`

这个请求的 URL, 可以是一个字符串, 一个[`https.request`](https://nodejs.org/api/https.html#https_https_request_options_callback)选项对象或[WHATWG `URL`](https://nodejs.org/api/url.html#url_class_url).

`options`里的属性将覆盖解析`url`里的属性 .

如果没有指定协议，它会抛出一个`TypeError`。

{{% alert color="primary" %}}
这也可以是一种选项。
{{% /alert %}}

## 选项说明

**类型:** `object`

任何[`https.request`](https://nodejs.org/api/https.html#https_https_request_options_callback)的选项。

{{% alert color="primary" %}}
传统 URL 支持被禁用。
`options.path` 为了向后兼容支持。
使用`options.pathname`和`options.searchParams`代替。
`options.auth`已被替换`options.username`＆`options.password`。
{{% /alert %}}

## prefixUrl

**类型:** `string` | `URL`

当指定，`prefixUrl`将被预置到`url`。
前缀可以是任何有效的 URL，相对或绝对的。
尾随斜线`/`是可选的 - 一个将被自动添加。

{{% alert title="" color="primary" %}}
如果`url`参数是一个 URL 实例，`prefixUrl`将被忽略.
{{% /alert %}}

{{% alert title="" color="primary" %}}
当使用此选项来执行一致性和避免混淆，在`input`里领先的斜杠是不允许的.
例如, 当前缀的网址是 `https://example.com/foo` 和 input 是 `/bar`, 有歧义产生的 URL 是否会成为`https://example.com/foo/bar` 或者 `https://example.com/bar`.
后者是由浏览器使用。
{{% /alert %}}

{{% alert title="" color="info" %}}
当用于[`got.extend()`]`创造 niche-specific Got-instances 是有用的.
{{% /alert %}}

{{% alert title="" color="info" %}}
你可以使用挂钩改变`prefixUrl`，只要 URL 仍包括了`prefixUrl`。
如果 URL 不包含它了，它会抛出。
{{% /alert %}}

```js
const got = require("got");

(async () => {
	await got("unicorn", { prefixUrl: "https://cats.com" });
	//=> 'https://cats.com/unicorn'

	const instance = got.extend({
		prefixUrl: "https://google.com"
	});

	await instance("unicorn", {
		hooks: {
			beforeRequest: [
				options => {
					options.prefixUrl = "https://cats.com";
				}
			]
		}
	});
	//=> 'https://cats.com/unicorn'
})();
```

## headers

**类型:** `object`\
**默认:** `{}`

请求头。

现有的标头将被覆盖。 设为`undefined`头将被省略。

## isStream

**类型:** `boolean`\
**默认:** `false`

返回`Stream`而不是`Promise`。 这等同于调用了`got.stream(url, options?)`.

## body

**类型:** `string` | `Buffer` | `stream.Readable` 或者 [`form-data`](https://github.com/form-data/form-data) 实例

{{% alert color="primary" %}}

1. 该`body`选项不能用`json`或`form`选项一起使用。
2. 如果你提供这个选项，`got.stream()`将是只读的。
3. 如果你提供带`GET`或`HEAD`方法`payload`, 它会抛出一个`TypeError`除非该方法`GET`且`allowGetBody`选项设置为`true`。
4. 此选项是不可枚举，不会与实例的默认值合并。

{{% /alert %}}

如果`body`是 `string` / `Buffer` / `fs.createReadStream` 实例 / [`form-data` 实例](https://github.com/form-data/form-data) `content-length`在头里将被自动设置 , 且在`options.headers`里`content-length`和`transfer-encoding`没有被手动设置.

## json

**类型:** `object` | `Array` | `number` | `string` | `boolean` | `null` _(JSON-serializable values)_

{{% alert color="primary" %}}

1. 如果你提供这个选项，`got.stream()`将是只读的。
2. 此选项是不可枚举，不会与实例的默认值合并。

{{% /alert %}}

JSON body. 如果`Content-Type`头未设置, 它将被设置为`application/json`.

## context

**类型:** `object`

用户数据。 相对于其他选项，`context`是不可枚举。

{{% alert color="primary" %}}
该对象永远不会合并，它只是通过。拿到不会修改对象以任何方式。
{{% /alert %}}

这是非常有用的，用于存储身份验证令牌:

```js
const got = require("got");

const instance = got.extend({
	hooks: {
		beforeRequest: [
			options => {
				if (!options.context || !options.context.token) {
					throw new Error("Token required");
				}

				options.headers.token = options.context.token;
			}
		]
	}
});

(async () => {
	const context = {
		token: "secret"
	};

	const response = await instance("https://httpbin.org/headers", { context });

	// Let's see the headers
	console.log(response.body);
})();
```

## responseType

**类型:** `string`\
**默认:** `'text'`

{{% alert color="primary" %}}
当使用流，此选项将被忽略。
{{% /alert %}}

解析方法。可以是 `'text'`, `'json'` 或 `'buffer'`.

`promise` 也有 `.text()`, `.json()` 和 `.buffer()` 方法, 它返回另一个`Got promise`用于解析主体.\
这就像选项设置为 `{responseType: 'json', resolveBodyOnly: true}`但不会影响主`Got promise`.

举例:

```js
(async () => {
	const responsePromise = got(url);
	const bufferPromise = responsePromise.buffer();
	const jsonPromise = responsePromise.json();

	const [response, buffer, json] = Promise.all([
		responsePromise,
		bufferPromise,
		jsonPromise
	]);
	// `response` is an instance of Got Response
	// `buffer` is an instance of Buffer
	// `json` is an object
})();
```

```js
// 这个
const body = await got(url).json();

// 于这个在语义上是相同的
const body = await got(url, { responseType: "json", resolveBodyOnly: true });
```

## resolveBodyOnly

**类型:** `boolean`\
**默认:** `false`

当设置为`true`,`promise`将返回[Response body](./response#body)而不是[Response](./response)对象。

## ignoreInvalidCookies

**类型:** `boolean`\
**默认:** `false`

忽略无效的`Cookies`，而不是抛出一个错误。 唯一有用的当`cookieJar`选项设置。 不建议。

## encoding

**类型:** `string`\
**默认:** `'utf8'`

[Encoding](https://nodejs.org/api/buffer.html#buffer_buffers_and_character_encodings) 要在响应数据的`setEncoding`使用。

为了得到一个[`Buffer`](https://nodejs.org/api/buffer.html), 你需要设置[`responseType`](#responseType) 为 `buffer` 代替.

## form

**类型:** `object` | `true`

参见[form](./form)

## searchParams

**类型:** `string` | `object<string, string | number>` | `URLSearchParams`

将被添加到请求的 URL 查询字符串。 这将覆盖`url`查询字符串。

如果你需要在一个数组来传递，你可以使用`URLSearchParams`情况下做到这一点:

```js
const got = require("got");

const searchParams = new URLSearchParams([
	["key", "a"],
	["key", "b"]
]);

got("https://example.com", { searchParams });

console.log(searchParams.toString());
//=> 'key=a&key=b'
```

And if you need a different array format, you could use the [`query-string`](https://github.com/sindresorhus/query-string) package:

```js
const got = require("got");
const queryString = require("query-string");

const searchParams = queryString.stringify(
	{ key: ["a", "b"] },
	{ arrayFormat: "bracket" }
);

got("https://example.com", { searchParams });

console.log(searchParams);
//=> 'key[]=a&key[]=b'
```

## timeout

**类型:** `number` | `object`

Milliseconds to wait for the server to end the response before aborting the request with [`got.TimeoutError`](#gottimeouterror) error (a.k.a. `request` property). By default, there's no timeout.

This also accepts an `object` with the following fields to constrain the duration of each phase of the request lifecycle:

- `lookup` starts when a socket is assigned and ends when the hostname has been resolved. Does not apply when using a Unix domain socket.
- `connect` starts when `lookup` completes (or when the socket is assigned if lookup does not apply to the request) and ends when the socket is connected.
- `secureConnect` starts when `connect` completes and ends when the handshaking process completes (HTTPS only).
- `socket` starts when the socket is connected. See [request.setTimeout](https://nodejs.org/api/http.html#http_request_settimeout_timeout_callback).
- `response` starts when the request has been written to the socket and ends when the response headers are received.
- `send` starts when the socket is connected and ends with the request has been written to the socket.
- `request` starts when the request is initiated and ends when the response's end event fires.

## retry

**类型:** `number` | `object`\
**默认:**

- limit: `2`
- calculateDelay: `({attemptCount, retryOptions, error, computedValue}) => computedValue`
- methods: `GET` `PUT` `HEAD` `DELETE` `OPTIONS` `TRACE`
- statusCodes: [`408`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408) [`413`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/413) [`429`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) [`500`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500) [`502`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/502) [`503`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/503) [`504`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/504) [`521`](https://support.cloudflare.com/hc/en-us/articles/115003011431#521error) [`522`](https://support.cloudflare.com/hc/en-us/articles/115003011431#522error) [`524`](https://support.cloudflare.com/hc/en-us/articles/115003011431#524error)
- maxRetryAfter: `undefined`
- errorCodes: `ETIMEDOUT` `ECONNRESET` `EADDRINUSE` `ECONNREFUSED` `EPIPE` `ENOTFOUND` `ENETUNREACH` `EAI_AGAIN`

An object representing `limit`, `calculateDelay`, `methods`, `statusCodes`, `maxRetryAfter` and `errorCodes` fields for maximum retry count, retry handler, allowed methods, allowed status codes, maximum [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) time and allowed error codes.

{{% alert color="primary" %}}
When using streams, this option is ignored. If the connection is reset when downloading, you need to catch the error and clear the file you were writing into to prevent duplicated content.
{{% /alert %}}
If `maxRetryAfter` is set to `undefined`, it will use `options.timeout`.\
If [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) header is greater than `maxRetryAfter`, it will cancel the request.

Delays between retries counts with function `1000 * Math.pow(2, retry) + Math.random() * 100`, where `retry` is attempt number (starts from 1).

The `calculateDelay` property is a `function` that receives an object with `attemptCount`, `retryOptions`, `error` and `computedValue` properties for current retry count, the retry options, error and default computed value. The function must return a delay in milliseconds (`0` return value cancels retry).

By default, it retries _only_ on the specified methods, status codes, and on these network errors:

- `ETIMEDOUT`: One of the [timeout](#timeout) limits were reached.
- `ECONNRESET`: Connection was forcibly closed by a peer.
- `EADDRINUSE`: Could not bind to any free port.
- `ECONNREFUSED`: Connection was refused by the server.
- `EPIPE`: The remote side of the stream being written has been closed.
- `ENOTFOUND`: Couldn't resolve the hostname to an IP address.
- `ENETUNREACH`: No internet connection.
- `EAI_AGAIN`: DNS lookup timed out.

## followRedirect

**类型:** `boolean`\
**默认:** `true`

如果重定向响应定义应该自动执行。

Note that if a `303` is sent by the server in response to any request type (`POST`, `DELETE`, etc.), Got will automatically request the resource pointed to in the location header via `GET`. This is in accordance with [the spec](https://tools.ietf.org/html/rfc7231#section-6.4.4).

## methodRewriting

**类型:** `boolean`\
**默认:** `true`

默认情况下，重定向将使用[方法重写](https://tools.ietf.org/html/rfc7231#section-6.4). 例如，发送 POST 请求和接收`302`时，将使用相同的 HTTP 方法(本例是`POST`)重新发送所述主体到新的位置.

## allowGetBody

**类型:** `boolean`\
**默认:** `false`

{{% alert color="primary" %}}
The [RFC 7321](https://tools.ietf.org/html/rfc7231#section-4.3.1) doesn't specify any particular behavior for the GET method having a payload, therefore **it's considered an [anti-pattern](https://en.wikipedia.org/wiki/Anti-pattern)**.
{{% /alert %}}

Set this to `true` to allow sending body for the `GET` method. However, the [HTTP/2 specification](https://tools.ietf.org/html/rfc7540#section-8.1.3) says that `An HTTP GET request includes request header fields and no payload body`, therefore when using the HTTP/2 protocol this option will have no effect. This option is only meant to interact with non-compliant servers when you have no other choice.

## maxRedirects

**类型:** `number`\
**默认:** `10`

如果超过了，请求将被中止，而`MaxRedirectsError`将被抛出。

## decompress

**类型:** `boolean`\
**默认:** `true`

自动解压缩的响应。
This will set the `accept-encoding` header to `gzip, deflate, br` on Node.js 11.7.0+ or `gzip, deflate` for older Node.js versions, unless you set it yourself.

Brotli (`br`) support requires Node.js 11.7.0 or later.

If this is disabled, a compressed response is returned as a `Buffer`. This may be useful if you want to handle decompression yourself or stream the raw compressed data.

## cache

**类型:** `object`\
**默认:** `false`

[缓存适配器实例](./cache) 用于存储缓存的响应数据。

## dnsCache

**类型:** `object`\
**默认:** `false`

[缓存适配器实例](./cache)) 用于存储缓存的 DNS 数据。

## request

**类型:** `Function`\
**默认:** `http.request | https.request` _(根据协议)_

自定义请求的功能。 这样做的主要目的是[支持 HTTP2 使用包装](#experimental-http2-support).

## useElectronNet

**类型:** `boolean`\
**默认:** `false`

[**作废**](https://github.com/sindresorhus/got#electron-support-has-been-deprecated)

When used in Electron, Got will use [`electron.net`](https://electronjs.org/docs/api/net/) instead of the Node.js `http` module. According to the Electron docs, it should be fully compatible, but it's not entirely. See [#443](https://github.com/sindresorhus/got/issues/443) and [#461](https://github.com/sindresorhus/got/issues/461).

## throwHttpErrors

**类型:** `boolean`\
**默认:** `true`

确定一个`got.HTTPError`被抛出的错误响应（非 2xx 状态码）。

如果这是禁用，遇到的错误状态代码的请求将返回`response`而不是抛出来解决。
如果你正在检查资源的可用性和期待的错误响应，这可能是有用的。

参加[error](./error)

## agent

参加[agent](./agent)

## hooks

参加[hooks](./hooks)

## pagination

分页参加[pagination](./pagination)

## cookie

cookie 参加[cookie](./cookie)
