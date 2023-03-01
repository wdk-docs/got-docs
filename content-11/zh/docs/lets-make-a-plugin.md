---
title: "让我们做一个插件！"
linkTitle: "制作插件"
weight: 23
description: >
  关于如何像老板一样使用`Got`的另一个例子 :electric_plug:
type: "docs"
---

好了，你已经学会了一些基本知识。那很棒！

当涉及到高级的用法，自定义实例是真的很有帮助。
例如，看一看[`gh-got`](https://github.com/sindresorhus/gh-got).
它看起来很复杂，但是......它真的不是。

在我们开始之前，我们需要找到[GitHub 的 API 文档](https://developer.github.com/v3/).

让我们写下来的最重要的信息:

1. 根端点 `https://api.github.com/`.
2. 我们将使用 API​​ 版本 3. 该`Accept`头需要被设置为 `application/vnd.github.v3+json`.
3. 体是一个 JSON 格式。
4. 我们将使用 OAuth2 以授权。
5. 我们可能会收到 `400 Bad Request` 或者 `422 Unprocessable Entity`.正文包含有关错误的详细信息.
6. _Pagination?_ 还没. 这将是 Got`的`一项内置功能。 当该功能推出，我们会相应地更新这个页面。
7. 速率限制。这些标题很有意思:

   - `X-RateLimit-Limit`
   - `X-RateLimit-Remaining`
   - `X-RateLimit-Reset`
   - `X-GitHub-Request-I 另外`X-GitHub-Request-Id`可能是有用的。

8. User-Agent 是必须的.

   当我们拥有所有必要的信息，我们就可以开始混合 :cake:

## 根端点

没有太多的在这里做的，只是扩展一个实例，并提供了`prefixUrl`选项:

```js
const got = require("got");

const instance = got.extend({
	prefixUrl: "https://api.github.com"
});

module.exports = instance;
```

## v3 API

GitHub 上需要知道我们正在使用的版本。为这个我们将使用`Accept`头:

```js
const got = require("got");

const instance = got.extend({
	prefixUrl: "https://api.github.com",
	headers: {
		accept: "application/vnd.github.v3+json"
	}
});

module.exports = instance;
```

## JSON body

我们将使用[`options.responseType`](../readme.md#responsetype):

```js
const got = require("got");

const instance = got.extend({
	prefixUrl: "https://api.github.com",
	headers: {
		accept: "application/vnd.github.v3+json"
	},
	responseType: "json"
});

module.exports = instance;
```

## 授权

这是常见设置一些环境变量，例如， `GITHUB_TOKEN`.
您可以修改您的所有应用程序中轻松的令牌，对不对？
凉。怎么样......我们希望为每个应用程序的唯一标记。
然后，我们需要创建一个新的选项 - 它会默认为环境变量，但你可以很容易地覆盖它。

让我们用处理程序，而不是钩。
这将使我们的代码更易读: 有 `beforeRequest`, `beforeError` 和 `afterResponse` 挂钩 对代码短短的几行 将事情不必要的复杂.

**提示:** 这是一个很好的做法是使用挂钩时，你的插件变得复杂。 尽量不要超负荷处理函数，但不要滥用挂钩无论是。

```js
const got = require("got");

const instance = got.extend({
	prefixUrl: "https://api.github.com",
	headers: {
		accept: "application/vnd.github.v3+json"
	},
	responseType: "json",
	token: process.env.GITHUB_TOKEN,
	handlers: [
		(options, next) => {
			// Authorization
			if (options.token && !options.headers.authorization) {
				options.headers.authorization = `token ${options.token}`;
			}

			return next(options);
		}
	]
});

module.exports = instance;
```

## 错误

我们应该命名我们的错误，只知道如果错误是从 API 响应。 精湛的错误，我们来了！

```js
...
	handlers: [
		(options, next) => {
			// Authorization
			if (options.token && !options.headers.authorization) {
				options.headers.authorization = `token ${options.token}`;
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

## 速率限制

嗯... `response.headers['x-ratelimit-remaining']` 不好看.
关于什么 `response.rateLimit.limit` 代替?<br>
是的，绝对。 以来 `response.headers` 是一个对象, 我们可以很容易地分析这些:

```js
const getRateLimit = headers => ({
	limit: parseInt(headers["x-ratelimit-limit"], 10),
	remaining: parseInt(headers["x-ratelimit-remaining"], 10),
	reset: new Date(parseInt(headers["x-ratelimit-reset"], 10) * 1000)
});

getRateLimit({
	"x-ratelimit-limit": "60",
	"x-ratelimit-remaining": "55",
	"x-ratelimit-reset": "1562852139"
});
// => {
// 	limit: 60,
// 	remaining: 55,
// 	reset: 2019-07-11T13:35:39.000Z
// }
```

让我们来整合它:

```js
const getRateLimit = (headers) => ({
	limit: parseInt(headers['x-ratelimit-limit'], 10),
	remaining: parseInt(headers['x-ratelimit-remaining'], 10),
	reset: new Date(parseInt(headers['x-ratelimit-reset'], 10) * 1000)
});

...
	handlers: [
		(options, next) => {
			// Authorization
			if (options.token && !options.headers.authorization) {
				options.headers.authorization = `token ${options.token}`;
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

## 蛋糕上的糖霜: `User-Agent` header.

```js
const package = require('./package');

const instance = got.extend({
	...
	headers: {
		accept: 'application/vnd.github.v3+json',
		'user-agent': `${package.name}/${package.version}`
	}
	...
});
```

## 哇。是不是这样？

对。 查看完整的源代码[这里](../../examples/gh-got.js). 这里有一个如何使用它的一个例子:

```js
const ghGot = require("gh-got");

(async () => {
	const response = await ghGot("users/sindresorhus");
	const creationDate = new Date(response.created_at);

	console.log(`Sindre的GitHub的个人资料被 ${creationDate.toGMTString()}创建`);
	// => Sindre's GitHub profile was created on Sun, 20 Dec 2009 22:57:02 GMT
})();
```

你知道你可以在许多情况下，组合成一个更大的，更强大吗？ 查看[扩展](./api/extend.md)指南.
