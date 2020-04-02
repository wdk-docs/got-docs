---
title: "Got 文档"
linkTitle: "文档"
weight: 1
type: "docs"
---

[![Got](./media/logo.svg)](https://github.com/sindresorhus/got)

巨大的感谢[MOXY](https://moxy.studio)赞助的 Sindre Sorhus！
![](https://sindresorhus.com/assets/thanks/moxy-logo.svg)

> 对于使用 Node.js 的人友好，功能强大的 HTTP 请求库

[![Build Status: Linux](https://travis-ci.org/sindresorhus/got.svg?branch=master)](https://travis-ci.org/sindresorhus/got)
[![Coverage Status](https://coveralls.io/repos/github/sindresorhus/got/badge.svg?branch=master)](https://coveralls.io/github/sindresorhus/got?branch=master)
[![Downloads](https://img.shields.io/npm/dm/got.svg)](https://npmjs.com/got)
[![Install size](https://packagephobia.now.sh/badge?p=got)](https://packagephobia.now.sh/result?p=got)

[从 Request 搬家？](documentation/migration-guides.md) [_(请注意，Request 是没有维护)_](https://github.com/request/request/issues/3142)

[看看是如何比较其他 HTTP 库](#comparison)

对于浏览器的使用，我们建议[Ky](https://github.com/sindresorhus/ky)由同一人。

## 强调

- [Promise API](#api)
- [Stream API](#streams)
- [分页 API (试验)](#pagination)
- [请求取消](#aborting-the-request)
- [符合 RFC 缓存](#cache-adapters)
- [遵循重定向](#followredirect)
- [失败重试](#retry)
- [事件进展](#onuploadprogress-progress)
- [手柄 gzip/deflate/brotli](#decompress)
- [超时处理](#timeout)
- [使用元数据错误](#errors)
- [JSON 模式](#json-mode)
- [WHATWG 的 URL 支持](#url)
- [钩](#hooks)
- [自定义默认实例](#instances)
- [类型](#types)
- [可组合](documentation/advanced-creation.md#merging-instances)
- [插件](documentation/lets-make-a-plugin.md)
- [由 3000+包和 1.6M +repos 使用](https://github.com/sindresorhus/got/network/dependents)
- 积极维护
