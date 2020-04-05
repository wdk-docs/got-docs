---
title: "流-stream"
linkTitle: "流"
weight: 3
description: >
  返回带有额外事件[全双工流](https://nodejs.org/api/stream.html#stream_class_stream_duplex)
type: "docs"
---

{{% alert color="primary" %}}
进度事件，重定向的事件和请求/响应的事件还可以与`promises`使用。
{{% /alert %}}

{{% alert color="primary" %}}
访问`response.isFromCache`你需要使用`got.stream(url, options).isFromCache`. 该值是不确定的，直到`response`事件。
{{% /alert %}}

## got.stream(url, options?)

Sets `options.isStream` to `true`.

## .on('request', request)

`request` event to get the request object of the request.

{{% alert color="info" %}}
You can use `request` event to abort request:
{{% /alert %}}

```js
got
	.stream("https://github.com")
	.on("request", request => setTimeout(() => request.abort(), 50));
```

## .on('response', response)

The `response` event to get the response object of the final request.

## .on('redirect', response, nextOptions)

The `redirect` event to get the response object of a redirect. The second argument is options for the next request to the redirect location.

## .on('uploadProgress', progress)

## .on('downloadProgress', progress)

Progress events for uploading (sending a request) and downloading (receiving a response). The `progress` argument is an object like:

```js
{
	percent: 0.1,
	transferred: 1024,
	total: 10240
}
```

If the `content-length` header is missing, `total` will be `undefined`.

```js
(async () => {
	const response = await got("https://sindresorhus.com")
		.on("downloadProgress", progress => {
			// Report download progress
		})
		.on("uploadProgress", progress => {
			// Report upload progress
		});

	console.log(response);
})();
```

## .on('error', error, body, response)

The `error` event emitted in case of a protocol error (like `ENOTFOUND` etc.) or status error (4xx or 5xx). The second argument is the body of the server response in case of status error. The third argument is a response object.
