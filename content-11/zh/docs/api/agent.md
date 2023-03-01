---
title: "代理-agent"
linkTitle: "代理"
weight: 12
description: >
  与`http.request`的[`agent` 选项](https://nodejs.org/api/http.html#http_http_request_url_options_callback)相同, 但是有一个额外的功能
type: "docs"
---

## `agent` 选项

如果您需要不同协议的不同代理, 你可以给`agent`选项传递一个代理映射. 这是必要的，因为一个协议的请求可能重定向到另一个。 在这种情况下，得到了会切换到合适的协议代理为您服务。

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

## tunnel

你可以使用带`agent`选项的[`tunnel`](https://github.com/koichik/node-tunnel)包代理

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

另外, 你可以在程序中用 [`global-agent`](https://github.com/gajus/global-agent) 配置所有 HTTP/HTTPS 全局流量代理.
