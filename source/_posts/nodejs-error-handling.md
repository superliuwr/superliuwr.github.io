---
title: Node.js Error Handling Best Practices
date: 2018-07-01 11:09:47
categories:
- Node.js
tags:
- Node.js
- Error Handling
- Best Practices
---
# Error hanlding

## Error types

In Node, errors occur any of the following ways:

<!-- more -->

* Explicit exceptions (those triggered by the throw keyword)
* Implicit exceptions (like ReferenceError: foo not defined)
* The ‘error’ event (which may trigger an exception)
* The error callback argument (no exceptions): pass the error to a callback, a function provided specifically for handling errors and the results of asynchronous operations
* pass the error to a reject Promise function

在 Node.js 中错误处理主要有一下几种方法:

* callback(err, data) 回调约定
* throw / try / catch
* EventEmitter 的 error 事件

callback(err, data) 这种形式的错误处理起来繁琐, 并不具备强制性, 目前已经处于仅需要了解, 不推荐使用的情况. 而 domain 模块则是半只脚踏进棺材了.

感谢 co 的先河, 现在的你已经简单的使用 try/catch 保护关键的位置, 以 koa 为例, 可以通过中间件的形式来进行错误处理, 详见 Koa error handling. 之后的 async/await 均属于这种模式.

通过 EventEmitter 的错误监听形式为各大关键的对象加上错误监听的回调. 例如监听 http server, tcp server 等对象的 error 事件以及 process 对象提供的 uncaughtException 和 unhandledRejection 等等.

使用 Promise 来封装异步, 并通过 Promise 的错误处理来 handle 错误.

如果上述办法不能起到良好的作用, 那么你需要学习如何优雅的 Let It Crash

#### The uncaught exception

A Node process will terminate on any uncaught exception (explicit or implicit).

However, you can override this behavior by adding an uncaughtException handler on the process object. The following illustrates programmatically what Node does on your behalf when an uncaught exception occurs:

```javascript
process.on('uncaughtException', function (er) {
  console.error(er.stack)
  process.exit(1)
})
```

An uncaughtException handler should be treated as a last opportunity to say your goodbyes before calling process.exit. It is not advised to keep the process running.

### The unhandled rejection

Promises are ubiquitous in Node.js code and sometimes chained to a very long list of functions that return promises and so on. Not using a proper .catch(…) rejection handler will cause an unhandledRejection event to be emitted, and if not properly caught and inspected, you may rob yourself of your only chance to detect and possibly fix the problem. Here is how you can set up a listener:

```javascript
process.on('unhandledRejection', function (reason, promise) {
    logger.error('Unhandled rejection', {reason: reason, promise: promise})
})
```

It is a good practice to set up a listener for an unhandledRejection event and log or even count the number of occurrences just to know what happened in your system and inspect possible instabilities due to improper rejection handling.

### The infamous ‘error’ event

Any EventEmitter can potentially emit an ‘error’ event and there are multiple objects that inherit from EventEmitters in Node (core and 3rd-party modules). The ‘error’ events emitted in Node core come from objects such as:

* Streams
* Servers
* Requests/Responses
* Child processes

Node treats this as a special event. If left unhandled, it will throw an exception (instead of silently ignoring the error).

Many 3rd-party modules will bubble up ‘error’ events or other errors from Node core modules as well as emit their own.

### Catching implicit exceptions

We can’t forget about common implicit exceptions. A great example of this is the SyntaxError thrown when using JSON.parse: `JSON.parse('undefined')`

These errors are avoided by a simple try/catch block:

```javasceript
try {
  JSON.parse(maybeJSON)
} catch (er) {
  console.error('Invalid JSON', er)
}
```

## Operational errors vs. programmer errors

Operational errors represent run-time problems experienced by correctly-written programs. These are not bugs in the program. In fact, these are usually problems with something else: the system itself (e.g., out of memory or too many open files), the system's configuration (e.g., no route to a remote host), the network (e.g., socket hang-up), or a remote service (e.g., a 500 error, failure to connect, or the like). 

Programmer errors are bugs in the program. These are things that can always be avoided by changing the code. They can never be handled properly (since by definition the code in question is broken).

Operational errors are error conditions that all correct programs must deal with, and as long as they're dealt with, they don't necessarily indicate a bug or even a serious problem.

By contrast, programmer errors are bugs. They're cases where you made a mistake, maybe by forgetting to validate user input, mistyping a variable name, or something like that. By definition there's no way to handle those.

This distinction is very important: operational errors are part of the normal operation of a program. Programmer errors are bugs.

### Handling operational errors

You may end up handling the same error at several levels of the stack. This happens when lower levels can't do anything useful except propagate the error to their caller, which propagates the error to its caller, and so on. Often, only the top-level caller knows what the appropriate response is, whether that's to retry the operation, report an error to the user, or something else. But that doesn't mean you should try to report all errors to a single top-level callback, because that callback itself can't know in what context the error occurred, what pieces of an operation have successfully completed, and which ones actually failed.

Let's make this concrete. For any given error, there are a few things you might do:

* Deal with the failure directly
* Propagate the failure to your client
* Retry the operation
* Blow up
* Log the error — and do nothing else

### (Not) handling programmer errors

Some people advocate attempting to recover from programmer errors — that is, allow the current operation to fail, but keep handling requests. This is not recommended. Consider that a programmer error is a case that you didn't think about when you wrote the original code. How can you be sure that the problem won't affect other requests? If other requests share any common state (a server, a socket, a pool of database connections, etc.), it's very possible that the other requests will do the wrong thing.

The best way to recover from programmer errors is to crash immediately. You should run your programs using a restarter that will automatically restart the program in the event of a crash. With a restarter in place, crashing is the fastest way to restore reliable service in the face of a transient programmer error.

The best way to debug these problems is to configure Node to dump core on an uncaught exception.

Finally, remember that a programmer error on a server just becomes an operational error on a client. Clients have to deal with servers crashing and network blips. That's not just theoretical — both really do happen in production systems.

### Patterns for writing functions

The single most important thing to do is document what your function does, including what arguments it takes (including their types and any other constraints), what it returns, what errors can happen, and what those errors mean. If you don't know what errors can happen or don't know what they mean, then your program cannot be correct except by accident. So if you're writing a new function, you have to tell your callers what errors can happen and what they mean.

The general rule is that a function may deliver operational errors synchronously (e.g., by throwing) or asynchronously (by passing them to a callback or emitting error on an EventEmitter), but it should not do both.

#### Bad input: programmer error or operational error?

How do you know what's a programmer error vs. an operational error? Quite simply: it's up to you to define and document what types your function will allow and how you'll try to interpret them. If you get something other than what you've documented to accept, that's a programmer error. If the input is something you've documented to accept but you can't process right now, that's an operational error.

You have to use your judgment to decide how strict you want to be, but we can make some suggestions. To get specific, imagine a function called "connect" that takes an IP address and a callback and invokes the callback asynchronously after either succeeding or failing. Suppose the user passes something that's obviously not a valid IP address, like 'bob'. In this case, you have a few options:

Document that the function only accepts strings representing valid IPv4 addresses, and throw an exception immediately if the user passes 'bob'. This is strongly recommended.
Document that the function accepts any string. If the user passes 'bob', emit an asynchronous error indicating that you couldn't connect to IP address 'bob'.

#### Specific recommendations for writing new functions

* Be clear about what your function does.
* Use Error objects (or subclasses) for all errors, and implement the Error contract.
* Use the Error's name property to distinguish errors programmatically.
* Augment the Error object with properties that explain details
  * 通过使用 verror 这样的方式, 让 Error 一层层封装, 并在每一层将错误的信息一层层的包上, 最后拿到的 Error 直接可以从 message 中获取用于定位问题的关键信息.
* If you pass a lower-level error to your caller, consider wrapping it instead.

An example:

```javascript
/*
 * Make a TCP connection to the given IPv4 address.  Arguments:
 *
 *    ip4addr        a string representing a valid IPv4 address
 *
 *    tcpPort        a positive integer representing a valid TCP port
 *
 *    timeout        a positive integer denoting the number of milliseconds
 *                   to wait for a response from the remote server before
 *                   considering the connection to have failed.
 *
 *    callback       invoked when the connection succeeds or fails.  Upon
 *                   success, callback is invoked as callback(null, socket),
 *                   where `socket` is a Node net.Socket object.  Upon failure,
 *                   callback is invoked as callback(err) instead.
 *
 * This function may fail for several reasons:
 *
 *    SystemError    For "connection refused" and "host unreachable" and other
 *                   errors returned by the connect(2) system call.  For these
 *                   errors, err.errno will be set to the actual errno symbolic
 *                   name.
 *
 *    TimeoutError   Emitted if "timeout" milliseconds elapse without
 *                   successfully completing the connection.
 *
 * All errors will have the conventional "remoteIp" and "remotePort" properties.
 * After any error, any socket that was created will be closed.
 */
function connect(ip4addr, tcpPort, timeout, callback) {
  assert.equal(typeof (ip4addr), 'string',
      "argument 'ip4addr' must be a string");
  assert.ok(net.isIPv4(ip4addr),
      "argument 'ip4addr' must be a valid IPv4 address");
  assert.equal(typeof (tcpPort), 'number',
      "argument 'tcpPort' must be a number");
  assert.ok(!isNaN(tcpPort) && tcpPort > 0 && tcpPort < 65536,
      "argument 'tcpPort' must be a positive integer between 1 and 65535");
  assert.equal(typeof (timeout), 'number',
      "argument 'timeout' must be a number");
  assert.ok(!isNaN(timeout) && timeout > 0,
      "argument 'timeout' must be a positive integer");
  assert.equal(typeof (callback), 'function');

  /* do work */
}
```

你应该总是抛出一个继承自 JavaScript 内建的 Error 类型的对象，而不要抛出 String 或普通的 Object, 因为只有语言内建的 Error 对象上才会有调用栈，抛出其他类型的对象将可能会导致调用栈无法正确地被记录。同时也要慎重地使用自定义的异常类型，因为目前 JavaScript 中和调用栈有关的 API（如 Error.captureStackTrace）还不在标准中，各个引擎的实现也不同，你很难写出一个在所有引擎都可用的自定义异常类型。因此如果你的代码可能会同时运行在 Node.js 和浏览器中，或者你在编写一个开源项目，那么建议你不要使用自定义的异常类型；如果你的代码不是开源的，运行环境也非常确定，则可以考虑使用引擎提供的私有 API 来自定义异常类型。

传递异常的过程中的一些最佳实践：

* 注意 Promise / callback chain 不要从中间断开
* 只处理已知的、必须在这里处理的异常，其他异常继续向外抛出
* 不要轻易地丢弃一个异常
* 传递的过程中可以向 err 对象上添加属性，补充上下文

## Error handling with Express

Read this [article](https://thecodebarbarian.com/80-20-guide-to-express-error-handling).

## Other fun stuff

* [Go style error handling](https://medium.com/front-end-hacking/error-handling-in-node-javascript-suck-unless-you-know-this-2018-aa0a14cfdd9d)

## A checklist
Errors happen; its important to build structures to handle them. Use this checklist to buff up your code:

* Where am I using throw? Am I prepared to handle these explicit exceptions when they occur?
* Am I safeguarding against common sources for implicit exceptions (like JSON.parse, undefined data values in a nodeback)?
* Am I handling ‘error’ events on all EventEmitters?
* Am I handling all error arguments in nodebacks?
* Am I notified of uncaught exceptions?

# References

* [Error handling in Node.js](https://www.joyent.com/node-js/production/design/errors)
* [Checklist: Best Practices of Node.JS Error Handling (2018)](https://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/)
* [ElemeFE/node-interview](https://github.com/ElemeFE/node-interview/blob/master/sections/zh-cn/error.md)