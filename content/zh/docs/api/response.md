---
title: "响应-response"
linkTitle: "响应"
weight: 3
description: >
  响应对象是典型地[Node.js的HTTP响应流](https://nodejs.org/api/http.html#http_class_http_incomingmessage), 然而, 如果从高速缓存返回的这将是一个[响应状物体](https://github.com/lukechilds/responselike)其行为以相同的方式。
type: "docs"
---

## request

**类型:** `object`

{{% alert color="primary" %}}

这不是一个[http.ClientRequest](https://nodejs.org/api/http.html#http_class_http_clientrequest).

{{% /alert %}}

- `options` - 在`request`上设定的`Got` 选项.

## body

**类型:** `string` | `object` | `Buffer` _(取决于 `options.responseType`)_

该请求的结果。

## url

**类型:** `string`

重定向后请求的网址或最终网址 。

## ip

**类型:** `string`

远程 IP 地址.

{{% alert color="primary" %}}

不可用时，响应被缓存。 这是一个希望暂时限制, 查看[lukechilds/cacheable-request#86](https://github.com/lukechilds/cacheable-request/issues/86).

{{% /alert %}}

## requestUrl

**类型:** `string`

原始请求 URL。

## timings

**类型:** `object`

该对象包含下列属性:

- `start` - Time 当请求启动事件.
- `socket` - Time 当一个`socket`被分配到请求.
- `lookup` - Time 当 DNS 查找完成.
- `connect` - Time 当成功地连接在`socket`.
- `secureConnect` - Time 当牢固地连`socket`.
- `upload` - Time 当请求完成上传.
- `response` - Time 当请求释放`response`事件.
- `end` - Time 当响应释放`end`事件.
- `error` - Time 当请求释放`error`事件.
- `abort` - Time 当请求释放`abort`事件.
- `phases` - `wait` - `timings.socket - timings.start` - `dns` - `timings.lookup - timings.socket` - `tcp` - `timings.connect - timings.lookup` - `tls` - `timings.secureConnect - timings.connect` - `request` - `timings.upload - (timings.secureConnect || timings.connect)` - `firstByte` - `timings.response - timings.upload` - `download` - `timings.end - timings.response` - `total` - `(timings.end || timings.error || timings.abort) - timings.start`

如果事情还没有被测量, 这将是 `undefined`.

{{% alert color="primary" %}}
时间是`number`类型， 从 UNIX 纪元所经过的毫秒.
{{% /alert %}}

## isFromCache

**类型:** `boolean`

响应是否从缓存中检索。

## redirectUrls

**类型:** `string[]`

重定向的 URL。

## retryCount

**类型:** `number`

请求被重试的次数.
