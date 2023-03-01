---
title: "类型"
linkTitle: ""
weight: 3
type: "docs"
---

`Got`输出一些方便`TypeScript`类型和接口。 为所有导出类型请查看类型定义。

## Got

`TypeScript`将自动推断类型`Got`实例，但如果你要定义类似的依赖，你可以从`Got`直接导入可用的类型。

```ts
import { GotRequestMethod } from "got";

interface Dependencies {
	readonly post: GotRequestMethod;
}
```

## 钩子

当写钩子，你可以参考它们的类型，让您的接口保持一致。

```ts
import { BeforeRequestHook } from "got";

const addAccessToken = (accessToken: string): BeforeRequestHook => options => {
	options.path = `${options.path}?access_token=${accessToken}`;
};
```
