# Axios

Axios与Got非常相似。区别在于Axios首先针对浏览器，而Got则充分利用了Node.js的特性。

## 常见的选项

这些选项也保持不变:

- [`url`](../2-options.md#url)
- [`method`](../2-options.md#method)
- [`headers`](../2-options.md#headers)
- [`maxRedirects`](../2-options.md#maxredirects)
- [`decompress`](../2-options.md#decompress)

## 重命名选项

我们非常关心可读性，所以我们重命名了这些选项:

- `httpAgent` → [`agent.http`](../2-options.md#agent)
- `httpsAgent` → [`agent.https`](../2-options.md#agent)
- `socketPath` → [`url`](../2-options.md#enableunixsockets)
- `responseEncoding` → [`encoding`](../2-options.md#encoding)
- `auth.username` → [`username`](../2-options.md#username)
- `auth.password` → [`password`](../2-options.md#password)
- `data` → [`body`](../2-options.md#body) / [`json`](../2-options.md#json) / [`form`](../2-options.md#form)
- `params` → [`searchParams`](../2-options.md#serachparams)

## 行为的改变

- `transformRequest` → [`hooks.beforeRequest`](../9-hooks.md#beforerequest)
  - The API is different.
- `transformResponse` → [`hooks.afterResponse`](../9-hooks.md#afterresponse)
  - The API is different.
- `baseUrl` → [`prefixUrl`](../2-options.md#prefixurl)
  - The `prefixUrl` is always prepended to the `url`.
- [`timeout`](../6-timeout.md)
  - This option is now an object. You can now set timeouts on particular events!
- [`responseType`](../2-options.md#responsetype)
  - Accepts `'text'`, `'json'` or `'buffer'`.

## 突发的变化

- `onUploadProgress`
  - This option does not exist. Instead, use [`got(…).on('uploadProgress', …)`](../3-streams.md#uploadprogress).
- `onDownloadProgress`
  - This option does not exist. Instead, use [`got(…).on('downloadProgress', …)`](../3-streams.md#downloadprogress).
- `maxContentLength`
  - This option does not exist. Instead, use [a handler](../examples/advanced-creation.js).
- `validateStatus`
  - This option does not exist. Got automatically validates the status according to [the specification](https://datatracker.ietf.org/doc/html/rfc7231#section-6).
- `proxy`
  - This option does not exist. You need to pass [an `agent`](../tips.md#proxy) instead.
- `cancelToken`
  - This option does not exist, but will be implemented soon. For now, use `promise.cancel()` or `stream.destroy()`.
- `paramsSerializer`
  - This option does not exist.
- `maxBodyLength`
  - This option does not exist.

## 响应

响应对象也不同:

- `response.data` → [`response.body`](../3-streams.md#response-1)
- `response.status` → [`response.statusCode`](../3-streams.md#response-1)
- `response.statusText` → [`response.statusMessage`](../3-streams.md#response-1)
- `response.config` → [`response.request.options`](../3-streams.md#response-1)
- [`response.request`](../3-streams.md#response-1)
  - Returns [a Got stream](../3-streams.md).

The `response.headers` object remains the same.

## 拦截器

而是提供了[hooks](../9-hooks.md)，这更灵活。

## 错误

错误看起来是一样的，不同的是`error.request`返回一个get流。
此外，Got还提供了[更多细节](../8-errors.md)，使调试更容易。

## 取消

虽然Got不支持[`AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)，你可以使用`promise.cancel()`。

## 方便的方法

方便的方法，如`axios.get(…)`等保持不变:`got.get(…)`。
而不是`axios.create(…)`使用 `got.extend(…)`。

## 你可以走了!

好吧，你已经走了这么远了  :tada:  
请查看[文档](../README.md#documentation)。值得花时间读一读
这里有一些很好的建议(../tips.md)。

如果有些事情不清楚或没有正常工作，不要犹豫[提出问题](https://github.com/sindresorhus/got/issues/new/choose).
