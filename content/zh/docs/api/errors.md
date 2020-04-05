---
title: "错误方法"
linkTitle: "错误方法"
weight: 10
description: >
  每个错误包含一个`options`属性 这是`Got`用于创建一个请求的选项 - 只是为了让调试更加容易。
type: "docs"
---

## got.CacheError

当缓存方法失败，例如，如果数据库出现故障或有一个文件系统错误。

## got.RequestError

当一个请求失败。 包含带错误类代码的`code`属性， 如 `ECONNREFUSED`.

## got.ReadError

从反应流中读取时失败。

## got.ParseError

当服务器响应代码为 2XX，和解析体失败。 包括`response`属性。

## got.HTTPError

当服务器响应代码是不是 2XX。 包括`response`属性。

## got.MaxRedirectsError

当服务器重定向你十次以上。 包括`response`属性。

## got.UnsupportedProtocolError

当给定的协议不支持。

## got.CancelError

当该请求被`.cancel()`中止。

## got.TimeoutError

当该请求由于[超时](#timeout)被中止. 包括`event`和`timings`属性。
