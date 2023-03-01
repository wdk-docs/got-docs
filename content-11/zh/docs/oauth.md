---
title: "OAuth"
linkTitle: ""
weight: 10
type: "docs"
---

您可以使用[`oauth-1.0a`](https://github.com/ddo/oauth-1.0a)包来创建签名的 OAuth 请求:

```js
const got = require("got");
const crypto = require("crypto");
const OAuth = require("oauth-1.0a");

const oauth = OAuth({
	consumer: {
		key: process.env.CONSUMER_KEY,
		secret: process.env.CONSUMER_SECRET
	},
	signature_method: "HMAC-SHA1",
	hash_function: (baseString, key) =>
		crypto
			.createHmac("sha1", key)
			.update(baseString)
			.digest("base64")
});

const token = {
	key: process.env.ACCESS_TOKEN,
	secret: process.env.ACCESS_TOKEN_SECRET
};

const url = "https://api.twitter.com/1.1/statuses/home_timeline.json";

got(url, {
	headers: oauth.toHeader(oauth.authorize({ url, method: "GET" }, token)),
	responseType: "json"
});
```
