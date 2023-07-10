# 流 API

## 构造函数

源码: [`source/core/index.ts`](./source/core/index.ts)

**`got.stream(url, options, defaults)`**

**`got(url, {...options, isStream: true}, defaults)`**

上面的两个函数由 `got` 主界面公开，并返回 `Request` 的一个新实例。

**`new Request(url, options, defaults)`**

**Extends: [`Duplex` stream](https://nodejs.org/api/stream.html#stream_class_stream_duplex)**

此构造函数接受与 Got 承诺相同的参数。

!!! note

    当连接到[ `ServerResponse` ](https://nodejs.org/api/http.html#http_class_http_serverresponse)时，头将被自动复制。
    为了防止这种行为，你需要覆盖一个[ `beforeRequest` ](9-hooks.md# beforeRequest)钩子中的请求头。

!!! note

    如果使用 `body` ，  `json` 或 `form` 选项，此流将是只读的。

!!! note

    - 当 `got.post('https://example.com')` 被解析时， `got.stream.post('https://example.com')` 将无限期挂起，直到提供正文。
    - 如果故意没有body，请记住 `stream.end()` 或将body选项设置为空字符串。

```js
import { promisify } from "node:util";
import stream from "node:stream";
import fs from "node:fs";
import got from "got";

const pipeline = promisify(stream.pipeline);

// 这个示例将URL的GET响应流式传输到文件。
await pipeline(
  got.stream("https://sindresorhus.com"),
  fs.createWriteStream("index.html")
);

// 对于POST, PUT, PATCH和DELETE方法， `got.stream` 返回一个 `stream.Writable` 。
// 这个例子将一个文件的内容post到一个URL。
await pipeline(
  fs.createReadStream("index.html"),
  got.stream.post("https://sindresorhus.com"),
  new stream.PassThrough()
);

// 为了在没有请求体的情况下POST、PUT、PATCH或DELETE，显式地指定一个空体:
await pipeline(
  got.stream.post("https://sindresorhus.com", { body: "" }),
  new stream.PassThrough()
);
```

请注意，为了捕捉读取错误， `new stream.PassThrough()` 是必需的。
如果没有，那么 `pipeline` 将不会捕获任何读取错误，因为没有流可以管道。
换句话说，它只在写入时检查错误。

!!! tip

    - 避免使用 `from.pipe(to)` ，因为它不会转发错误。

## 选项

### `stream.options`

**类型: [`Options`](2-options.md)**

用于发出请求的选项。

### `stream.response`

**类型: [`IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage)**

底层的 `IncomingMessage` 实例。

### `stream.requestUrl`

**类型: [`URL`](https://nodejs.org/api/url.html#url_the_whatwg_url_api)**

这次尝试中的当前“URL”对象。

### `stream.redirectUrls`

**类型: [`URL[]`](https://nodejs.org/api/url.html#url_the_whatwg_url_api)**

连续请求的 url 数组。

### `stream.retryCount`

**类型: `number`**

当前重试计数。

!!! note

    - 重试时必须重写。

### `stream.ip`

**类型: `string | undefined`**

目的 IP 地址。

### `stream.isAborted`

**类型: `boolean`**

请求是否被中止。

### `stream.socket`

**类型: `net.Socket | tls.Socket | undefined`**

用于此特定请求的套接字。

### `stream.downloadProgress`

**类型: [`Progress`](typescript.md#progress)**

表示已下载数据量的对象。

### `stream.uploadProgress`

**类型: [`Progress`](typescript.md#progress)**

表示已上传数据量的对象。

!!! note

    - 当一个chunk大于 `highWaterMark` 时，进度将不会被触发。体需要被分成几块。

```js
import got from "got";

const body = Buffer.alloc(1024 * 1024); // 1MB

function* chunkify(buffer, chunkSize = 64 * 1024) {
  for (let pos = 0; pos < buffer.byteLength; pos += chunkSize) {
    yield buffer.subarray(pos, pos + chunkSize);
  }
}

const stream = got.stream.post("https://httpbin.org/anything", {
  body: chunkify(body),
});

stream.resume();

stream.on("uploadProgress", (progress) =    {
  console.log(progress);
});
```

### `stream.timings`

**类型: [`Timings`](typescript.md#timings)**

An object representing performance information.

To generate the timings, Got uses the [`http-timer`](https://github.com/szmarczak/http-timer) package.

### `stream.isFromCache`

**类型: `boolean | undefined`**

Whether the response has been fetched from cache.

### `stream.reusedSocket`

**类型: `boolean`**

Whether the socket was used for other previous requests.

# Events

### `stream.on('response', …)`

#### `response`

**类型: [`PlainResponse`](typescript.md#plainresponse)**

This is emitted when a HTTP response is received.

```js
import { pipeline } from "node:stream/promises";
import { createWriteStream } from "node:fs";
import got from "got";

const readStream = got.stream("http://example.com/image.png", {
  throwHttpErrors: false,
});

const onError = (error) =    {
  // Do something with it.
};

readStream.on("response", async (response) =    {
  if (response.headers.age     3600) {
    console.log("Failure - response too old");
    readStream.destroy(); // Destroy the stream to prevent hanging resources.
    return;
  }

  // Prevent `onError` being called twice.
  readStream.off("error", onError);

  try {
    await pipeline(readStream, createWriteStream("image.png"));

    console.log("Success");
  } catch (error) {
    onError(error);
  }
});

readStream.once("error", onError);
```

### `stream.on('downloadProgress', …)`

#### `progress`

**类型: [`Progress`](typescript.md#progress)**

This is emitted on every time `stream.downloadProgress` is updated.

### `stream.on('uploadProgress', …)`

#### `progress`

**类型: [`Progress`](typescript.md#progress)**

This is emitted on every time `stream.uploadProgress` is updated.

<a name="retry"></a>

### `stream.on('retry', …)`

To enable retrying when using streams, a retry handler must be attached.

When this event is emitted, you should reset the stream you were writing to and prepare the body again.

!!! note

    - [`HTTPError`s](./8-errors.md#httperror) cannot be retried if [`options.throwHttpErrors`](./2-options.md#throwhttperrors) is `false`.
      This is because stream data is saved to `error.response.body` and streams can be read only once.
    - For the Promise API, there is no such limitation.

#### `retryCount`

**类型: `number`**

The current retry count.

#### `error`

**类型: [`RequestError`](8-errors.md#requesterror)**

The error that caused this retry.

#### `createRetryStream`

**类型: `(options?: OptionsInit) =    Request`**

```js
import fs from "node:fs";
import got from "got";

let writeStream;

const fn = (retryStream) =    {
  const options = {
    headers: {
      foo: "bar",
    },
  };

  const stream = retryStream ?? got.stream("https://example.com", options);

  if (writeStream) {
    writeStream.destroy();
  }

  writeStream = fs.createWriteStream("example-com.html");

  stream.pipe(writeStream);

  // If you don't attach the listener, it will NOT make a retry.
  // It automatically checks the listener count so it knows whether to retry or not :)
  stream.once("retry", (retryCount, error, createRetryStream) =    {
    fn(createRetryStream()); // or: fn(createRetryStream(optionsToMerge))
  });
};

fn();
```

### `stream.on('redirect', …)`

#### `updatedOptions`

**类型: [`Options`](2-options.md)**

The new options used to make the next request.

#### `response`

**类型: [`IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage)**

The `IncomingMessage` instance the redirect came from.

# Internal usage

This are the functions used internally by Got.  
Other non-documented functions are private and should not be accessible.

### `stream.flush()`

This function is executed automatically by Got. It marks the current stream as ready. If an error occurs before `stream.flush()` is called, it's thrown immediately after `stream.flush()`.

### `stream._beforeError(error)`

This function is called instead `stream.destroy(error)`, required in order to exectue async logic, such as reading the response (e.g. when `ERR_NON_2XX_3XX_RESPONSE` occurs).

### `stream._noPipe`

**类型: `boolean`**

Whether piping is disabled or not. This property is used by the Promise API.

---

# `Response`

源码: [`source/core/response.ts`](./source/core/response.ts)

**Extends: [`IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage)**

### `requestUrl`

**类型: `URL`**

The original request URL. It is the first argument when calling `got(…)`.

### `redirectUrls`

**类型: `URL[]`**

The redirect URLs.

### `request`

**类型: `Request`**

The underlying Got stream.

### `ip`

**类型: `string`**

The server's IP address.

!!! note

    - Not available when the response is cached.

### `isFromCache`

**类型: `boolean`**

Whether the response comes from cache or not.

### `ok`

**类型: `boolean`**

Whether the response was successful

!!! note

    - A request is successful when the status code of the final request is `2xx` or `3xx`.
    - When [following redirects](2-options.md#followredirect), a request is successful **only** when the status code of the final request is `2xx`.
    - `304` responses are always considered successful.
    - Got throws automatically when `response.ok` is `false` and `throwHttpErrors` is `true`.

### `statusCode`

**类型: `number`**

The [HTTP status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).

### `url`

**类型: `string`**

The final URL after all redirects.

### `timings`

**类型: [`Timings`](typescript.md#timings)**

The same as `request.timings`.

### `retryCount`

**类型: `number`**

The same as `request.retryCount`.

### `rawBody`

**类型: `Buffer`**

!!! note

    - This property is only accessible when using Promise API.

The raw response body buffer.

### `body`

**类型: `unknown`**

!!! note

    - This property is only accessible when using Promise API.

The parsed response body.

### `aborted`

**类型: `boolean`**

The same as `request.aborted`.

### `complete`

**类型: `boolean`**

If `true`, the response has been fully parsed.

### `socket`

**类型: `net.Socket | tls.TLSSocket`**

The same as `request.socket`.

### `headers`

**类型: `object<string, string>`**

The [response headers](https://nodejs.org/api/http.html#http_message_headers).

### `statusMessage`

**类型: `string`**

The status message corresponding to the status code.
