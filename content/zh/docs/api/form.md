---
title: "表单数据-form"
linkTitle: "表单数据"
weight: 14
description: >
  form 数据合成与上传处理
type: "docs"
---

{{% alert color="primary" %}}

1. 如果你提供这个选项，`got.stream()`将是只读的。
2. 此选项是不可枚举，不会与实例的默认值合并。

{{% /alert %}}

使用[`(new URLSearchParams(object)).toString()`](https://nodejs.org/api/url.html#url_constructor_new_urlsearchparams_obj)将`form body`转换为查询字符串.

如果设置为`true`和`Content-Type`头没有设置, 它将被设置为`application/x-www-form-urlencoded`.

您可以使用[`form-data`](https://github.com/form-data/form-data)包来创建表单数据的 POST 请求:

```js
const fs = require("fs");
const got = require("got");
const FormData = require("form-data");

const form = new FormData();

form.append("my_file", fs.createReadStream("/foo/bar.jpg"));

got.post("https://example.com", { body: form });
```
