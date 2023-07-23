# 插件

> 另一个关于如何像老板一样使用 Got 的例子 :electric_plug:

好了，你们已经学了一些基础知识。太好了!

当涉及到高级用法时，自定义实例非常有用。
例如，看看[`gh-got`](https://github.com/sindresorhus/gh-got)。
看起来很复杂，但是……这很简单，而且非常有用。

在我们开始之前，我们需要找到[GitHub API 文档](https://developer.github.com/v3/)。

让我们写下最重要的信息:

1. 根端点是 `https://api.github.com/`.
2. 我们将使用API的版本3。 `Accept`标头需要设置为 `application/vnd.github.v3+json`.
3. 正文是JSON格式。
4. 我们将使用OAuth2进行授权。
5. 我们可能会收到 `400 Bad Request` 或 `422 Unprocessable Entity`. 正文包含有关错误的详细信息。
6. _分页?_ 是啊!本机支持。
7. 率限制。这些头文件很有趣:

      - `X-RateLimit-Limit`
      - `X-RateLimit-Remaining`
      - `X-RateLimit-Reset`

        也 `X-GitHub-Request-Id` 可能对调试有用。

8. `User-Agent` 头是必需的。

当我们有了所有必要的信息，我们可以开始搅拌 :cake:

## 根端点

这里没什么可做的。只需要扩展一个实例并提供`prefixUrl`选项:

```js
import got from "got";

const instance = got.extend({
  prefixUrl: "https://api.github.com",
});

export default instance;
```

## v3 API

GitHub 需要知道我们正在使用哪个 API 版本。我们将使用`Accept`头文件:

```js
import got from "got";

const instance = got.extend({
  prefixUrl: "https://api.github.com",
  headers: {
    accept: "application/vnd.github.v3+json",
  },
});

export default instance;
```

## JSON body

我们会用到它[`options.responseType`](2-options.md#responsetype):

```js
import got from "got";

const instance = got.extend({
  prefixUrl: "https://api.github.com",
  headers: {
    accept: "application/vnd.github.v3+json",
  },
  responseType: "json",
});

export default instance;
```

## 授权

通常会设置一些环境变量，例如`GITHUB_TOKEN`。
你可以很容易地修改所有应用中的令牌，对吧?酷。是什么……
我们想为每个应用程序提供一个唯一的令牌。然后我们将需要创建一个新选项-它将默认为环境变量，但你可以很容易地覆盖它。

Got 执行选项验证，不知道`token`是一个需要的选项，所以它会抛出。
我们可以在一个`init`钩子中处理它，并将它保存在`options.context`中。

```js
import got from "got";

const instance = got.extend({
  prefixUrl: "https://api.github.com",
  headers: {
    accept: "application/vnd.github.v3+json",
  },
  responseType: "json",
  context: {
    token: process.env.GITHUB_TOKEN,
  },
  hooks: {
    init: [
      (raw, options) => {
        if ("token" in raw) {
          options.context.token = raw.token;
          delete raw.token;
        }
      },
    ],
  },
});

export default instance;
```

对于其余部分，我们将使用处理程序。
我们可以使用钩子，但这样会更易于阅读。
在几行代码中使用`beforeerequest`， `beforeError`和`afterResponse`钩子会使事情不必要地复杂化。

!!! tip

      - 当你的插件变得复杂时，使用钩子是一个很好的实践。
      - 尽量不要重载处理程序函数，但也不要滥用钩子。

```js
import got from "got";

const instance = got.extend({
  prefixUrl: "https://api.github.com",
  headers: {
    accept: "application/vnd.github.v3+json",
  },
  responseType: "json",
  context: {
    token: process.env.GITHUB_TOKEN,
  },
  hooks: {
    init: [
      (raw, options) => {
        if ("token" in raw) {
          options.context.token = raw.token;
          delete raw.token;
        }
      },
    ],
  },
  handlers: [
    (options, next) => {
      // Authorization
      const { token } = options.context;
      if (token && !options.headers.authorization) {
        options.headers.authorization = `token ${token}`;
      }

      return next(options);
    },
  ],
});

export default instance;
```

## 错误

我们应该为错误命名，以便知道错误是否来自 API 响应。超级错误，我们来了!

```js
...
	handlers: [
		(options, next) => {
			// Authorization
			const {token} = options.context;
			if (token && !options.headers.authorization) {
				options.headers.authorization = `token ${token}`;
			}

			// Don't touch streams
			if (options.isStream) {
				return next(options);
			}

			// Magic begins
			return (async () => {
				try {
					const response = await next(options);

					return response;
				} catch (error) {
					const {response} = error;

					// Nicer errors
					if (response && response.body) {
						error.name = 'GitHubError';
						error.message = `${response.body.message} (${response.statusCode} status code)`;
					}

					throw error;
				}
			})();
		}
	]
...
```

注意，通过在处理程序中提供我们自己的错误，我们不会改变`beforeError`钩子中的错误。  
转换是最后一件事。

## 速度限制

嗯…… `response.headers['x-ratelimit-remaining']` 看起来不太好。 
用`response.rateLimit.limit`代替怎么样?  
是的,当然。因为`response.headers`是一个对象，我们可以很容易地解析这些:

```js
const getRateLimit = (headers) => ({
  limit: Number.parseInt(headers["x-ratelimit-limit"], 10),
  remaining: Number.parseInt(headers["x-ratelimit-remaining"], 10),
  reset: new Date(Number.parseInt(headers["x-ratelimit-reset"], 10) * 1000),
});

getRateLimit({
  "x-ratelimit-limit": "60",
  "x-ratelimit-remaining": "55",
  "x-ratelimit-reset": "1562852139",
});
// => {
// 	limit: 60,
// 	remaining: 55,
// 	reset: 2019-07-11T13:35:39.000Z
// }
```

我们来积分一下:

```js
const getRateLimit = (headers) => ({
	limit: Number.parseInt(headers['x-ratelimit-limit'], 10),
	remaining: Number.parseInt(headers['x-ratelimit-remaining'], 10),
	reset: new Date(Number.parseInt(headers['x-ratelimit-reset'], 10) * 1000)
});

...
	handlers: [
		(options, next) => {
			// Authorization
			const {token} = options.context;
			if (token && !options.headers.authorization) {
				options.headers.authorization = `token ${token}`;
			}

			// Don't touch streams
			if (options.isStream) {
				return next(options);
			}

			// Magic begins
			return (async () => {
				try {
					const response = await next(options);

					// Rate limit for the Response object
					response.rateLimit = getRateLimit(response.headers);

					return response;
				} catch (error) {
					const {response} = error;

					// Nicer errors
					if (response && response.body) {
						error.name = 'GitHubError';
						error.message = `${response.body.message} (${response.statusCode} status code)`;
					}

					// Rate limit for errors
					if (response) {
						error.rateLimit = getRateLimit(response.headers);
					}

					throw error;
				}
			})();
		}
	]
...
```

## 蛋糕上的糖霜: `User-Agent`头。

```js
const packageJson = {
	name: 'gh-got',
	version: '12.0.0'
};

const instance = got.extend({
	...
	headers: {
		accept: 'application/vnd.github.v3+json',
		'user-agent': `${packageJson.name}/${packageJson.version}`
	},
	...
});
```

## 哇。就是这样吗？

是的。查看完整的源代码[在这里](examples/gh-got.js)。下面是如何使用它的一个例子:

```js
import ghGot from "gh-got";

const response = await ghGot("users/sindresorhus");
const creationDate = new Date(response.created_at);

console.log(
  `Sindre's GitHub profile was created on ${creationDate.toGMTString()}`
);
// => Sindre's GitHub profile was created on Sun, 20 Dec 2009 22:57:02 GMT
```

## 分页

```js
import ghGot from "gh-got";

const countLimit = 50;
const pagination = ghGot.paginate("repos/sindresorhus/got/commits", {
  pagination: { countLimit },
});

console.log(`Printing latest ${countLimit} Got commits (newest to oldest):`);

for await (const commitData of pagination) {
  console.log(commitData.commit.message);
}
```

这是……惊人的!我们不必自己实现分页。一切都搞定了。

## 在最后

您知道可以将许多实例混合到一个更大、更强大的实例中吗?查看[高级创作](examples/advanced-creation.js)指南。
