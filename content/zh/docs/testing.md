---
title: "测试"
linkTitle: ""
weight: 13
type: "docs"
---

您可以使用[`nock`](https://github.com/node-nock/nock)包来测试你的请求`mock`端点:

```js
const got = require("got");
const nock = require("nock");

nock("https://sindresorhus.com")
	.get("/")
	.reply(200, "Hello world!");

(async () => {
	const response = await got("https://sindresorhus.com");
	console.log(response.body);
	//=> 'Hello world!'
})();
```

对于真正的集成测试，我们建议您使用带[`create-test-server`](https://github.com/lukechilds/create-test-server)[`ava`](https://github.com/avajs/ava).
我们使用宏，所以我们不必`server.listen（）`和`server.close（）`每一个测试。
看看我们的测试之一:

```js
test(
	"retry function gets iteration count",
	withServer,
	async (t, server, got) => {
		let knocks = 0;
		server.get("/", (request, response) => {
			if (knocks++ === 1) {
				response.end("who`s there?");
			}
		});

		await got({
			retry: {
				calculateDelay: ({ attemptCount }) => {
					t.true(is.number(attemptCount));
					return attemptCount < 2 ? 1 : 0;
				}
			}
		});
	}
);
```
