---
title: "表单数据"
linkTitle: ""
weight: 9
type: "docs"
---

您可以使用[`form-data`](https://github.com/form-data/form-data)包来创建表单数据的 POST 请求:

```js
const fs = require("fs");
const got = require("got");
const FormData = require("form-data");

const form = new FormData();

form.append("my_file", fs.createReadStream("/foo/bar.jpg"));

got.post("https://example.com", {
	body: form
});
```
