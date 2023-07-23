# 错误

源代码:

- [`source/core/errors.ts`](./source/core/errors.ts)
- [`source/as-promise/types.ts`](./source/as-promise/types.ts)
- [`source/core/response.ts`](./source/core/response.ts)

所有Got错误包含各种元数据，例如:

- `code` - 一个字符串就像 `ERR_NON_2XX_3XX_RESPONSE`,
- `options` - [`Options`](2-options.md)的实例,
- `request` - Got Stream的一个实例,
- `response` (optional) - Got Response的一个实例,
- `timings` (optional) - 指出 `response.timings`.

### 捕获异步堆栈跟踪

阅读文章[这里](async-stack-traces.md)。

!!! Note

    当根错误设置了`code`属性时，错误代码可能会有所不同。

## `RequestError`

**Code: `ERR_GOT_REQUEST_ERROR`**

当请求失败时。包含带有错误类代码的 `code` 属性，如`ECONNREFUSED`。下面的所有错误都继承了这个错误。

## `CacheError`

**Code: `ERR_CACHE_ACCESS`**

当缓存方法失败时，例如，如果数据库宕机或存在文件系统错误。

## `ReadError`

**Code: `ERR_READING_RESPONSE_STREAM`**

从响应流读取失败时。

## `ParseError`

**Code: `ERR_BODY_PARSE_FAILURE`**

当服务器响应代码为2xx时，解析体失败。包含一个`response`属性。

## `UploadError`

**Code: `ERR_UPLOAD`**

当请求体是流并且从该流读取时发生错误时。

## `HTTPError`

**Code: `ERR_NON_2XX_3XX_RESPONSE`**

请求不成功时。

当最后请求的状态码为`2xx` 或 `3xx`时，请求成功。

当[跟随重定向](2-options.md#followredirect)时，只有当最终请求的状态码为`2xx`时，请求才会成功。

**Note:**

> - `304` 回应总是被认为是成功的。

## `MaxRedirectsError`

**Code: `ERR_TOO_MANY_REDIRECTS`**

当服务器重定向你超过10次。包含一个`response`属性。

## `UnsupportedProtocolError`

!!! Note

    此错误不是公开的。

**Code: `ERR_UNSUPPORTED_PROTOCOL`**

当给定不支持的协议时。

## `TimeoutError`

**Code: `ETIMEDOUT`**

当请求由于[timeout](6-timeout.md)而中止时。包括`event`(一个字符串)属性和`timings`。

## `CancelError`

**Code: `ERR_CANCELED`**

当使用`promise.cancel()`终止请求时。

## `RetryError`

**Code: `ERR_RETRYING`**

抛出时总是触发新的重试。

## `AbortError`

**Code: `ERR_ABORTED`**

当请求被[AbortController.abort()](https://developer.mozilla.org/en-US/docs/Web/API/AbortController/abort)终止时。
