# 同步 API

源码: [`source/as-promise/index.ts`](../source/as-promise/index.ts)

主Got函数返回一个[`Promise`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)。
虽然为了支持取消，使用[`PCancelable`](https://github.com/sindresorhus/p-cancelable)代替纯`Promise`。

## <code>got(url: string | URL, options?: [OptionsInit](typescript.md#optionsinit), defaults?: [Options](2-options.md))</code>

**返回: <code>Promise<[Response](response.md)>**</code>

最常见的方法是将URL作为第一个参数传递，然后将选项作为第二个参数传递。

```js
import got from "got";

const { headers } = await got("https://httpbin.org/anything", {
  headers: {
    foo: "bar",
  },
}).json();
```

## <code>got(options: [OptionsInit](typescript.md#optionsinit))</code>

**Returns: <code>Promise<[Response](3-streams.md#response-1)>**</code>

Alternatively, you can pass only options containing a `url` property.

```js
import got from "got";

const { headers } = await got({
  url: "https://httpbin.org/anything",
  headers: {
    foo: "bar",
  },
}).json();
```

This is semantically the same as the first approach.

## `promise.json<T>()`

**Returns: `Promise<T>`**

A shortcut method that gives a Promise returning a JSON object.

It is semantically the same as settings [`options.resolveBodyOnly`](2-options.md#resolvebodyonly) to `true` and [`options.responseType`](2-options.md#responsetype) to `'json'`.

## `promise.buffer()`

**Returns: `Promise<Buffer>`**

A shortcut method that gives a Promise returning a [Buffer](https://nodejs.org/api/buffer.html).

It is semantically the same as settings [`options.resolveBodyOnly`](2-options.md#resolvebodyonly) to `true` and [`options.responseType`](2-options.md#responsetype) to `'buffer'`.

## `promise.text()`

**Returns: `Promise<string>`**

A shortcut method that gives a Promise returning a string.

It is semantically the same as settings [`options.resolveBodyOnly`](2-options.md#resolvebodyonly) to `true` and [`options.responseType`](2-options.md#responsetype) to `'text'`.

## `promise.cancel(reason?: string)`

取消请求，并可选择提供原因。

取消是同步的。
在承诺已经完成或多次之后调用它没有任何作用。

这将导致promise以[`CancelError`](8-errors.md#cancelerror)拒绝。

## `promise.isCanceled`

**类型: `boolean`**

Whether the promise is canceled.

## `promise.on(event, handler)`

The events are the same as in [Stream API](3-streams.md#events).

## `promise.off(event, handler)`

Removes listener registered with [`promise.on`](1-promise.md#promiseonevent-handler)

```js
import { createReadStream } from "node:fs";
import got from "got";

const ongoingRequestPromise = got.post(uploadUrl, {
  body: createReadStream("sample.txt"),
});

const eventListener = (progress: Progress) => {
  console.log(progress);
};

ongoingRequestPromise.on("uploadProgress", eventListener);

setTimeout(() => {
  ongoingRequestPromise.off("uploadProgress", eventListener);
}, 500);

await ongoingRequestPromise;
```
