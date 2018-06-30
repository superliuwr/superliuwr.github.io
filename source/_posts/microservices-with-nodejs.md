---
title: Microservices with Node.js
date: 2018-06-28 23:42:14
categories:
- Microservices
tags:
- Microservices
---
# Microservices

## Framework

## Container

## Database

## Service Gateway

## Middlewares
### Pub/Sub

### Queue

## Authentication and Authorisation

## Reliability

## Health Check

## Service Registry And Discovery

## Load Balancing

## Configuration centre

## Logging

Use [Bunyan](https://github.com/trentm/node-bunyan) to log structured records.

Bunyan log records are JSON. A few fields are added automatically: "pid", "hostname", "time" and "v".

```javascript
// Log with an optional 'fields' object as the first parameter
log.info({foo: 'bar', err: err}, 'some msg about this error');
```

Bunyan has a concept of a child logger to specialize a logger for a sub-component of your application, i.e. to create a new logger with additional bound fields that will be included in its log records. A child logger is created with `log.child(...)`.

Bunyan has a concept of "serializer" functions to produce a JSON-able object from a JavaScript object, so you can easily do the following:

`log.info({req: <request object>}, 'something about handling this request');`

and have the req entry in the log record be just a reasonable subset of <request object> fields (or computed data about those fields).

### 收集渠道

1. 系统未捕获错误，埋点如下，输出级别为 error。
```javascript
process.on('uncaughtException', processErrorHandler);  
process.on('unhandledRejection', processErrorHandler);
```
2. 程序内 byLog 的输出
3. console.error 等于 byLog.error

针对 Express 可做如下埋点
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

### Set traceToken or reqId

## Metrics

## Monitoring

## Circuit Breaker

## Distributed Tracing

# Key Points to validate

* How many Microservices should I divide the business capabilities into thinking in terms of domain driven design.
* Partitioning the data into different services with their own database solution with the understanding of which segment of data needs to be kept redundant locally for each of these services.
* How data is synchronized between Microservices for redundant data.
* Consistency requirements of data. (Whether its transactional requiring strong consistency vs possibility of implementing eventual consistency).
* Selection of right interfaces, channels, and middleware for communications and interaction between services.
* Required level of Security, Reliability, Performance, and Efficiency thinking the costs in mind.
* Tools and technologies to be used for individual Microservices. (Serverless, Containers, Web Technologies & etc.)
* How to version and share code (If required) in the form of libraries using repositories like NPM.
* How the teams and processes (Development, DevOps, Change Management & etc.) are structured around Microservices.

# References
