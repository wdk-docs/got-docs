---
title: "安装使用"
linkTitle: ""
weight: 1
description: >
  Got的安装与使用
type: "docs"
---

## 安装

```
$ npm install got
```

## 常规举例

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

## 流举例

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
