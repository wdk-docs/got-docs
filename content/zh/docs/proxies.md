---
title: "代理"
linkTitle: ""
weight: 7
type: "docs"
---

你可以使用带`agent`选项的[`tunnel`](https://github.com/koichik/node-tunnel)包代理:

```js
const got = require("got");
const tunnel = require("tunnel");

got("https://sindresorhus.com", {
	agent: tunnel.httpOverHttp({
		proxy: {
			host: "localhost"
		}
	})
});
```

另外, 在你的程序用 [`global-agent`](https://github.com/gajus/global-agent) 配置所有 HTTP/HTTPS 流量全球代理.
