# 同步 API

源码: [`source/as-promise/index.ts`](./source/as-promise/index.ts)

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

或者，你可以只传递包含`url`属性的选项。

```js
import got from "got";

const { headers } = await got({
  url: "https://httpbin.org/anything",
  headers: {
    foo: "bar",
  },
}).json();
```

这在语义上与第一种方法相同。

## `promise.json<T>()`

**Returns: `Promise<T>`**

一个快捷方法，它给出一个返回JSON对象的Promise。

它在语义上与将[`options.resolveBodyOnly`](2-options.md#resolvebodyonly)设置为`true`和将[`options.responseType`](2-options.md#responsetype)设置为`'json'`相同。

## `promise.buffer()`

**Returns: `Promise<Buffer>`**

给出一个Promise返回一个[Buffer](https://nodejs.org/api/buffer.html)的快捷方法.

它在语义上与将 [`options.resolveBodyOnly`](2-options.md#resolvebodyonly)设置为`true`和将[`options.responseType`](2-options.md#responsetype)设置为`'buffer'`相同。

## `promise.text()`

**Returns: `Promise<string>`**

一个快捷方法，给出一个返回字符串的Promise。

它在语义上与将 [`options.resolveBodyOnly`](2-options.md#resolvebodyonly) 设置为 `true` 和将 [`options.responseType`](2-options.md#responsetype) 设置为 `'text'`相同。.

## `promise.cancel(reason?: string)`

取消请求，并可选择提供原因。

取消是同步的。
在承诺已经完成或多次之后调用它没有任何作用。

这将导致promise以[`CancelError`](8-errors.md#cancelerror)拒绝。

## `promise.isCanceled`

**类型: `boolean`**

承诺是否被取消。

## `promise.on(event, handler)`

事件与[流API](3-streams.md#events)中相同.

## `promise.off(event, handler)`

移除用[`promise.on`](1-promise.md#promiseonevent-handler)注册的监听器

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
