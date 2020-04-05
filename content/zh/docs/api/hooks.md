---
title: "钩子-hooks"
linkTitle: "钩子选项"
weight: 9
description: >
  钩子允许请求生命周期期间修改。钩子函数可以是并行和串行。
type: "docs"
---

**类型:** `object<string, Function[]>`

## hooks.init

**类型:** `Function[]`\
**默认:** `[]`

Called with plain [request options](#options), right before their normalization. This is especially useful in conjunction with [`got.extend()`](#instances) when the input needs custom handling.

See the [Request migration guide](documentation/migration-guides.md#breaking-changes) for an example.

{{% alert color="primary" %}}
这个钩子必须是同步的！
{{% /alert %}}

## hooks.beforeRequest

**类型:** `Function[]`\
**默认:** `[]`

Called with [normalized](source/normalize-arguments.ts) [request options](#options). Got will make no further changes to the request before it is sent (except the body serialization). This is especially useful in conjunction with [`got.extend()`](#instances) when you want to create an API client that, for example, uses HMAC-signing.

See the [AWS section](#aws) for an example.

## hooks.beforeRedirect

**类型:** `Function[]`\
**默认:** `[]`

Called with [normalized](source/normalize-arguments.ts) [request options](#options) and the redirect [response](#response). Got will make no further changes to the request. This is especially useful when you want to avoid dead sites. Example:

```js
const got = require("got");

got("https://example.com", {
	hooks: {
		beforeRedirect: [
			(options, response) => {
				if (options.hostname === "deadSite") {
					options.hostname = "fallbackSite";
				}
			}
		]
	}
});
```

## hooks.beforeRetry

**类型:** `Function[]`\
**默认:** `[]`

{{% alert color="primary" %}}
When using streams, this hook is ignored.
{{% /alert %}}
Called with [normalized](source/normalize-arguments.ts) [request options](#options), the error and the retry count. Got will make no further changes to the request. This is especially useful when some extra work is required before the next try. Example:

```js
const got = require("got");

got.post("https://example.com", {
	hooks: {
		beforeRetry: [
			(options, error, retryCount) => {
				if (error.response.statusCode === 413) {
					// Payload too large
					options.body = getNewBody();
				}
			}
		]
	}
});
```

{{% alert color="primary" %}}
When retrying in a `afterResponse` hook, all remaining `beforeRetry` hooks will be called without the `error` and `retryCount` arguments.
{{% /alert %}}

## hooks.afterResponse

**类型:** `Function[]`\
**默认:** `[]`

{{% alert color="primary" %}}
When using streams, this hook is ignored.
{{% /alert %}}
Called with [response object](#response) and a retry function. Calling the retry function will trigger `beforeRetry` hooks.

Each function should return the response. This is especially useful when you want to refresh an access token. Example:

```js
const got = require("got");

const instance = got.extend({
	hooks: {
		afterResponse: [
			(response, retryWithMergedOptions) => {
				if (response.statusCode === 401) {
					// Unauthorized
					const updatedOptions = {
						headers: {
							token: getNewToken() // Refresh the access token
						}
					};

					// Save for further requests
					instance.defaults.options = got.mergeOptions(
						instance.defaults.options,
						updatedOptions
					);

					// Make a new retry
					return retryWithMergedOptions(updatedOptions);
				}

				// No changes otherwise
				return response;
			}
		],
		beforeRetry: [
			(options, error, retryCount) => {
				// This will be called on `retryWithMergedOptions(...)`
			}
		]
	},
	mutableDefaults: true
});
```

## hooks.beforeError

**类型:** `Function[]`\
**默认:** `[]`

Called with an `Error` instance. The error is passed to the hook right before it's thrown. This is especially useful when you want to have more detailed errors.

{{% alert color="primary" %}}
Errors thrown while normalizing input options are thrown directly and not part of this hook.
{{% /alert %}}

```js
const got = require("got");

got("https://api.github.com/some-endpoint", {
	hooks: {
		beforeError: [
			error => {
				const { response } = error;
				if (response && response.body) {
					error.name = "GitHubError";
					error.message = `${response.body.message} (${response.statusCode})`;
				}

				return error;
			}
		]
	}
});
```
