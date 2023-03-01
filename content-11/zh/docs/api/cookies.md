---
title: "Cookies 支持"
linkTitle: "Cookies"
weight: 8
description: >
  Cookie支持. 你不必在意解析或如何保存它们。
type: "docs"
---

## cookieJar

类型: `object` | [`tough.CookieJar` instance](https://github.com/salesforce/tough-cookie#cookiejar)

{{% alert color="primary" %}}
如果你提供这个选项， `options.headers.cookie` 将被覆盖。
{{% /alert %}}

[Example](#cookies).

## cookieJar.setCookie

类型: `Function<Promise>`

该函数有两个参数: `rawCookie` (`string`) 和 `url` (`string`).

## cookieJar.getCookieString

类型: `Function<Promise>`

该函数有一个参数: `url` (`string`).

## tough-cookie

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
