---
title: "UNIX域套接字"
linkTitle: ""
weight: 11
type: "docs"
---

请求也可以通过[UNIX 域套接字](http://serverfault.com/questions/124517/whats-the-difference-between-unix-socket-and-tcp-ip-socket)发送。 使用下面的 URL 方案: `PROTOCOL://unix:SOCKET:PATH`.

- `PROTOCOL` - `http` 要么 `https` _(optional)_
- `SOCKET` - 绝对路径中的 UNIX 域套接字，例如: `/var/run/docker.sock`
- `PATH` - 请求路径，例如: `/v2/keys`

```js
const got = require("got");

got("http://unix:/var/run/docker.sock:/containers/json");

// 或没有协议（HTTP默认情况下）
got("unix:/var/run/docker.sock:/containers/json");
```
