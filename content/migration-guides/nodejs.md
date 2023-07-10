# Node.js

让我们提出一个简单的请求。对于Node.js，这是:

```js
import http from "node:http";

const request = http.request("https://httpbin.org/anything", (response) => {
  if (response.statusCode >= 400) {
    request.destroy(new Error());
    return;
  }

  const chunks = [];

  response.on("data", (chunk) => {
    chunks.push(chunk);
  });

  response.once("end", () => {
    const buffer = Buffer.concat(chunks);

    if (response.statusCode >= 400) {
      const error = new Error(`Unsuccessful response: ${response.statusCode}`);
      error.body = buffer.toString();
      return;
    }

    const text = buffer.toString();

    console.log(text);
  });

  response.once("error", console.error);
});

request.once("error", console.error);
request.end();
```

使用 Got, 这就变成了:

```js
import got from "got";

try {
  const { body } = await got("https://httpbin.org/anything");
  console.log(body);
} catch (error) {
  console.error(error);
}
```

更加清晰。但是流呢?

```js
import http from "node:http";
import fs from "node:fs";

const source = fs.createReadStream("article.txt");

const request = http.request(
  "https://httpbin.org/anything",
  {
    method: "POST",
  },
  (response) => {
    response.pipe(fs.createWriteStream("httpbin.txt"));
  }
);

source.pipe(request);
```

嗯，就这么简单:

```js
import got from "got";
import stream from "node:stream";
import fs from "node:fs";

await stream.promises.pipeline(
  fs.createReadStream("article.txt"),
  got.stream.post("https://httpbin.org/anything"),
  fs.createWriteStream("httpbin.txt")
);
```

优点是Got还自动处理错误，因此您不必创建自定义侦听器。

此外，Got还支持重定向、压缩、高级超时、缓存、分页、cookie、钩子等等!

## 下一个什么?

不幸的是，Got选项与Node.js选项差别太大。提供一个简短的总结是不可能的。  
别担心，你会很快学会的——它们很容易理解!每个选项都附有示例。

请查看[文档](../README.md#documentation)。
值得花时间去读。
这里有一些很好的建议(../tips.md)。

如果有些事情不清楚或没有正常工作，不要犹豫[提出问题](https://github.com/sindresorhus/got/issues/new/choose).
