# 重试 API

!!! note

	如果你正在寻找使用流的重试实现，请查看[重试流API](3-streams.md#retry)。

!!! tip

	你可以在任何钩子中抛出 [`RetryError`](8-errors.md#retryerror)来触发重试。

!!! tip

	`afterResponse`钩子公开了一个专用函数，用于使用合并选项进行重试。
	 [了解更多](9-hooks.md#afterresponse).

## `retry`

**类型: `object`**  
**Default:**

```js
{
	limit: 2,
	methods: [
		'GET',
		'PUT',
		'HEAD',
		'DELETE',
		'OPTIONS',
		'TRACE'
	],
	statusCodes: [
		408,
		413,
		429,
		500,
		502,
		503,
		504,
		521,
		522,
		524
	],
	errorCodes: [
		'ETIMEDOUT',
		'ECONNRESET',
		'EADDRINUSE',
		'ECONNREFUSED',
		'EPIPE',
		'ENOTFOUND',
		'ENETUNREACH',
		'EAI_AGAIN'
	],
	maxRetryAfter: undefined,
	calculateDelay: ({computedValue}) => computedValue,
	backoffLimit: Number.POSITIVE_INFINITY,
	noise: 100
}
```

This option represents the `retry` object.

### `limit`

**类型: `number`**

The maximum retry count.

### `methods`

**类型: `string[]`**

The allowed methods to retry on.

!!! note

	By default, Got does not retry on `POST`.

### `statusCodes`

**类型: `number[]`**

!!! note

	Only [**unsuccessful**](8-errors.md#) requests are retried. In order to retry successful requests, use an [`afterResponse`](9-hooks.md#afterresponse) hook.

The allowed [HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) to retry on.

### `errorCodes`

**类型: `string[]`**

The allowed error codes to retry on.

- `ETIMEDOUT` - One of the [timeout limits](6-timeout.md) was reached.
- `ECONNRESET`- The connection was forcibly closed.
- `EADDRINUSE`- Could not bind to any free port.
- `ECONNREFUSED`- The connection was refused by the server.
- `EPIPE` - The remote side of the stream being written has been closed.
- `ENOTFOUND` - Could not resolve the hostname to an IP address.
- `ENETUNREACH` - No internet connection.
- `EAI_AGAIN` - DNS lookup timed out.

### `maxRetryAfter`

**类型: `number | undefined`**  
**默认: `options.timeout.request`**

The upper limit of [`retry-after` header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After). If `undefined`, it will use `options.timeout` as the value.

If the limit is exceeded, the request is canceled.

### `calculateDelay`

**类型: `Function`**

```ts
(retryObject: RetryObject) => Promisable<number>;
```

```ts
interface RetryObject {
  attemptCount: number;
  retryOptions: RetryOptions;
  error: RequestError;
  computedValue: number;
  retryAfter?: number;
}
```

The function used to calculate the delay before the next request is made. Returning `0` cancels the retry.

!!! note

	This function is responsible for the entire retry mechanism, including the `limit` property. To support this, you need to check if `computedValue` is different than `0`.

!!! tip

	This is especially useful when you want to scale down the computed value.

```js
import got from "got";

await got("https://httpbin.org/anything", {
  retry: {
    calculateDelay: ({ computedValue }) => {
      return computedValue / 10;
    },
  },
});
```

### `backoffLimit`

**类型: `number`**

The upper limit of the `computedValue`.

By default, the `computedValue` is calculated in the following way:

```ts
2 ** (attemptCount - 1) * 1000 + noise;
```

The delay increases exponentially.  
In order to prevent this, you can set this value to a fixed value, such as `1000`.

### `noise`

**类型: `number`**

The maximum acceptable retry noise in the range of `-100` to `+100`.
