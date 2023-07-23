# 捕获异步堆栈跟踪

**Caution:**

> - 捕获异步堆栈跟踪会严重降低性能!

**想跳过这篇文章?请参阅[Conclusion](#conclusion)，我们在其中讨论了一个普通的解决方案。**

我们生活在一个充满虫子的世界。
软件变得越来越复杂，这使得调试变得越来越困难。
你是否曾经犯过错误，却不知道错误来自哪里?是的，通常不容易找到。

您可能已经注意到，错误的`.stack`有时看起来不完整。
这通常是由于计时器触发的异步函数的执行。
示例如下:

```js
await new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error("here"));
  });
});
```

```js
file:///home/szm/Desktop/got/demo.js:3
                reject(new Error('here'));
                       ^

Error: here
    at Timeout._onTimeout (file:///home/szm/Desktop/got/demo.js:3:10)
    at listOnTimeout (node:internal/timers:557:17)
    at processTimers (node:internal/timers:500:7)
```

堆栈跟踪不会显示超时设置的位置。目前无法用原生的`Promise`来确定这一点。然而，[`bluebird`](https://github.com/petkaantonov/bluebird/)提供了一个专门用于捕获异步堆栈跟踪的选项:

```js
import Bluebird from "bluebird";

Bluebird.config({ longStackTraces: true });
global.Promise = Bluebird;

await new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error("here"));
  });
});
```

```
node:internal/process/esm_loader:74
    internalBinding('errors').triggerUncaughtException(
                              ^

Error: here
    at Timeout._onTimeout (file:///home/szm/Desktop/got/demo.js:7:10)
    at listOnTimeout (node:internal/timers:557:17)
    at processTimers (node:internal/timers:500:7)
From previous event:
    at file:///home/szm/Desktop/got/demo.js:5:7
    at ModuleJob.run (node:internal/modules/esm/module_job:183:25)
    at async Loader.import (node:internal/modules/esm/loader:178:24)
    at async Object.loadESM (node:internal/process/esm_loader:68:5)
    at async handleMainPromise (node:internal/modules/run_main:63:12)
```

现在很清楚了。我们知道超时设置在第5行。蓝鸟对Got来说应该足够了:

```js
import Bluebird from "bluebird";
import got from "got";

Bluebird.config({ longStackTraces: true });
global.Promise = Bluebird;

try {
  await got("https://httpbin.org/delay/1", {
    timeout: {
      request: 1,
    },
    retry: {
      limit: 0,
    },
  });
} catch (error) {
  console.error(error.stack);
}
```

```
TimeoutError: Timeout awaiting 'request' for 1ms
    at ClientRequest.<anonymous> (file:///home/szm/Desktop/got/dist/source/core/index.js:780:61)
    at Object.onceWrapper (node:events:514:26)
    at ClientRequest.emit (node:events:406:35)
    at TLSSocket.socketErrorListener (node:_http_client:447:9)
    at TLSSocket.emit (node:events:394:28)
    at emitErrorNT (node:internal/streams/destroy:157:8)
    at emitErrorCloseNT (node:internal/streams/destroy:122:3)
    at processTicksAndRejections (node:internal/process/task_queues:83:21)
    at Timeout.timeoutHandler [as _onTimeout] (file:///home/szm/Desktop/got/dist/source/core/timed-out.js:42:25)
    at listOnTimeout (node:internal/timers:559:11)
    at processTimers (node:internal/timers:500:7)
From previous event:
    at new PCancelable (file:///home/szm/Desktop/got/node_modules/p-cancelable/index.js:31:19)
    at asPromise (file:///home/szm/Desktop/got/dist/source/as-promise/index.js:21:21)
    at lastHandler (file:///home/szm/Desktop/got/dist/source/create.js:42:27)
    at iterateHandlers (file:///home/szm/Desktop/got/dist/source/create.js:49:28)
    at got (file:///home/szm/Desktop/got/dist/source/create.js:69:16)
    at file:///home/szm/Desktop/got/demo.js:8:8
    at ModuleJob.run (node:internal/modules/esm/module_job:183:25)
    at async Loader.import (node:internal/modules/esm/loader:178:24)
    at async Object.loadESM (node:internal/process/esm_loader:68:5)
    at async handleMainPromise (node:internal/modules/run_main:63:12)
```

正如预期的那样，我们知道在哪里设置了超时。
不幸的是，如果我们将重试计数限制增加到`1`，堆栈跟踪将保持不变。
这是因为`bluebird`不跟踪I/O事件。请注意，这对于大多数情况应该是足够的。
为了进一步调试，我们可以使用[`async_hooks`](https://nodejs.org/api/async_hooks.html)来代替。
一位Stack Overflow用户提出了一个很棒的解决方案:

```js
import asyncHooks from "async_hooks";

const traces = new Map();

asyncHooks
  .createHook({
    init(id) {
      const trace = {};
      Error.captureStackTrace(trace);
      traces.set(id, trace.stack.replace(/(^.+$\n){4}/m, "\n"));
    },
    destroy(id) {
      traces.delete(id);
    },
  })
  .enable();

globalThis.Error = class extends Error {
  constructor(message) {
    super(message);
    this.constructor.captureStackTrace(this, this.constructor);
  }

  static captureStackTrace(what, where) {
    super.captureStackTrace.call(Error, what, where);

    const trace = traces.get(asyncHooks.executionAsyncId());
    if (trace) {
      what.stack += trace;
    }
  }
};
```

如果我们把`bluebird`部分替换成这个，我们得到:

```
Error: Timeout awaiting 'request' for 1ms
    at ClientRequest.<anonymous> (file:///home/szm/Desktop/got/dist/source/core/index.js:780:61)
    at Object.onceWrapper (node:events:514:26)
    at ClientRequest.emit (node:events:406:35)
    at TLSSocket.socketErrorListener (node:_http_client:447:9)
    at TLSSocket.emit (node:events:394:28)
    at emitErrorNT (node:internal/streams/destroy:157:8)
    at emitErrorCloseNT (node:internal/streams/destroy:122:3)
    at processTicksAndRejections (node:internal/process/task_queues:83:21)
    at emitInitScript (node:internal/async_hooks:493:3)
    at process.nextTick (node:internal/process/task_queues:133:5)
    at onDestroy (node:internal/streams/destroy:96:15)
    at TLSSocket.Socket._destroy (node:net:677:5)
    at _destroy (node:internal/streams/destroy:102:25)
    at TLSSocket.destroy (node:internal/streams/destroy:64:5)
    at ClientRequest.destroy (node:_http_client:371:16)
    at emitInitScript (node:internal/async_hooks:493:3)
    at initAsyncResource (node:internal/timers:162:5)
    at new Timeout (node:internal/timers:196:3)
    at setTimeout (node:timers:164:19)
    at addTimeout (file:///home/szm/Desktop/got/dist/source/core/timed-out.js:32:25)
    at timedOut (file:///home/szm/Desktop/got/dist/source/core/timed-out.js:59:31)
    at Request._onRequest (file:///home/szm/Desktop/got/dist/source/core/index.js:771:32)
    at emitInitScript (node:internal/async_hooks:493:3)
    at promiseInitHook (node:internal/async_hooks:323:3)
    at promiseInitHookWithDestroyTracking (node:internal/async_hooks:327:3)
    at Request.flush (file:///home/szm/Desktop/got/dist/source/core/index.js:274:24)
    at makeRequest (file:///home/szm/Desktop/got/dist/source/as-promise/index.js:125:30)
    at Request.<anonymous> (file:///home/szm/Desktop/got/dist/source/as-promise/index.js:121:17)
    at Object.onceWrapper (node:events:514:26)
    at emitInitScript (node:internal/async_hooks:493:3)
    at promiseInitHook (node:internal/async_hooks:323:3)
    at promiseInitHookWithDestroyTracking (node:internal/async_hooks:327:3)
    at file:///home/szm/Desktop/got/dist/source/core/index.js:357:27
    at processTicksAndRejections (node:internal/process/task_queues:96:5)
    at emitInitScript (node:internal/async_hooks:493:3)
    at promiseInitHook (node:internal/async_hooks:323:3)
    at promiseInitHookWithDestroyTracking (node:internal/async_hooks:327:3)
    at file:///home/szm/Desktop/got/dist/source/core/index.js:338:50
    at Request._beforeError (file:///home/szm/Desktop/got/dist/source/core/index.js:388:11)
    at ClientRequest.<anonymous> (file:///home/szm/Desktop/got/dist/source/core/index.js:781:18)
    at Object.onceWrapper (node:events:514:26)
    at emitInitScript (node:internal/async_hooks:493:3)
    at process.nextTick (node:internal/process/task_queues:133:5)
    at onDestroy (node:internal/streams/destroy:96:15)
    at TLSSocket.Socket._destroy (node:net:677:5)
    at _destroy (node:internal/streams/destroy:102:25)
    at TLSSocket.destroy (node:internal/streams/destroy:64:5)
    at ClientRequest.destroy (node:_http_client:371:16)
    at emitInitScript (node:internal/async_hooks:493:3)
    at initAsyncResource (node:internal/timers:162:5)
    at new Timeout (node:internal/timers:196:3)
    at setTimeout (node:timers:164:19)
    at addTimeout (file:///home/szm/Desktop/got/dist/source/core/timed-out.js:32:25)
    at timedOut (file:///home/szm/Desktop/got/dist/source/core/timed-out.js:59:31)
    at Request._onRequest (file:///home/szm/Desktop/got/dist/source/core/index.js:771:32)
    at emitInitScript (node:internal/async_hooks:493:3)
    at promiseInitHook (node:internal/async_hooks:323:3)
    at promiseInitHookWithDestroyTracking (node:internal/async_hooks:327:3)
    at Request.flush (file:///home/szm/Desktop/got/dist/source/core/index.js:274:24)
    at lastHandler (file:///home/szm/Desktop/got/dist/source/create.js:37:26)
    at iterateHandlers (file:///home/szm/Desktop/got/dist/source/create.js:49:28)
    at got (file:///home/szm/Desktop/got/dist/source/create.js:69:16)
    at Timeout.timeoutHandler [as _onTimeout] (file:///home/szm/Desktop/got/dist/source/core/timed-out.js:42:25)
    at listOnTimeout (node:internal/timers:559:11)
    at processTimers (node:internal/timers:500:7)
```

这是一个非常长的，并不是一个完整的Node.js应用程序，只是一个演示。想象一下，如果将它与数据库、文件系统等一起使用，将需要多长时间。

#### 结论

所有这些变通方法都对性能有很大的影响。
然而，这种疯狂有一个可能的解决方案。
Got提供了处理程序、钩子和上下文。
我们可以在处理程序中捕获堆栈跟踪，将其存储在一个上下文中，并将其暴露在`beforeError`钩子中。

```js
import got from "got";

const instance = got.extend({
  handlers: [
    (options, next) => {
      Error.captureStackTrace(options.context);
      return next(options);
    },
  ],
  hooks: {
    beforeError: [
      (error) => {
        error.source = error.options.context.stack.split("\n");
        return error;
      },
    ],
  },
});

try {
  await instance("https://httpbin.org/delay/1", {
    timeout: {
      request: 100,
    },
    retry: {
      limit: 0,
    },
  });
} catch (error) {
  console.error(error);
}
```

```
RequestError: Timeout awaiting 'request' for 100ms
    at ClientRequest.<anonymous> (file:///home/szm/Desktop/got/dist/source/core/index.js:780:61)
    at Object.onceWrapper (node:events:514:26)
    at ClientRequest.emit (node:events:406:35)
    at TLSSocket.socketErrorListener (node:_http_client:447:9)
    at TLSSocket.emit (node:events:394:28)
    at emitErrorNT (node:internal/streams/destroy:157:8)
    at emitErrorCloseNT (node:internal/streams/destroy:122:3)
    at processTicksAndRejections (node:internal/process/task_queues:83:21)
    at Timeout.timeoutHandler [as _onTimeout] (file:///home/szm/Desktop/got/dist/source/core/timed-out.js:42:25)
    at listOnTimeout (node:internal/timers:559:11)
    at processTimers (node:internal/timers:500:7) {
  input: undefined,
  code: 'ETIMEDOUT',
  timings: { <too long to include> },
  name: 'TimeoutError',
  options: { <too long to include> },
  event: 'request',
  source: [
    'Error',
    '    at got.extend.handlers (file:///home/szm/Desktop/got/demo.js:6:10)',
    '    at iterateHandlers (file:///home/szm/Desktop/got/dist/source/create.js:49:28)',
    '    at got (file:///home/szm/Desktop/got/dist/source/create.js:69:16)',
    '    at file:///home/szm/Desktop/got/demo.js:23:8',
    '    at ModuleJob.run (node:internal/modules/esm/module_job:183:25)',
    '    at async Loader.import (node:internal/modules/esm/loader:178:24)',
    '    at async Object.loadESM (node:internal/process/esm_loader:68:5)',
    '    at async handleMainPromise (node:internal/modules/run_main:63:12)'
  ]
}
```

耶!这样更容易读懂。此外，我们仅在调用`got`时捕获堆栈跟踪。
这肯定会对性能产生一些影响，但它比前面提到的其他解决方案的性能要好得多。

想知道更多吗?看看这些链接:

- https://stackoverflow.com/questions/54914770/is-there-a-good-way-to-surface-error-traces-in-production-across-event-emitters
- https://github.com/nodejs/node/issues/11370
- https://github.com/puppeteer/puppeteer/issues/2037
- https://github.com/nodejs/node/pull/13870
