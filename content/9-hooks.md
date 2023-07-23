# 钩子 API

## `hooks`

**类型: `object<string, Function[]>`**

该选项表示要运行的钩子。抛出的错误将自动转换为[`RequestError`](8-errors.md#requesterror).

### `init`

**类型: `InitHook[]`**  
**默认: `[]`**

```ts
(plainRequestOptions: OptionsInit, options: Options) => void
```

用普通请求选项调用，就在它们规范化之前。
第二个参数表示当前的[`Options`](2-options.md)实例。

!!! note

    这个钩子必须是同步的。

!!! note

    每次合并选项时都会调用这个函数。

!!! note

    `options`对象可能没有`url`属性。要修改它，可以使用`beforeRequest`钩子。

!!! note

    这个钩子在创建一个新的`Options`实例时被调用。
    不要将它与`Request`或`got(…)`的创建混淆。

!!! note

    当使用`got(url)`或`got(url, undefined, defaults)`时，这个钩子将 **不** 被调用。

当输入需要自定义处理时，这与`got.extend()`结合使用特别有用。

例如，这可以用于修复错别字，以便更快地从旧版本迁移。

```js
import got from "got";

const instance = got.extend({
  hooks: {
    init: [
      (plain) => {
        if ("followRedirects" in plain) {
          plain.followRedirect = plain.followRedirects;
          delete plain.followRedirects;
        }
      },
    ],
  },
});

// 通常，下面的代码会抛出:
const response = await instance("https://example.com", {
  followRedirects: true,
});

// 没有名为`followRedirects`的选项，但我们在`init`钩子中纠正了它。
```

或者你可以创建自己的选项并将其存储在一个上下文中:

```js
import got from "got";

const instance = got.extend({
  hooks: {
    init: [
      (plain, options) => {
        if ("secret" in plain) {
          options.context.secret = plain.secret;
          delete plain.secret;
        }
      },
    ],
    beforeRequest: [
      (options) => {
        options.headers.secret = options.context.secret;
      },
    ],
  },
});

const { headers } = await instance("https://httpbin.org/anything", {
  secret: "passphrase",
}).json();

console.log(headers.Secret);
//=> 'passphrase'
```

### `beforeRequest`

**类型: `BeforeRequestHook[]`**  
**默认: `[]`**

```ts
(options: Options) => Promisable<void | Response | ResponseLike>;
```

在使用`options.createNativeRequestOptions()`发出请求之前调用。
当您想要签名请求时，此钩子与`got.extend()`结合使用特别有用。

!!! note

    在发送请求之前，Got不会对请求做进一步的更改。

!!! note

    更改`options.json`或`options.form`对请求没有影响，您应该更改`options.body`。
    如果需要，相应地更新`options.headers`。

```js
import got from "got";

const response = await got.post("https://httpbin.org/anything", {
  json: { payload: "old" },
  hooks: {
    beforeRequest: [
      (options) => {
        options.body = JSON.stringify({ payload: "new" });
        options.headers["content-length"] = options.body.length.toString();
      },
    ],
  },
});
```

!!! tip

    你可以通过提前返回一个[`ClientRequest`-like](https://nodejs.org/api/http.html#http_class_http_clientrequest)实例或一个[`IncomingMessage`-like](https://nodejs.org/api/http.html#http_class_http_incomingmessage)实例来间接覆盖`request`函数。这在创建自定义缓存机制时非常有用。
    [阅读更多关于这个技巧的内容](cache.md#advanced-caching-mechanisms).

### `beforeRedirect`

**类型: `BeforeRedirectHook[]`**  
**默认: `[]`**

```ts
(updatedOptions: Options, plainResponse: PlainResponse) => Promisable<void>;
```

相当于`beforeRequest`，但在重定向时。

!!! tip

    当你想要避免死站点时，这是特别有用的。

```js
import got from "got";

const response = await got("https://example.com", {
  hooks: {
    beforeRedirect: [
      (options, response) => {
        if (options.hostname === "deadSite") {
          options.hostname = "fallbackSite";
        }
      },
    ],
  },
});
```

### `beforeRetry`

**类型: `BeforeRetryHook[]`**  
**默认: `[]`**

```ts
(error: RequestError, retryCount: number) => Promisable<void>;
```

The equivalent of `beforeError` but when retrying. Additionally, there is a second argument `retryCount`, the current retry number.

!!! note

    当使用Stream API时，这个钩子将被忽略。

!!! note

    当重试时，`beforeRequest`钩子在重试后被调用。

!!! note

    如果没有重试，则调用`beforeError`钩子。

当您想要检索重试的原因时，此钩子特别有用。

```js
import got from "got";

await got("https://httpbin.org/status/500", {
  hooks: {
    beforeRetry: [
      (error, retryCount) => {
        console.log(`Retrying [${retryCount}]: ${error.code}`);
        // Retrying [1]: ERR_NON_2XX_3XX_RESPONSE
      },
    ],
  },
});
```

### `afterResponse`

**类型: `AfterResponseHook[]`**  
**默认: `[]`**

```ts
(response: Response, retryWithMergedOptions: (options: OptionsInit) => never) =>
  Promisable<Response | CancelableRequest<Response>>;
```

每个函数都应该返回响应。这在您想要刷新访问令牌时特别有用。

!!! note

    当使用Stream API时，这个钩子将被忽略。

!!! note

    Calling the `retryWithMergedOptions` function will trigger `beforeRetry` hooks. If the retry is successful, all remaining `afterResponse` hooks will be called. In case of an error, `beforeRetry` hooks will be called instead.
>   Meanwhile the `init`, `beforeRequest` , `beforeRedirect` as well as already executed `afterResponse` hooks will be skipped.

```js
import got from "got";

const instance = got.extend({
  hooks: {
    afterResponse: [
      (response, retryWithMergedOptions) => {
        // Unauthorized
        if (response.statusCode === 401) {
          // Refresh the access token
          const updatedOptions = {
            headers: {
              token: getNewToken(),
            },
          };

          // Update the defaults
          instance.defaults.options.merge(updatedOptions);

          // Make a new retry
          return retryWithMergedOptions(updatedOptions);
        }

        // No changes otherwise
        return response;
      },
    ],
    beforeRetry: [
      (error) => {
        // This will be called on `retryWithMergedOptions(...)`
      },
    ],
  },
  mutableDefaults: true,
});
```

### `beforeError`

**类型: `BeforeErrorHook[]`**  
**默认: `[]`**

```ts
(error: RequestError) => Promisable<RequestError>;
```

用[`RequestError`](8-errors.md#requesterror)实例调用。在抛出钩子之前，错误被传递给钩子。

当您希望获得更详细的错误时，这尤其有用。

```js
import got from "got";

await got("https://api.github.com/repos/sindresorhus/got/commits", {
  responseType: "json",
  hooks: {
    beforeError: [
      (error) => {
        const { response } = error;
        if (response && response.body) {
          error.name = "GitHubError";
          error.message = `${response.body.message} (${response.statusCode})`;
        }

        return error;
      },
    ],
  },
});
```
