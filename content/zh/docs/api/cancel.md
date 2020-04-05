---
title: "中止请求-cancel"
linkTitle: "中止请求"
weight: 11
description: >
  返回的承诺`Got`具有[`.cancel()`](https://github.com/sindresorhus/p-cancelable)方法，该方法被调用时，将中止请求。
type: "docs"
---

```js
(async () => {
	const request = got(url, options);

	// …

	// In another part of the code
	if (something) {
		request.cancel();
	}

	// …

	try {
		await request;
	} catch (error) {
		if (request.isCanceled) {
			// Or `error instanceof got.CancelError`
			// Handle cancelation
		}

		// Handle other errors
	}
})();
```

当使用钩字，简单地抛出异常中止请求。

```js
const got = require("got");

(async () => {
	const request = got(url, {
		hooks: {
			beforeRequest: [
				() => {
					throw new Error("Oops. Request canceled.");
				}
			]
		}
	});

	try {
		await request;
	} catch (error) {
		// …
	}
})();
```
