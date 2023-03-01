---
title: "合并选项-mergeOptions"
linkTitle: "合并选项"
weight: 6
description: >
  扩展父选项。避免使用[对象蔓延](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#Spread_in_object_literals)因为它没有递归工作
type: "docs"
---

## got.mergeOptions(parentOptions, newOptions)

```js
const a = {headers: {cat: 'meow', wolf: ['bark', 'wrrr']}};
const b = {headers: {cow: 'moo', wolf: ['auuu']}};

{...a, ...b}            // => {headers: {cow: 'moo', wolf: ['auuu']}}
got.mergeOptions(a, b)  // => {headers: {cat: 'meow', cow: 'moo', wolf: ['auuu']}}
```

选项都深深合并为一个新的对象。每个键的值被确定如下:

- If the new property is set to `undefined`, it keeps the old one.
- If both properties are an instances of `URLSearchParams`, a new URLSearchParams instance is created. The values are merged using [`urlSearchParams.append(key, value)`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams/append).
- If the parent property is an instance of `URL` and the new value is a `string` or `URL`, a new URL instance is created: [`new URL(new, parent)`](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL#Syntax).
- If the new property is a plain `object`: - If the parent property is a plain `object` too, both values are merged recursively into a new `object`. - Otherwise, only the new value is deeply cloned.
- If the new property is an `Array`, it overwrites the old one with a deep clone of the new property.
- Properties that are not enumerable, such as `context`, `body`, `json`, and `form`, will not be merged.

  ```js
  const a = { json: { cat: "meow" } };
  const b = { json: { cow: "moo" } };

  got.mergeOptions(a, b);
  //=> {json: {cow: 'moo'}}
  ```

- 否则，新值分配给键。
