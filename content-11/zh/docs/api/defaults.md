---
title: "defaults"
linkTitle: "defaults"
weight: 7
description: >
  在实例中使用的`Got`默认值。
type: "docs"
---

## got.defaults

**类型:** `object`

## handlers

**类型:** `Function[]`\
**默认:** `[]`

函数的数组. 您可以直接调用`got()`执行。 They are some sort of "global hooks" - these functions are called first. The last handler (_it's hidden_) is either [`asPromise`](source/as-promise.ts) or [`asStream`](source/as-stream.ts), depending on the `options.isStream` property.

每个处理器有两个参数:

## next()

返回一个`Promise`或`Stream`取决于[`options.isStream`](#isstream).

```js
const settings = {
	handlers: [
		(options, next) => {
			if (options.isStream) {
				// It's a Stream, so we can perform stream-specific actions on it
				return next(options).on("request", request => {
					setTimeout(() => {
						request.abort();
					}, 50);
				});
			}

			// It's a Promise
			return next(options);
		}
	],
	options: got.mergeOptions(got.defaults.options, {
		responseType: "json"
	})
};

const jsonGot = got.extend(settings);
```

## mutableDefaults

**类型:** `boolean`\
**默认:** `false`

只读布尔描述默认值是否可变与否。 如果设置为`true`, 您可以[随着时间的推移更新头](#hooksafterresponse), 例如, 到期时更新的访问令牌.
