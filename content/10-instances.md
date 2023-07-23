# 实例

源码: [`source/create.ts`](./source/create.ts)

## `got.defaults`

### `options`

**类型: [`Options`](2-options.md)**

用于此实例的选项。

### `handlers`

**类型: [`Handler[]`](typescript.md#handler)**

```ts
(options: Options, next: …) => next(options)
```

处理程序数组。
`next`函数返回一个[`Promise`](1-promise.md)或一个[`Request` Got stream](3-streams.md)。

你可以通过调用`got(…)`直接执行它们。
它们是某种“全局钩子”——首先调用这些函数。最后一个处理程序(它是不可见的)是`asPromise`或`asStream`，取决于`options.isStream`属性。

### `mutableDefaults`

**类型: `boolean`**  
**默认: `false`**

确定是否可以修改`got.defaults.options`。

## `got.extend(…options, …instances)`

!!! Tip

     `options` 可以包括 `handlers` 和 `mutableDefaults`.

!!! Note

    不可枚举的属性，例如 `body`, `json`, 和 `form`, 不会被合并。

用合并的默认选项配置一个新的`got`实例。
使用[`options.merge(…)`](2-options.md#merge)将这些选项与父实例的`defaults.options`合并。

```js
import got from "got";

const client = got.extend({
  prefixUrl: "https://httpbin.org",
  headers: {
    "x-foo": "bar",
  },
});

const { headers } = await client.get("headers").json();
console.log(headers["x-foo"]); //=> 'bar'

const jsonClient = client.extend({
  responseType: "json",
  resolveBodyOnly: true,
  headers: {
    "x-lorem": "impsum",
  },
});

const { headers: headers2 } = await jsonClient.get("headers");
console.log(headers2["x-foo"]); //=> 'bar'
console.log(headers2["x-lorem"]); //=> 'impsum'
```

!!! Note

    - 处理程序可以是异步的，并且可以返回 `Promise`, 但从来没有 `Promise<Stream>` 如果 `options.isStream` 是 `true`.
    - 流必须始终同步处理。
    - 为了使用流执行异步工作，  `beforeRequest` 应该使用Hook。

创建可以同时处理承诺和流的处理程序的推荐方法是:

```js
import got from "got";

// Create a non-async handler, but we can return a Promise later.
const handler = (options, next) => {
  if (options.isStream) {
    // It's a Stream, return synchronously.
    return next(options);
  }

  // For asynchronous work, return a Promise.
  return (async () => {
    try {
      const response = await next(options);
      response.yourOwnProperty = true;
      return response;
    } catch (error) {
      // Every error will be replaced by this one.
      // Before you receive any error here,
      // it will be passed to the `beforeError` hooks first.
      // Note: this one won't be passed to `beforeError` hook. It's final.
      throw new Error("Your very own error.");
    }
  })();
};

const instance = got.extend({ handlers: [handler] });
```
