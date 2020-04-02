---
title: "用法"
linkTitle: ""
weight: 2
type: "docs"
---

## 举例

### 常规

```js
const got = require("got");

(async () => {
	try {
		const response = await got("https://sindresorhus.com");
		console.log(response.body);
		//=> '<!doctype html> ...'
	} catch (error) {
		console.log(error.response.body);
		//=> 'Internal server error ...'
	}
})();
```

### 流

```js
const stream = require("stream");
const { promisify } = require("util");
const fs = require("fs");
const got = require("got");

const pipeline = promisify(stream.pipeline);

(async () => {
	await pipeline(
		got.stream("https://sindresorhus.com"),
		fs.createWriteStream("index.html")
	);

	// For POST, PUT, and PATCH methods `got.stream` returns a `stream.Writable`
	await pipeline(
		fs.createReadStream("index.html"),
		got.stream.post("https://sindresorhus.com")
	);
})();
```

**Tip:** Using `from.pipe(to)` doesn't forward errors. If you use it, switch to [`Stream.pipeline(from, ..., to, callback)`](https://nodejs.org/api/stream.html#stream_stream_pipeline_streams_callback) instead (available from Node v10).

## API

It's a `GET` request by default, but can be changed by using different methods or via `options.method`.

**默认情况下，`Got`将重试失败。 要禁用此选项, 设置 [`options.retry`](#retry) 为 `0`.**

### got(url?, options?)

Returns a Promise for a [`response` object](#response) or a [stream](#streams-1) if `options.isStream` is set to true.

#### url

Type: `string | object`

The URL to request, as a string, a [`https.request` options object](https://nodejs.org/api/https.html#https_https_request_options_callback), or a [WHATWG `URL`](https://nodejs.org/api/url.html#url_class_url).

Properties from `options` will override properties in the parsed `url`.

If no protocol is specified, it will throw a `TypeError`.

**Note:** this can also be an option.

#### options

Type: `object`

Any of the [`https.request`](https://nodejs.org/api/https.html#https_https_request_options_callback) options.

**Note:** Legacy URL support is disabled. `options.path` is supported only for backwards compatibility. Use `options.pathname` and `options.searchParams` instead. `options.auth` has been replaced with `options.username` & `options.password`.

##### prefixUrl

Type: `string | URL`

When specified, `prefixUrl` will be prepended to `url`. The prefix can be any valid URL, either relative or absolute. A trailing slash `/` is optional - one will be added automatically.

**Note:** `prefixUrl` will be ignored if the `url` argument is a URL instance.

**Note:** Leading slashes in `input` are disallowed when using this option to enforce consistency and avoid confusion. For example, when the prefix URL is `https://example.com/foo` and the input is `/bar`, there's ambiguity whether the resulting URL would become `https://example.com/foo/bar` or `https://example.com/bar`. The latter is used by browsers.

**Tip:** Useful when used with [`got.extend()`](#custom-endpoints) to create niche-specific Got-instances.

**Tip:** You can change `prefixUrl` using hooks as long as the URL still includes the `prefixUrl`. If the URL doesn't include it anymore, it will throw.

```js
const got = require("got");

(async () => {
	await got("unicorn", { prefixUrl: "https://cats.com" });
	//=> 'https://cats.com/unicorn'

	const instance = got.extend({
		prefixUrl: "https://google.com"
	});

	await instance("unicorn", {
		hooks: {
			beforeRequest: [
				options => {
					options.prefixUrl = "https://cats.com";
				}
			]
		}
	});
	//=> 'https://cats.com/unicorn'
})();
```

##### headers

Type: `object`\
Default: `{}`

Request headers.

Existing headers will be overwritten. Headers set to `undefined` will be omitted.

##### isStream

Type: `boolean`\
Default: `false`

Returns a `Stream` instead of a `Promise`. This is equivalent to calling `got.stream(url, options?)`.

##### body

Type: `string | Buffer | stream.Readable` or [`form-data` instance](https://github.com/form-data/form-data)

**Note #1:** The `body` option cannot be used with the `json` or `form` option.

**Note #2:** If you provide this option, `got.stream()` will be read-only.

**Note #3:** If you provide a payload with the `GET` or `HEAD` method, it will throw a `TypeError` unless the method is `GET` and the `allowGetBody` option is set to `true`.

**Note #4:** This option is not enumerable and will not be merged with the instance defaults.

The `content-length` header will be automatically set if `body` is a `string` / `Buffer` / `fs.createReadStream` instance / [`form-data` instance](https://github.com/form-data/form-data), and `content-length` and `transfer-encoding` are not manually set in `options.headers`.

##### json

Type: `object | Array | number | string | boolean | null` _(JSON-serializable values)_

**Note #1:** If you provide this option, `got.stream()` will be read-only.
**Note #2:** This option is not enumerable and will not be merged with the instance defaults.

JSON body. If the `Content-Type` header is not set, it will be set to `application/json`.

##### context

Type: `object`

User data. In contrast to other options, `context` is not enumerable.

**Note:** The object is never merged, it's just passed through. Got will not modify the object in any way.

It's very useful for storing auth tokens:

```js
const got = require("got");

const instance = got.extend({
	hooks: {
		beforeRequest: [
			options => {
				if (!options.context || !options.context.token) {
					throw new Error("Token required");
				}

				options.headers.token = options.context.token;
			}
		]
	}
});

(async () => {
	const context = {
		token: "secret"
	};

	const response = await instance("https://httpbin.org/headers", { context });

	// Let's see the headers
	console.log(response.body);
})();
```

##### responseType

Type: `string`\
Default: `'text'`

**Note:** When using streams, this option is ignored.

The parsing method. Can be `'text'`, `'json'` or `'buffer'`.

The promise also has `.text()`, `.json()` and `.buffer()` methods which return another Got promise for the parsed body.\
It's like setting the options to `{responseType: 'json', resolveBodyOnly: true}` but without affecting the main Got promise.

Example:

```js
(async () => {
	const responsePromise = got(url);
	const bufferPromise = responsePromise.buffer();
	const jsonPromise = responsePromise.json();

	const [response, buffer, json] = Promise.all([
		responsePromise,
		bufferPromise,
		jsonPromise
	]);
	// `response` is an instance of Got Response
	// `buffer` is an instance of Buffer
	// `json` is an object
})();
```

```js
// This
const body = await got(url).json();

// is semantically the same as this
const body = await got(url, { responseType: "json", resolveBodyOnly: true });
```

##### resolveBodyOnly

Type: `boolean`\
Default: `false`

When set to `true` the promise will return the [Response body](#body-1) instead of the [Response](#response) object.

##### cookieJar

Type: `object` | [`tough.CookieJar` instance](https://github.com/salesforce/tough-cookie#cookiejar)

**Note:** If you provide this option, `options.headers.cookie` will be overridden.

Cookie support. You don't have to care about parsing or how to store them. [Example](#cookies).

##### cookieJar.setCookie

Type: `Function<Promise>`

The function takes two arguments: `rawCookie` (`string`) and `url` (`string`).

##### cookieJar.getCookieString

Type: `Function<Promise>`

The function takes one argument: `url` (`string`).

##### ignoreInvalidCookies

Type: `boolean`\
Default: `false`

Ignore invalid cookies instead of throwing an error. Only useful when the `cookieJar` option has been set. Not recommended.

##### encoding

Type: `string`\
Default: `'utf8'`

[Encoding](https://nodejs.org/api/buffer.html#buffer_buffers_and_character_encodings) to be used on `setEncoding` of the response data.

To get a [`Buffer`](https://nodejs.org/api/buffer.html), you need to set [`responseType`](#responseType) to `buffer` instead.

##### form

Type: `object | true`

**Note #1:** If you provide this option, `got.stream()` will be read-only.
**Note #2:** This option is not enumerable and will not be merged with the instance defaults.

The form body is converted to query string using [`(new URLSearchParams(object)).toString()`](https://nodejs.org/api/url.html#url_constructor_new_urlsearchparams_obj).

If set to `true` and the `Content-Type` header is not set, it will be set to `application/x-www-form-urlencoded`.

##### searchParams

Type: `string | object<string, string | number> | URLSearchParams`

Query string that will be added to the request URL. This will override the query string in `url`.

If you need to pass in an array, you can do it using a `URLSearchParams` instance:

```js
const got = require("got");

const searchParams = new URLSearchParams([
	["key", "a"],
	["key", "b"]
]);

got("https://example.com", { searchParams });

console.log(searchParams.toString());
//=> 'key=a&key=b'
```

And if you need a different array format, you could use the [`query-string`](https://github.com/sindresorhus/query-string) package:

```js
const got = require("got");
const queryString = require("query-string");

const searchParams = queryString.stringify(
	{ key: ["a", "b"] },
	{ arrayFormat: "bracket" }
);

got("https://example.com", { searchParams });

console.log(searchParams);
//=> 'key[]=a&key[]=b'
```

##### timeout

Type: `number | object`

Milliseconds to wait for the server to end the response before aborting the request with [`got.TimeoutError`](#gottimeouterror) error (a.k.a. `request` property). By default, there's no timeout.

This also accepts an `object` with the following fields to constrain the duration of each phase of the request lifecycle:

- `lookup` starts when a socket is assigned and ends when the hostname has been resolved. Does not apply when using a Unix domain socket.
- `connect` starts when `lookup` completes (or when the socket is assigned if lookup does not apply to the request) and ends when the socket is connected.
- `secureConnect` starts when `connect` completes and ends when the handshaking process completes (HTTPS only).
- `socket` starts when the socket is connected. See [request.setTimeout](https://nodejs.org/api/http.html#http_request_settimeout_timeout_callback).
- `response` starts when the request has been written to the socket and ends when the response headers are received.
- `send` starts when the socket is connected and ends with the request has been written to the socket.
- `request` starts when the request is initiated and ends when the response's end event fires.

##### retry

Type: `number | object`\
Default:

- limit: `2`
- calculateDelay: `({attemptCount, retryOptions, error, computedValue}) => computedValue`
- methods: `GET` `PUT` `HEAD` `DELETE` `OPTIONS` `TRACE`
- statusCodes: [`408`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408) [`413`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/413) [`429`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) [`500`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500) [`502`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/502) [`503`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/503) [`504`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/504) [`521`](https://support.cloudflare.com/hc/en-us/articles/115003011431#521error) [`522`](https://support.cloudflare.com/hc/en-us/articles/115003011431#522error) [`524`](https://support.cloudflare.com/hc/en-us/articles/115003011431#524error)
- maxRetryAfter: `undefined`
- errorCodes: `ETIMEDOUT` `ECONNRESET` `EADDRINUSE` `ECONNREFUSED` `EPIPE` `ENOTFOUND` `ENETUNREACH` `EAI_AGAIN`

An object representing `limit`, `calculateDelay`, `methods`, `statusCodes`, `maxRetryAfter` and `errorCodes` fields for maximum retry count, retry handler, allowed methods, allowed status codes, maximum [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) time and allowed error codes.

**Note:** When using streams, this option is ignored. If the connection is reset when downloading, you need to catch the error and clear the file you were writing into to prevent duplicated content.

If `maxRetryAfter` is set to `undefined`, it will use `options.timeout`.\
If [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After) header is greater than `maxRetryAfter`, it will cancel the request.

Delays between retries counts with function `1000 * Math.pow(2, retry) + Math.random() * 100`, where `retry` is attempt number (starts from 1).

The `calculateDelay` property is a `function` that receives an object with `attemptCount`, `retryOptions`, `error` and `computedValue` properties for current retry count, the retry options, error and default computed value. The function must return a delay in milliseconds (`0` return value cancels retry).

By default, it retries _only_ on the specified methods, status codes, and on these network errors:

- `ETIMEDOUT`: One of the [timeout](#timeout) limits were reached.
- `ECONNRESET`: Connection was forcibly closed by a peer.
- `EADDRINUSE`: Could not bind to any free port.
- `ECONNREFUSED`: Connection was refused by the server.
- `EPIPE`: The remote side of the stream being written has been closed.
- `ENOTFOUND`: Couldn't resolve the hostname to an IP address.
- `ENETUNREACH`: No internet connection.
- `EAI_AGAIN`: DNS lookup timed out.

##### followRedirect

Type: `boolean`\
Default: `true`

Defines if redirect responses should be followed automatically.

Note that if a `303` is sent by the server in response to any request type (`POST`, `DELETE`, etc.), Got will automatically request the resource pointed to in the location header via `GET`. This is in accordance with [the spec](https://tools.ietf.org/html/rfc7231#section-6.4.4).

##### methodRewriting

Type: `boolean`\
Default: `true`

By default, redirects will use [method rewriting](https://tools.ietf.org/html/rfc7231#section-6.4). For example, when sending a POST request and receiving a `302`, it will resend the body to the new location using the same HTTP method (`POST` in this case).

##### allowGetBody

Type: `boolean`\
Default: `false`

**Note:** The [RFC 7321](https://tools.ietf.org/html/rfc7231#section-4.3.1) doesn't specify any particular behavior for the GET method having a payload, therefore **it's considered an [anti-pattern](https://en.wikipedia.org/wiki/Anti-pattern)**.

Set this to `true` to allow sending body for the `GET` method. However, the [HTTP/2 specification](https://tools.ietf.org/html/rfc7540#section-8.1.3) says that `An HTTP GET request includes request header fields and no payload body`, therefore when using the HTTP/2 protocol this option will have no effect. This option is only meant to interact with non-compliant servers when you have no other choice.

##### maxRedirects

Type: `number`\
Default: `10`

If exceeded, the request will be aborted and a `MaxRedirectsError` will be thrown.

##### decompress

Type: `boolean`\
Default: `true`

Decompress the response automatically. This will set the `accept-encoding` header to `gzip, deflate, br` on Node.js 11.7.0+ or `gzip, deflate` for older Node.js versions, unless you set it yourself.

Brotli (`br`) support requires Node.js 11.7.0 or later.

If this is disabled, a compressed response is returned as a `Buffer`. This may be useful if you want to handle decompression yourself or stream the raw compressed data.

##### cache

Type: `object`\
Default: `false`

[Cache adapter instance](#cache-adapters) for storing cached response data.

##### dnsCache

Type: `object`\
Default: `false`

[Cache adapter instance](#cache-adapters) for storing cached DNS data.

##### request

Type: `Function`\
Default: `http.request | https.request` _(Depending on the protocol)_

Custom request function. The main purpose of this is to [support HTTP2 using a wrapper](#experimental-http2-support).

##### useElectronNet

Type: `boolean`\
Default: `false`

[**Deprecated**](https://github.com/sindresorhus/got#electron-support-has-been-deprecated)

When used in Electron, Got will use [`electron.net`](https://electronjs.org/docs/api/net/) instead of the Node.js `http` module. According to the Electron docs, it should be fully compatible, but it's not entirely. See [#443](https://github.com/sindresorhus/got/issues/443) and [#461](https://github.com/sindresorhus/got/issues/461).

##### throwHttpErrors

Type: `boolean`\
Default: `true`

Determines if a `got.HTTPError` is thrown for error responses (non-2xx status codes).

If this is disabled, requests that encounter an error status code will be resolved with the `response` instead of throwing. This may be useful if you are checking for resource availability and are expecting error responses.

##### agent

Same as the [`agent` option](https://nodejs.org/api/http.html#http_http_request_url_options_callback) for `http.request`, but with an extra feature:

If you require different agents for different protocols, you can pass a map of agents to the `agent` option. This is necessary because a request to one protocol might redirect to another. In such a scenario, Got will switch over to the right protocol agent for you.

```js
const got = require("got");
const HttpAgent = require("agentkeepalive");
const { HttpsAgent } = HttpAgent;

got("https://sindresorhus.com", {
	agent: {
		http: new HttpAgent(),
		https: new HttpsAgent()
	}
});
```

##### hooks

Type: `object<string, Function[]>`

Hooks allow modifications during the request lifecycle. Hook functions may be async and are run serially.

##### hooks.init

Type: `Function[]`\
Default: `[]`

Called with plain [request options](#options), right before their normalization. This is especially useful in conjunction with [`got.extend()`](#instances) when the input needs custom handling.

See the [Request migration guide](documentation/migration-guides.md#breaking-changes) for an example.

**Note:** This hook must be synchronous!

##### hooks.beforeRequest

Type: `Function[]`\
Default: `[]`

Called with [normalized](source/normalize-arguments.ts) [request options](#options). Got will make no further changes to the request before it is sent (except the body serialization). This is especially useful in conjunction with [`got.extend()`](#instances) when you want to create an API client that, for example, uses HMAC-signing.

See the [AWS section](#aws) for an example.

##### hooks.beforeRedirect

Type: `Function[]`\
Default: `[]`

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

##### hooks.beforeRetry

Type: `Function[]`\
Default: `[]`

**Note:** When using streams, this hook is ignored.

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

**Note:** When retrying in a `afterResponse` hook, all remaining `beforeRetry` hooks will be called without the `error` and `retryCount` arguments.

##### hooks.afterResponse

Type: `Function[]`\
Default: `[]`

**Note:** When using streams, this hook is ignored.

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

##### hooks.beforeError

Type: `Function[]`\
Default: `[]`

Called with an `Error` instance. The error is passed to the hook right before it's thrown. This is especially useful when you want to have more detailed errors.

**Note:** Errors thrown while normalizing input options are thrown directly and not part of this hook.

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

#### \_pagination

Type: `object`

**Note:** This feature is marked as experimental as we're [looking for feedback](https://github.com/sindresorhus/got/issues/1052) on the API and how it works. The feature itself is stable, but the API may change based on feedback. So if you decide to try it out, we suggest locking down the `got` dependency semver range or use a lockfile.

##### \_pagination.transform

Type: `Function`\
Default: `response => JSON.parse(response.body)`

A function that transform [`Response`](#response) into an array of items. This is where you should do the parsing.

##### \_pagination.paginate

Type: `Function`\
Default: [`Link` header logic](source/index.ts)

The function takes three arguments:

- `response` - The current response object.
- `allItems` - An array of the emitted items.
- `currentItems` - Items from the current response.

It should return an object representing Got options pointing to the next page. If there are no more pages, `false` should be returned.

For example, if you want to stop when the response contains less items than expected, you can use something like this:

```js
const got = require("got");

(async () => {
	const limit = 10;

	const items = got.paginate("https://example.com/items", {
		searchParams: {
			limit,
			offset: 0
		},
		_pagination: {
			paginate: (response, allItems, currentItems) => {
				const previousSearchParams = response.request.options.searchParams;
				const { offset: previousOffset } = previousSearchParams;

				if (currentItems.length < limit) {
					return false;
				}

				return {
					searchParams: {
						...previousSearchParams,
						offset: previousOffset + limit
					}
				};
			}
		}
	});

	console.log("Items from all pages:", items);
})();
```

##### \_pagination.filter

Type: `Function`\
Default: `(item, allItems, currentItems) => true`

Checks whether the item should be emitted or not.

##### \_pagination.shouldContinue

Type: `Function`\
Default: `(item, allItems, currentItems) => true`

Checks whether the pagination should continue.

For example, if you need to stop **before** emitting an entry with some flag, you should use `(item, allItems, currentItems) => !item.flag`. If you want to stop **after** emitting the entry, you should use `(item, allItems, currentItems) => allItems.some(entry => entry.flag)` instead.

##### \_pagination.countLimit

Type: `number`\
Default: `Infinity`

The maximum amount of items that should be emitted.

### Response

The response object will typically be a [Node.js HTTP response stream](https://nodejs.org/api/http.html#http_class_http_incomingmessage), however, if returned from the cache it will be a [response-like object](https://github.com/lukechilds/responselike) which behaves in the same way.

#### request

Type: `object`

**Note:** This is not a [http.ClientRequest](https://nodejs.org/api/http.html#http_class_http_clientrequest).

- `options` - The Got options that were set on this request.

#### body

Type: `string | object | Buffer` _(Depending on `options.responseType`)_

The result of the request.

#### url

Type: `string`

The request URL or the final URL after redirects.

#### ip

Type: `string`

The remote IP address.

**Note:** Not available when the response is cached. This is hopefully a temporary limitation, see [lukechilds/cacheable-request#86](https://github.com/lukechilds/cacheable-request/issues/86).

#### requestUrl

Type: `string`

The original request URL.

#### timings

Type: `object`

The object contains the following properties:

- `start` - Time when the request started.
- `socket` - Time when a socket was assigned to the request.
- `lookup` - Time when the DNS lookup finished.
- `connect` - Time when the socket successfully connected.
- `secureConnect` - Time when the socket securely connected.
- `upload` - Time when the request finished uploading.
- `response` - Time when the request fired `response` event.
- `end` - Time when the response fired `end` event.
- `error` - Time when the request fired `error` event.
- `abort` - Time when the request fired `abort` event.
- `phases` - `wait` - `timings.socket - timings.start` - `dns` - `timings.lookup - timings.socket` - `tcp` - `timings.connect - timings.lookup` - `tls` - `timings.secureConnect - timings.connect` - `request` - `timings.upload - (timings.secureConnect || timings.connect)` - `firstByte` - `timings.response - timings.upload` - `download` - `timings.end - timings.response` - `total` - `(timings.end || timings.error || timings.abort) - timings.start`

If something has not been measured yet, it will be `undefined`.

**Note:** The time is a `number` representing the milliseconds elapsed since the UNIX epoch.

#### isFromCache

Type: `boolean`

Whether the response was retrieved from the cache.

#### redirectUrls

Type: `string[]`

The redirect URLs.

#### retryCount

Type: `number`

The number of times the request was retried.

### Streams

**Note:** Progress events, redirect events and request/response events can also be used with promises.

**Note:** To access `response.isFromCache` you need to use `got.stream(url, options).isFromCache`. The value will be undefined until the `response` event.

### got.stream(url, options?)

Sets `options.isStream` to `true`.

Returns a [duplex stream](https://nodejs.org/api/stream.html#stream_class_stream_duplex) with additional events:

#### .on('request', request)

`request` event to get the request object of the request.

**Tip:** You can use `request` event to abort request:

```js
got
	.stream("https://github.com")
	.on("request", request => setTimeout(() => request.abort(), 50));
```

#### .on('response', response)

The `response` event to get the response object of the final request.

#### .on('redirect', response, nextOptions)

The `redirect` event to get the response object of a redirect. The second argument is options for the next request to the redirect location.

#### .on('uploadProgress', progress)

#### .on('downloadProgress', progress)

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

#### .on('error', error, body, response)

The `error` event emitted in case of a protocol error (like `ENOTFOUND` etc.) or status error (4xx or 5xx). The second argument is the body of the server response in case of status error. The third argument is a response object.

### Pagination

### got.paginate(url, options?)

Returns an async iterator:

```js
(async () => {
	const countLimit = 10;

	const pagination = got.paginate(
		"https://api.github.com/repos/sindresorhus/got/commits",
		{
			_pagination: { countLimit }
		}
	);

	console.log(`Printing latest ${countLimit} Got commits (newest to oldest):`);

	for await (const commitData of pagination) {
		console.log(commitData.commit.message);
	}
})();
```

See [`options._pagination`](#_pagination) for more pagination options.

### got.get(url, options?)

### got.post(url, options?)

### got.put(url, options?)

### got.patch(url, options?)

### got.head(url, options?)

### got.delete(url, options?)

Sets `options.method` to the method name and makes a request.

## 实例

### got.extend(...options)

Configure a new `got` instance with default `options`. The `options` are merged with the parent instance's `defaults.options` using [`got.mergeOptions`](#gotmergeoptionsparentoptions-newoptions). You can access the resolved options with the `.defaults` property on the instance.

```js
const client = got.extend({
	prefixUrl: "https://example.com",
	headers: {
		"x-unicorn": "rainbow"
	}
});

client.get("demo");

/* HTTP Request =>
 * GET /demo HTTP/1.1
 * Host: example.com
 * x-unicorn: rainbow
 */
```

```js
(async () => {
	const client = got.extend({
		prefixUrl: "httpbin.org",
		headers: {
			"x-foo": "bar"
		}
	});
	const { headers } = await client.get("headers").json();
	//=> headers['x-foo'] === 'bar'

	const jsonClient = client.extend({
		responseType: "json",
		resolveBodyOnly: true,
		headers: {
			"x-baz": "qux"
		}
	});
	const { headers: headers2 } = await jsonClient.get("headers");
	//=> headers2['x-foo'] === 'bar'
	//=> headers2['x-baz'] === 'qux'
})();
```

Additionally, `got.extend()` accepts two properties from the `defaults` object: `mutableDefaults` and `handlers`. Example:

```js
// You can now modify `mutableGot.defaults.options`.
const mutableGot = got.extend({ mutableDefaults: true });

const mergedHandlers = got.extend({
	handlers: [
		(options, next) => {
			delete options.headers.referer;

			return next(options);
		}
	]
});
```

**Note:** Handlers can be asynchronous. The recommended approach is:

```js
const handler = (options, next) => {
	if (options.stream) {
		// It's a Stream
		return next(options);
	}

	// It's a Promise
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

### got.extend(...instances)

Merges many instances into a single one:

- options are merged using [`got.mergeOptions()`](#gotmergeoptionsparentoptions-newoptions) (+ hooks are merged too),
- handlers are stored in an array (you can access them through `instance.defaults.handlers`).

### got.extend(...options, ...instances, ...)

It's possible to combine options and instances.\
It gives the same effect as `got.extend(...options).extend(...instances)`:

```js
const a = { headers: { cat: "meow" } };
const b = got.extend({
	options: {
		headers: {
			cow: "moo"
		}
	}
});

// The same as `got.extend(a).extend(b)`.
// Note `a` is options and `b` is an instance.
got.extend(a, b);
//=> {headers: {cat: 'meow', cow: 'moo'}}
```

### got.mergeOptions(parentOptions, newOptions)

Extends parent options. Avoid using [object spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#Spread_in_object_literals) as it doesn't work recursively:

```js
const a = {headers: {cat: 'meow', wolf: ['bark', 'wrrr']}};
const b = {headers: {cow: 'moo', wolf: ['auuu']}};

{...a, ...b}            // => {headers: {cow: 'moo', wolf: ['auuu']}}
got.mergeOptions(a, b)  // => {headers: {cat: 'meow', cow: 'moo', wolf: ['auuu']}}
```

Options are deeply merged to a new object. The value of each key is determined as follows:

- If the new property is set to `undefined`, it keeps the old one.
- If both properties are an instances of `URLSearchParams`, a new URLSearchParams instance is created. The values are merged using [`urlSearchParams.append(key, value)`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/append).
- If the parent property is an instance of `URL` and the new value is a `string` or `URL`, a new URL instance is created: [`new URL(new, parent)`](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL#Syntax).
- If the new property is a plain `object`: - If the parent property is a plain `object` too, both values are merged recursively into a new `object`. - Otherwise, only the new value is deeply cloned.
- If the new property is an `Array`, it overwrites the old one with a deep clone of the new property.
- Properties that are not enumerable, such as `context`, `body`, `json`, and `form`, will not be merged.

```js
const a = { json: { cat: "meow" } };
const b = { json: { cow: "moo" } };

got.mergeOptions(a, b);
//=> {json: {cow: 'moo'}}
```

- Otherwise, the new value is assigned to the key.

### got.defaults

Type: `object`

The Got defaults used in that instance.

#### [options](#options)

#### handlers

Type: `Function[]`\
Default: `[]`

An array of functions. You execute them directly by calling `got()`. They are some sort of "global hooks" - these functions are called first. The last handler (_it's hidden_) is either [`asPromise`](source/as-promise.ts) or [`asStream`](source/as-stream.ts), depending on the `options.isStream` property.

Each handler takes two arguments:

##### [options](#options)

##### next()

Returns a `Promise` or a `Stream` depending on [`options.isStream`](#isstream).

```js
const settings = {
	handlers: [
		(options, next) => {
			if (options.isStream) {
				// It's a Stream, so we can perform stream-specific actions on it
				return next(options).on("request", request => {
					setTimeout(() => {
						request.abort();
					}, 50);
				});
			}

			// It's a Promise
			return next(options);
		}
	],
	options: got.mergeOptions(got.defaults.options, {
		responseType: "json"
	})
};

const jsonGot = got.extend(settings);
```

#### mutableDefaults

Type: `boolean`\
Default: `false`

A read-only boolean describing whether the defaults are mutable or not. If set to `true`, you can [update headers over time](#hooksafterresponse), for example, update an access token when it expires.
