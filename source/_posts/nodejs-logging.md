---
title: Node.js Logging Best Practices
date: 2018-07-01 10:04:37
categories:
- Node.js
tags:
- Node.js
- Logging
- Best Practices
---
 Logs can be a dumb warehouse of debug statements or the enabler of a beautiful dashboard that tells the story of your app. Plan your logging platform from day 1: how logs are collected, stored and analyzed to ensure that the desired information (e.g. error rate, following an entire transaction through services and servers, etc) can really be extracted.

## Bunyan

Use [Bunyan](https://github.com/trentm/node-bunyan) to log structured records. Alternatively you can use [pino](https://www.npmjs.com/package/pino).

Bunyan log records are JSON. A few fields are added automatically: "pid", "hostname", "time" and "v".

```javascript
// Log with an optional 'fields' object as the first parameter
log.info({foo: 'bar', err: err}, 'some msg about this error');
```

### Child Logger
Bunyan has a concept of a child logger to specialize a logger for a sub-component of your application, i.e. to create a new logger with additional bound fields that will be included in its log records. A child logger is created with `log.child(...)`.

Bunyan has a concept of "serializer" functions to produce a JSON-able object from a JavaScript object, so you can easily do the following:

`log.info({req: <request object>}, 'something about handling this request');`

and have the req entry in the log record be just a reasonable subset of <request object> fields (or computed data about those fields).

## 收集渠道

1. 系统未捕获错误:
```javascript
process.on('uncaughtException', processErrorHandler);  
process.on('unhandledRejection', processErrorHandler);
```
2. 程序内 byLog 的输出
3. console.error

针对 Express 可做如下埋点:

```javascript
// 在所有路由最后，添加错误处理，必须同时含有四个参数
// http://expressjs.com/en/guide/error-handling.html
app.use(function(err, req, res, next) {  
    res.send({
        code: 500,
        message: `System Error! Please send serial number ${req.x_request_id} to administer`
    });
    byLog.error(err, null, {
        req_id: req.x_request_id
    });
});
```

## Logging in Distributed Systems
### Have an Application Instance Identifier
Use the `name` field in bunyan.

### Always Use UTC Time

### Adding correlation IDs
This ID has to be passed around in function calls, and it has to be sent to downstream services too.

### Using Trace
https://trace.risingstack.com/

## Never, ever log credentials, passwords or any sensitive information.

## Logging in NPM modules

Use the [debug](https://www.npmjs.com/package/debug) module.

By default, it will not produce any output. To enable this logger, you have run your application with a special environment variable, called DEBUG.

`DEBUG=my-namespace,express* node app.js`

Once you do that, the debug module will come to life and will start producing log events for stdout.

## References

