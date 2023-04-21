# Request

让我们以[Request's readme](https://github.com/request/request#super-simple-to-use)中的第一个例子为例:

```js
import request from "request";

request("https://google.com", (error, response, body) => {
  console.log("error:", error);
  console.log("statusCode:", response && response.statusCode);
  console.log("body:", body);
});
```

对于 Got，它是:

```js
import got from "got";

try {
  const response = await got("https://google.com");
  console.log("statusCode:", response.statusCode);
  console.log("body:", response.body);
} catch (error) {
  console.log("error:", error);
}
```

现在看起来好多了，是吧? 😎

## 常见的选项

这些 Got 选项与 Request 相同:

- [`url`](../2-options.md#url)
- [`body`](../2-options.md#body)
- [`followRedirect`](../2-options.md#followredirect)
- [`encoding`](../2-options.md#encoding)
- [`maxRedirects`](../2-options.md#maxredirects)
- [`localAddress`](../2-options.md#localaddress)
- [`headers`](../2-options.md#headers)
- [`createConnection`](../2-options.md#createconnection)
- [UNIX sockets](../2-options.md#enableunixsockets): `http://unix:SOCKET:PATH`

`time` 选项不存在，假设[它总是正确](../6-timeout.md).

如果你熟悉这些，就可以开始了。

## 重命名选项

!!! Note

    得到存储HTTPS选项在[`httpoptions`](../2-options.md# httpoptions)。其中一些已经被重新命名。
    [了解更多](../5-https.md).

可读性对我们来说非常重要，所以我们对这些选项有不同的名称:

- `qs` → [`searchParams`](../2-options.md#serachparams)
- `strictSSL` → [`rejectUnauthorized`](../2-options.md#rejectunauthorized)
- `gzip` → [`decompress`](../2-options.md#decompress)
- `jar` → [`cookieJar`](../2-options.md#cookiejar) (accepts [`tough-cookie`](https://github.com/salesforce/tough-cookie) jar)
- `jsonReviver` → [`parseJson`](../2-options.md#parsejson)
- `jsonReplacer` → [`stringifyJson`](../2-options.md#stringifyjson)

## 行为变化

- The [`agent` option](../2-options.md#agent) is now an object with `http`, `https` and `http2` properties.
- The [`timeout` option](../6-timeout.md) is now an object. You can set timeouts on particular events!
- The [`searchParams` option](https://github.com/sindresorhus/got#searchParams) is always serialized using [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams).
- In order to pass a custom query string, provide it with the `url` option.  
  `got('https://example.com', {searchParams: {test: ''}})` → `https://example.com/?test=`  
  `got('https://example.com/?test')` → `https://example.com/?test`
- To use streams, call `got.stream(url, options)` or `got(url, {…, isStream: true})`.

## 突发的变化

- The `json` option is not a `boolean`, it's an `object`. It will be stringified and used as a body.
- The `form` option is an `object` and will be used as `application/x-www-form-urlencoded` body.
- All headers are converted to lowercase.  
  According to [the spec](https://datatracker.ietf.org/doc/html/rfc7230#section-3.2), the headers are case-insensitive.
- No `oauth` / `hawk` / `aws` / `httpSignature` option.  
  To sign requests, you need to create a [custom instance](../examples/advanced-creation.js).
- No `agentClass` / `agentOptions` / `pool` option.
- No `forever` option.  
  You need to pass an agent with `keepAlive` option set to `true`.
- No `proxy` option. You need to [pass a custom agent](../tips.md#proxy).
- No `auth` option.  
  You need to use [`username`](../2-options.md#username) / [`password`](../2-options.md#password) instead or set the `authorization` header manually.
- No `baseUrl` option.  
  Instead, there is [`prefixUrl`](../2-options.md#prefixurl) which appends a trailing slash if not present.
- No `removeRefererHeader` option.  
  You can remove the `referer` header in a [`beforeRequest` hook](../9-hooks.md#beforerequest).
- No `followAllRedirects` option.

Hooks are very powerful. [Read more](../9-hooks.md) to see what else you achieve using hooks.

## 关于流的更多信息

让我们快速看一下 Request 自述中的另一个例子:

```js
http.createServer((serverRequest, serverResponse) => {
  if (serverRequest.url === "/doodle.png") {
    serverRequest
      .pipe(request("https://example.com/doodle.png"))
      .pipe(serverResponse);
  }
});
```

这里很酷的特性是 Request 可以用流代理报头，但 Got 也可以这样做!

```js
import { promisify } from "node:util";
import stream from "node:stream";
import got from "got";

const pipeline = promisify(stream.pipeline);

const server = http.createServer(async (serverRequest, serverResponse) => {
  if (serverRequest.url === "/doodle.png") {
    await pipeline(
      got.stream("https://example.com/doodle.png"),
      serverResponse
    );
  }
});

server.listen(8080);
```

就流而言，什么都没有真正改变。

## 方便的方法

- If you were using `request.get`, `request.post`, and so on - you can do the same with Got.
- The `request.defaults({…})` method has been renamed. You can do the same with `got.extend({…})`.
- There is no `request.cookie()` nor `request.jar()`. You have to use `tough-cookie` directly.

## 你可以开始了!

好吧，你已经走了这么远:tada:
看一下[文档](../../readme.md#documentation)。值得花时间读一读。
[这里](../tips.md)有一些很棒的建议。

如果某件事不清楚或没有按照它应该的方式运行，不要犹豫[打开一个问题](https://github.com/sindresorhus/got/issues/new/choose).
