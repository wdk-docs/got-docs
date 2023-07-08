# 选项



源码: [`source/core/options.ts`](../source/core/options.ts)

就像 `fetch` 在 `Request` 实例中存储选项一样，Got 在 `options` 实例中存储选项。
它由 `getter` 和 `setter` 组成，提供快速的选项规范化和验证。

**默认情况下，Got 将在失败时重试。若要禁用此选项，请执行以下操作, 设置 [`options.retry`](7-retry.md) 为 `{limit: 0}`.**

## 合并行为解释

当已经设置了一个选项时，在默认情况下，再次设置它将用深度克隆替换它。
否则，合并行为将在选项的相应部分中进行记录。

## 如何存储选项

构造函数 - `new Options(url, options, defaults)` - 接受与' got '函数相同的参数。

```js
import got, { Options } from "got";

const options = new Options({
  prefixUrl: "https://httpbin.org",
  headers: {
    foo: "foo",
  },
});

options.headers.foo = "bar";

// Note that `Options` stores normalized options, therefore it needs to be passed as the third argument.
const { headers } = await got("anything", undefined, options).json();
console.log(headers.Foo);
//=> 'bar'
```

如果首选一个普通对象，它可以以以下方式使用:

```js
import got from "got";

const options = {
  prefixUrl: "https://httpbin.org",
  headers: {
    foo: "bar",
  },
};

options.headers.foo = "bar";

// Note that `options` is a plain object, therefore it needs to be passed as the second argument.
const { headers } = await got("anything", options).json();
console.log(headers.Foo);
//=> 'bar'
```

注意，当提供了无效的选项时，构造函数会抛出，比如不存在的选项或输入错误。
在第二个例子中，它只在执行承诺时抛出。

对于 TypeScript 用户，' got '导出一个名为' OptionsInit '的专用类型。
它是一个普通对象，可以存储与“Options”相同的属性。

在性能方面，使用哪一个没有区别，尽管构造函数可能是首选的，因为它会自动验证数据。
`Options`方法可能会有轻微的提升，因为它只是克隆选项，没有标准化。
它对于存储自定义 Got 客户端的基本配置也很有用。

## 重置选项

与 Got 11 不同，显式指定' undefined '不再保留父值。
为了保持父值，你不能将一个选项设置为' undefined '。
这样做将重置这些值:

```js
instance(…, {searchParams: undefined}});
instance(…, {cookieJar: undefined}});
instance(…, {responseType: undefined}});
instance(…, {prefixUrl: ''});
instance(…, {agent: {http: undefined, https: undefined, http2: undefined}});
instance(…, {context: {token: undefined, …}});
instance(…, {https: {rejectUnauthorized: undefined, …}});
instance(…, {cacheOptions: {immutableMinTimeToLive: undefined, …}});
instance(…, {headers: {'user-agent': undefined, …}});
instance(…, {timeout: {request: undefined, …}});
```

为了重置' hooks '， ' retry '和' pagination '，必须创建另一个 Got 实例:

```js
const defaults = new Options();

const secondInstance = instance.extend({ mutableDefaults: true });
secondInstance.defaults.options.hooks = defaults.hooks;
secondInstance.defaults.options.retry = defaults.retry;
secondInstance.defaults.options.pagination = defaults.pagination;
```
## 属性

### `url`

**类型: <code>string | [URL](https://nodejs.org/api/url.html#url_the_whatwg_url_api)</code>**

要请求的 URL。通常' url '表示一个[WHATWG url](https://url.spec.whatwg.org/#url-class).

```js
import got from "got";

// This:
await got("https://httpbin.org/anything");

// is semantically the same as this:
await got(new URL("https://httpbin.org/anything"));

// as well as this:
await got({
  url: "https://httpbin.org/anything",
});
```

!!! Note

    如果没有指定协议则抛出。

!!! Note

    如果' url '是一个字符串，那么' query '字符串将不会被解析为搜索参数。
    这符合[规范](https://datatracker.ietf.org/doc/html/rfc7230#section-2.7)。
    如果你想传递搜索参数，使用下面的' searchParams '选项。

```js
import got from "got";

await got("https://httpbin.org/anything?query=a b"); //=> ?query=a%20b
await got("https://httpbin.org/anything", { searchParams: { query: "a b" } }); //=> ?query=a+b

// The query string is overridden by `searchParams`
await got("https://httpbin.org/anything?query=a b", {
  searchParams: { query: "a b" },
}); //=> ?query=a+b
```

!!! Note

    不允许使用前导斜杠，以加强一致性并避免混淆。
    例如，当前缀URL是' https://example.com/foo '，输入是' /bar '时，
    结果URL会变成' https://example.com/foo/bar '还是' https://example.com/bar '是不明确的。
    后者由浏览器使用。

### `searchParams`

**类型: <code>string | [URLSearchParams](https://nodejs.org/api/url.html#url_class_urlsearchparams) | object&lt;string, [Primitive](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)&gt;</code>**

[WHATWG URL 搜索参数](https://url.spec.whatwg.org/#interface-urlsearchparams)要添加到请求 URL 中。

```js
import got from "got";

const response = await got("https://httpbin.org/anything", {
  searchParams: {
    hello: "world",
    foo: 123,
  },
}).json();

console.log(response.args);
//=> {hello: 'world', foo: 123}
```

如果你需要传递一个数组，你可以使用' URLSearchParams '实例:

```js
import got from "got";

const searchParams = new URLSearchParams([
  ["key", "a"],
  ["key", "b"],
]);

await got("https://httpbin.org/anything", { searchParams });

console.log(searchParams.toString());
//=> 'key=a&key=b'
```

!!! Note

    这将覆盖' url '中的' query '字符串。

!!! Note

    - ' null '值不进行字符串化，而是使用空字符串。
    - ' undefined '值将清除原始键。

!!! note "合并行为"

    - 覆盖现有属性。

### `prefixUrl`

**类型: `string`**  
**默认: `''`**

要加在 `url` 前面的字符串。

前缀可以是任何有效的 URL，无论是相对 URL 还是[绝对 URL](https://url.spec.whatwg.org/#absolute-url-string)。
后面的斜杠 `/` 是可选的，会自动添加。

```js
import got from "got";

// This:
const instance = got.extend({ prefixUrl: "https://httpbin.org" });
await instance("anything");

// is semantically the same as this:
await got("https://httpbin.org/anything");
```

!!! Note

    更改' prefixUrl '也更新' url '选项如果设置。

!!! Note

    如果你传递一个绝对URL作为' URL '，你需要设置' prefixUrl '为一个空字符串。

### `signal`

**类型: [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)**

你可以使用[`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)来中止请求.

_需要Node.js 16或更高版本._

```js
import got from "got";

const abortController = new AbortController();

const request = got("https://httpbin.org/anything", {
  signal: abortController.signal,
});

setTimeout(() => {
  abortController.abort();
}, 100);
```

### `method`

**类型: `string`**  
**默认: `GET`**

用于发出请求的[HTTP方法](https://datatracker.ietf.org/doc/html/rfc7231#section-4)。 
最常见的方法是: `GET`, `HEAD`, `POST`, `PUT`, `DELETE`.

```js
import got from "got";

const { method } = await got("https://httpbin.org/anything", {
  method: "POST",
}).json();

console.log(method);
// => 'POST'
```

### `headers`

**类型: `object<string, string>`**  
**默认: `{}`**

要发送的[HTTP头](https://datatracker.ietf.org/doc/html/rfc7231#section-8.3)。设置为`undefined`的标头将被省略。

```js
import got from "got";

const { headers } = await got
  .post("https://httpbin.org/anything", {
    headers: {
      hello: "world",
    },
  })
  .json();

console.log(headers);
// => {hello: 'world'}
```

!!! note "合并行为"

    覆盖现有属性。

### `isStream`

**类型: `boolean`**  
**默认: `false`**

`got`函数是否应该返回[`Request`双工流](3-streams.md)或[`Promise<Response>`](1-promise.md)。

```js
import got from "got";

// This:
const stream = got("https://httpbin.org/anything", { isStream: true });

// is semantically the same as this:
const stream = got.stream("https://httpbin.org/anything");

stream.setEncoding("utf8");
stream.on("data", console.log);
```

### `body`

**类型: `string | Buffer | stream.Readable | Generator | AsyncGenerator | FormData` or [`form-data` instance](https://github.com/form-data/form-data)**

要发送的有效载荷。

对于 `string` 和 `Buffer` 类型，如果 `content-length` 和 `transfer-encoding` 头缺失， `content-length` 头将自动设置。

**从 Got 12 开始，当 `body` 是[ `fs.createReadStream()` ](https://nodejs.org/api/fs.html#fs_fs_createreadstream_path_options)的实例时， `content-length` 头不会自动设置。.**

```js
import got from "got";

const { data } = await got
  .post("https://httpbin.org/anything", {
    body: "Hello, world!",
  })
  .json();

console.log(data);
//=> 'Hello, world!'
```

从 Got 12 开始，您可以使用符合规范的[ `FormData` ](https://developer.mozilla.org/en-US/docs/Web/API/FormData)对象作为请求体，例如[`formdata-node`](https://github.com/octet-stream/form-data)或[`formdata-polyfill`](https://github.com/jimmywarting/FormData):

```js
import got from "got";
import { FormData } from "formdata-node"; // or:
// import {FormData} from 'formdata-polyfill/esm.min.js';

const form = new FormData();
form.set("greeting", "Hello, world!");

const data = await got
  .post("https://httpbin.org/post", {
    body: form,
  })
  .json();

console.log(data.form.greeting);
//=> 'Hello, world!'
```

!!! Note

    如果指定了 `body` ，则不能使用 `json` 或 `form` 选项。

!!! Note

    如果使用此选项， `got.stream()` 将是只读的。

!!! Note

    除非[ `allowGetBody` 选项](#allowGetBody)设置为 `true` ，否则用 `GET` 传递 `body` 将抛出。

!!! Note

    此选项不可枚举，并且不会与实例默认值合并。

### `json`

**类型: JSON-serializable values**

JSON body. 如果设置， `content-type` 头默认为 `application/json` 。

```js
import got from "got";

const { data } = await got
  .post("https://httpbin.org/anything", {
    json: {
      hello: "world",
    },
  })
  .json();

console.log(data);
//=> `{hello: 'world'}`
```

### `form`

**类型: <code>object&lt;string, [Primitive](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)&gt;</code>**

使用 `(new URLSearchParams(form)).toString()` 将表单主体转换为查询字符串.

如果设置了， `content-type` 头默认为[ `application/x-www-form-urlencoded` ](https://url.spec.whatwg.org/#application/x-www-form-urlencoded).

```js
import got from "got";

const { data } = await got
  .post("https://httpbin.org/anything", {
    form: {
      hello: "world",
    },
  })
  .json();

console.log(data);
//=> 'hello=world'
```

### `parseJson`

**类型: `(text: string) => unknown`**  
**默认: `(text: string) => JSON.parse(text)`**

用于解析 JSON 响应的函数。

```js
import got from "got";
import Bourne from "@hapi/bourne";

// Preventing prototype pollution by using Bourne
const parsed = await got("https://example.com", {
  parseJson: (text) => Bourne.parse(text),
}).json();

console.log(parsed);
```

### `stringifyJson`

**类型: `(object: unknown) => string`**  
**默认: `(object: unknown) => JSON.stringify(object)`**

用于对 JSON 请求体进行字符串化的函数。

**例如:忽略所有以下划线开头的属性**

```js
import got from "got";

await got.post("https://example.com", {
  stringifyJson: (object) =>
    JSON.stringify(object, (key, value) => {
      if (key.startsWith("_")) {
        return;
      }

      return value;
    }),
  json: {
    some: "payload",
    _ignoreMe: 1234,
  },
});
```

**例如:所有数字都是字符串**

```js
import got from "got";

await got.post("https://example.com", {
  stringifyJson: (object) =>
    JSON.stringify(object, (key, value) => {
      if (typeof value === "number") {
        return value.toString();
      }

      return value;
    }),
  json: {
    some: "payload",
    number: 1,
  },
});
```

### `allowGetBody`

**类型: `boolean`**  
**默认: `false`**

将此设置为 `true` 以允许为 `GET` 方法发送正文。

然而，[HTTP/2 规范](https://datatracker.ietf.org/doc/html/rfc7540#section-8.1.3)说:

> HTTP GET 请求包括请求头字段，没有有效负载主体

因此，该选项在使用 HTTP/2 时无效。

!!! Note

    此选项仅用于在没有其他选择时与不兼容的服务器交互。

!!! Note

    [RFC 7231](https://tools.ietf.org/html/rfc7231#section-4.3.1)没有为具有有效负载的GET方法指定任何特定行为，因此它被认为是一个[**反模式**](https://en.wikipedia.org/wiki/Anti-pattern)。

### `timeout`

**类型: `object`**

See the [Timeout API](6-timeout.md).

!!! note "合并行为"

    覆盖现有属性。

### `retry`

**类型: `object`**

See the [Retry API](7-retry.md).

!!! note "合并行为"

    覆盖现有属性。

### `hooks`

**类型: `object`**

See the [Hooks API](9-hooks.md).

!!! note "合并行为"

    Merges arrays via `[...hooksArray, ...next]`

### `encoding`

**类型: `string`**  
**默认: `'utf8'`**

[Encoding](https://nodejs.org/api/buffer.html#buffer_buffers_and_character_encodings) to be used on [`setEncoding`](https://nodejs.org/api/stream.html#stream_readable_setencoding_encoding) of the response data.

To get a [`Buffer`](https://nodejs.org/api/buffer.html), you need to set `responseType` to `'buffer'` instead. Don't set this option to `null`.

```js
import got from "got";

const response = await got("https://httpbin.org/anything", {
  encoding: "base64",
}).text();

console.log(response);
//=> base64 string
```

!!! Note

    This option does not affect streams! Instead, do:

```js
import got from "got";

const stream = got.stream("https://httpbin.org/anything");

stream.setEncoding("base64");
stream.on("data", console.log);
```

### `responseType`

**类型: `'text' | 'json' | 'buffer'`**  
**默认: `'text'`**

The parsing method.

The promise also has `.text()`, `.json()` and `.buffer()` methods which return another Got promise for the parsed body.  
It's like setting the options to `{responseType: 'json', resolveBodyOnly: true}` but without affecting the main Got promise.

```js
import got from "got";

const responsePromise = got("https://httpbin.org/anything");
const bufferPromise = responsePromise.buffer();
const jsonPromise = responsePromise.json();

const [response, buffer, json] = await Promise.all([
  responsePromise,
  bufferPromise,
  jsonPromise,
]);
// `response` is an instance of Got Response
// `buffer` is an instance of Buffer
// `json` is an object
```

!!! Note

    When using streams, this option is ignored.

!!! Note

    `'buffer'` will return the raw body buffer. Any modifications will also alter the result of `.text()` and `.json()`. Before overwriting the buffer, please copy it first via `Buffer.from(buffer)`.

> See https://github.com/nodejs/node/issues/27080

### `resolveBodyOnly`

**类型: `boolean`**  
**默认: `false`**

If `true`, the promise will return the [Response body](3-streams.md#response-1) instead of the [Response object](3-streams.md#response-1).

```js
import got from "got";

const url = "https://httpbin.org/anything";

// This:
const body = await got(url).json();

// is semantically the same as this:
const body = await got(url, { responseType: "json", resolveBodyOnly: true });
```

### `context`

**类型: `object<string, unknown>`**  
**默认: `{}`**

!!! Note

    内部的不可枚举属性**没有**合并。

包含用户数据。它对于存储认证令牌非常有用:

```js
import got from "got";

const instance = got.extend({
  hooks: {
    beforeRequest: [
      (options) => {
        if (typeof options.context.token !== "string") {
          throw new Error("Token required");
        }

        options.headers.token = options.context.token;
      },
    ],
  },
});

const context = {
  token: "secret",
};

const { headers } = await instance("https://httpbin.org/headers", {
  context,
}).json();

console.log(headers);
//=> {token: 'secret', …}
```

这个选项是可枚举的。为了在内部定义不可枚举的属性，执行以下操作:

```js
import got from "got";

const context = {};

Object.defineProperties(context, {
  token: {
    value: "secret",
    enumerable: false,
    configurable: true,
    writable: true,
  },
});

const instance = got.extend({ context });

console.log(instance.defaults.options.context);
//=> {}
```

!!! note "合并行为"

    覆盖现有属性。

### `cookieJar`

**类型: <code>object | [tough.cookieJar](https://github.com/salesforce/tough-cookie#cookiejar)</code>**

!!! Note

    设置此选项将导致 `cookie` 报头被覆盖。

Cookie 的支持。自动处理解析和存储。

```js
import got from "got";
import { CookieJar } from "tough-cookie";

const cookieJar = new CookieJar();

await cookieJar.setCookie("foo=bar", "https://example.com");
await got("https://example.com", { cookieJar });
```

### `cookieJar.setCookie`

**类型: `(rawCookie: string, url: string) => void | Promise<void>`**

See [ToughCookie API](https://github.com/salesforce/tough-cookie#setcookiecookieorstring-currenturl-options-cberrcookie) for more information.

### `cookieJar.getCookieString`

**类型: `(currentUrl: string) => string | Promise<string>`**

参见 [ToughCookie API](https://github.com/salesforce/tough-cookie#getcookiestring) 了解更多信息。

### `ignoreInvalidCookies`

**类型: `boolean`**  
**默认: `false`**

忽略无效的cookies，而不是抛出错误。
仅在设置了`cookieJar`选项时有用。

!!! Note

    这是不推荐的!使用风险自负。

### `followRedirect`

**类型: `boolean`**  
**默认: `true`**

定义是否应该自动跟随重定向响应。

!!! Note

    如果服务器在响应任何请求类型(POST, DELETE等)时发送了`303`，Got将通过GET请求指向位置头中的资源。
    这符合[规范](https://tools.ietf.org/html/rfc7231#section-6.4.4)。
    你也可以选择为其他重定向代码打开这个行为-参见[`methodRewriting`](#methodrewriting)

```js
import got from "got";

const instance = got.extend({ followRedirect: false });

const response = await instance("http://google.com");

console.log(response.headers.location);
//=> 'https://google.com'
```

### `maxRedirects`

**类型: `number`**  
**默认: `10`**

如果超出，请求将被中止，并抛出[`MaxRedirectsError`](8-errors.md#maxredirectserror)。

```js
import got from "got";

const instance = got.extend({ maxRedirects: 3 });

try {
  await instance("https://nghttp2.org/httpbin/absolute-redirect/5");
} catch (error) {
  //=> 'Redirected 3 times. Aborting.'
  console.log(error.message);
}
```

### `decompress`

**类型: `boolean`**  
**默认: `true`**

自动解压缩响应。这将设置`accept-encoding`头为`gzip, deflate, br`。

如果禁用，则作为`Buffer`返回压缩响应。
如果您想自己处理解压，这可能很有用。

```js
import got from "got";

const response = await got("https://google.com");

console.log(response.headers["content-encoding"]);
//=> 'gzip'
```

### `dnsLookup`

**类型: `Function`**  
**默认: [`dns.lookup`](https://nodejs.org/api/dns.html#dns_dns_lookup_hostname_options_callback)**

自定义DNS解析逻辑。

函数签名与`dns.lookup`相同。

### `dnsCache`

**类型: <code>[CacheableLookup](https://github.com/szmarczak/cacheable-lookup) | false</code>**

一个用于进行DNS查找的 `CacheableLookup` 实例。
在向不同的公共主机名发出大量请求时非常有用。

!!! note

    - 当向内部主机名(如localhost、databasel.local等)发出请求时，该功能应该保持禁用状态。
    - CacheableLookup在后台使用 `dns.resolver4(…)`和`dns.resolver6(…)`，当前两个失败时返回到`dns.lookup(…)`，这可能会导致额外的延迟。
  
### `dnsLookupIpVersion`

**类型: `4 | 6`**  
**默认: `undefined`**

要使用的IP版本。指定`undefined` 将使用默认配置。

### `request`

**类型: <code>Function<[ClientRequest](https://nodejs.org/api/http.html#http_class_http_clientrequest) | [IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage)> | AsyncFunction<[ClientRequest](https://nodejs.org/api/http.html#http_class_http_clientrequest) | [IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage)></code>**  
**默认: `http.request | https.request` _(depending on the protocol)_**

自定义请求函数。

这样做的主要目的是[使用包装器支持HTTP/2](https://github.com/szmarczak/http2-wrapper).

### `cache`

**类型: `object | false`**  
**默认: `false`**

[缓存适配器实例](cache.md)用于存储缓存的响应数据。

### `cacheOptions`

**类型: `object`**  
**默认: `{}`**

[缓存选项](https://github.com/kornelski/http-cache-semantics#constructor-options)用于指定请求。

### `http2`

**类型: `boolean`**  
**默认: `false`**

!!! note

    此选项需要Node.js 15.10.0或更新版本，因为旧版本的Node.js对HTTP/2的支持非常错误。

如果`true`， `request`选项将默认为`http2wrapper.auto`，整个`agent`对象将被传递。

!!! note

    为了确定服务器是否真正支持HTTP/2，将进行ALPN协商。如果没有，则使用HTTP/1.1。

!!! note

    将`request` 选项设置为`https.request`将禁用HTTP/2的使用。它需要使用`http2wrapper.auto`。

!!! note

    没有直接的[`h2c`](https://datatracker.ietf.org/doc/html/rfc7540#section-3.1)支持。
    然而，你可以在`beforeRequest`钩子中提供一个`h2session`选项。
    参见[例子](examples/h2c.js)。

```js
import got from "got";

const { headers } = await got("https://httpbin.org/anything", {
  http2: true,
});

console.log(headers[":status"]);
//=> 200
```

!!! note

    当前Got版本可能使用旧版本的[`http2-wrapper`](https://github.com/szmarczak/http2-wrapper)。
    如果你想用最新的，把`request`设置为`http2wrapper.auto`，把`http2`设置为`true`。

```js
import http2wrapper from "http2-wrapper";
import got from "got";

const { headers } = await got("https://httpbin.org/anything", {
  http2: true,
  request: http2wrapper.auto,
});

console.log(headers[":status"]);
//=> 200
```

请参阅[`http2-wrapper` 文档](https://github.com/szmarczak/http2-wrapper)了解有关代理和代理支持的更多信息。

### `agent`

**类型: `object`**  
**默认: `{}`**

一个具有`http`, `https` 和 `http2`属性的对象。

Got将自动解析协议并使用相应的代理。默认值为:

```js
{
	http: http.globalAgent,
	https: https.globalAgent,
	http2: http2.globalAgent
}
```

!!! note

    HTTP/2`Agent`必须是 [`http2wrapper.Agent`](https://github.com/szmarczak/http2-wrapper#new-http2agentoptions)的实例

### `throwHttpErrors`

**类型: `boolean`**  
**默认: `true`**

如果`true`，当状态码不是`2xx` / `3xx`时，它将[抛出](8-errors.md#httperror)。

如果禁用此功能，则遇到错误状态码的请求将通过响应而不是抛出来解决。
如果您正在检查资源可用性并期望得到错误响应，这可能很有用。

### `username`

**类型: `string`**  
**默认: `''`**

用于[基本身份验证](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)的`username`.

### `password`

**类型: `string`**  
**默认: `''`**

用于[基本身份验证](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)的`password`.

### `localAddress`

**类型: `string | undefined`**  
**默认: `undefined`**

发出请求的本地IP地址。

### `createConnection`

**类型: `Function | undefined`**  
**默认: `undefined`**

当未使用 `agent` 选项时，用于检索`net.Socket`实例的函数。

### `https`

**类型: `object`**

参见[高级HTTPS API](5-https.md).

### `pagination`

**类型: `object`**

参见[分页API](4-pagination.md).

### `setHost`

**类型: `boolean`**  
**默认: `true`**

指定是否自动添加 `Host` 标头。

### `maxHeaderSize`

**类型: `number | undefined`**  
**默认: `undefined`**

可选地覆盖[`--max-http-header-size`](https://nodejs.org/api/cli.html#cli_max_http_header_size_size)的值(默认 16KB: `16384`).

### `methodRewriting`

**类型: `boolean`**  
**默认: `false`**

指定HTTP请求方法在重定向时是否应该[重写为`GET`](https://tools.ietf.org/html/rfc7231#section-6.4)。

由于[规范](https://tools.ietf.org/html/rfc7231#section-6.4)倾向于只在`303`响应上重写HTTP方法，这是Got的默认行为。
将`methodRewriting`设置为`true`也会重写`301`和`302`响应，正如规范所允许的那样。
这是 `curl` 和浏览器所遵循的行为。

!!! note

    Got从不对`307` 和 `308`响应执行方法重写，因为这是[规范明确禁止的](https://www.rfc-editor.org/rfc/rfc7231#section-6.4.7).

### `enableUnixSockets`

**类型: `boolean`**  
**默认: `true`**

当启用时，请求也可以通过[UNIX域套接字](https://serverfault.com/questions/124517/what-is-the-difference-between-unix-sockets-and-tcp-ip-sockets)发送。
请注意，在即将到来的主要版本(Got v13)中，出于安全原因，此默认值将更改为`false` 。

!!! **Warning**

    如果接受不受信任的用户输入URL，请确保执行自己的URL消毒。

使用以下URL方案: `PROTOCOL://unix:SOCKET:PATH`

- `PROTOCOL` - `http` 或 `https`
- `SOCKET` - 例如，UNIX域套接字的绝对路径: `/var/run/docker.sock`
- `PATH` - 例如，请求路径: `/v2/keys`

```js
import got from "got";

await got("http://unix:/var/run/docker.sock:/containers/json", {
  enableUnixSockets: true,
});

// Or without protocol (HTTP by default)
await got("unix:/var/run/docker.sock:/containers/json", {
  enableUnixSockets: true,
});

// Disable Unix sockets
const gotUnixSocketsDisabled = got.extend({ enableUnixSockets: false });

// RequestError: Using UNIX domain sockets but option `enableUnixSockets` is not enabled
await gotUnixSocketsDisabled(
  "http://unix:/var/run/docker.sock:/containers/json"
);
```

## 方法

### `options.merge(other: Options | OptionsInit)`

将`other` 合并到当前实例中。

如果你查看[源代码](../source/core/options.ts)，你会注意到内部有一个 `this._merging` 属性。
当它为`true`时，Setters 的工作方式略有不同。

### `options.toJSON()`

返回一个新的普通对象，可以存储为[JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#tojson_behavior).

### `options.createNativeRequestOptions()`

为本地Node.js HTTP请求选项创建一个新对象。

换句话说，它将Got选项转换为Node.js选项。

!!! note

    其他一些东西，比如超时，是由Got内部处理的。

### `options.getRequestFunction()`

返回用于发出请求的 [`http.request`-like](https://nodejs.org/api/http.html#http_http_request_url_options_callback) 函数。

### `options.freeze()`

使整个 `Options` 实例只读。
