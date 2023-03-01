---
title: "分页-paginate"
linkTitle: "分页"
weight: 4
description: >
  返回一个异步迭代器
type: "docs"
---

## got.paginate(url, options?)

```js
(async () => {
	const countLimit = 10;

	const pagination = got.paginate(
		"https://api.github.com/repos/sindresorhus/got/commits",
		{
			_pagination: { countLimit }
		}
	);

	console.log(`Printing latest ${countLimit} Got commits (newest to oldest):`);

	for await (const commitData of pagination) {
		console.log(commitData.commit.message);
	}
})();
```

See [`options._pagination`](#_pagination) for more pagination options.

## \_pagination

类型: `object`

{{% alert color="primary" %}}
This feature is marked as experimental as we're [looking for feedback](https://github.com/sindresorhus/got/issues/1052) on the API and how it works. The feature itself is stable, but the API may change based on feedback. So if you decide to try it out, we suggest locking down the `got` dependency semver range or use a lockfile.
{{% /alert %}}

## \_pagination.transform

类型: `Function`\
默认: `response => JSON.parse(response.body)`

A function that transform [`Response`](#response) into an array of items. This is where you should do the parsing.

## \_pagination.paginate

类型: `Function`\
默认: [`Link` header logic](source/index.ts)

该函数有三个参数:

- `response` - 流响应对象.
- `allItems` - 所发射的项目的数组.
- `currentItems` - 从目前的反应项目.

It should return an object representing Got options pointing to the next page. If there are no more pages, `false` should be returned.

For example, if you want to stop when the response contains less items than expected, you can use something like this:

```js
const got = require("got");

(async () => {
	const limit = 10;

	const items = got.paginate("https://example.com/items", {
		searchParams: {
			limit,
			offset: 0
		},
		_pagination: {
			paginate: (response, allItems, currentItems) => {
				const previousSearchParams = response.request.options.searchParams;
				const { offset: previousOffset } = previousSearchParams;

				if (currentItems.length < limit) {
					return false;
				}

				return {
					searchParams: {
						...previousSearchParams,
						offset: previousOffset + limit
					}
				};
			}
		}
	});

	console.log("Items from all pages:", items);
})();
```

## \_pagination.filter

类型: `Function`\
默认: `(item, allItems, currentItems) => true`

检查项目是否应该被发射与否。

## \_pagination.shouldContinue

类型: `Function`\
默认: `(item, allItems, currentItems) => true`

检查分页是否应该继续。

For example, if you need to stop **before** emitting an entry with some flag, you should use `(item, allItems, currentItems) => !item.flag`. If you want to stop **after** emitting the entry, you should use `(item, allItems, currentItems) => allItems.some(entry => entry.flag)` instead.

## \_pagination.countLimit

类型: `number`\
默认: `Infinity`

应发出项目的最高额度。
