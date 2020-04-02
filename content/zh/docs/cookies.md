---
title: "Cookies"
linkTitle: ""
weight: 8
type: "docs"
---

您可以使用[`tough-cookie`](https://github.com/salesforce/tough-cookie)包:

```js
const { promisify } = require("util");
const got = require("got");
const { CookieJar } = require("tough-cookie");

(async () => {
	const cookieJar = new CookieJar();
	const setCookie = promisify(cookieJar.setCookie.bind(cookieJar));

	await setCookie("foo=bar", "https://example.com");
	await got("https://example.com", { cookieJar });
})();
```
