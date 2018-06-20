---
title: Node.js Production Practices
date: 2018-06-19 10:10:18
categories:
- Node.js
tags:
- Node.js
- Best Practices
---

# Development

## Design

### Project Structure

### Handle Different Environments and Configurations

### Validation

### Use Messaging for Background Processes

If you are using HTTP for sending messages, then whenever the receiving party is down, all your messages are lost. However, if you pick a persistent transport layer, like a message queue to send messages, you won't have this problem.

If the receiving service is down, the messages will be kept, and can be processed later. If the service is not down, but there is an issue, processing can be retried, so no data gets lost.

## Unit Testing

## E2E Testing

## Linting and Styling
Prettier
ESLint/TSLint

https://github.com/standard/standard

## Security

# Delivery and Deployment

## Dockerize

## Documentation

JSDoc

## SemVer

Versioning your application / modules is critical - your consumers must know if a new version of a module is published and what needs to be done on their side to get the new version.

This is where semantic versioning comes into the picture. Given a version number MAJOR.MINOR.PATCH, increment the:
* MAJOR version when you make incompatible API changes,
* MINOR version when you add functionality (without breaking the API), and
* PATCH version when you make backwards-compatible bug fixes.

## PM2

## Logging and Tracing

Separate access logs and application logs

Opentracing

## Monitoring and Metrics
Prometheus

## Scaling

## Profiling

* `npm install -g autocannon`
* `npm install -g clinic`

`autocannon -c100 localhost:3000/seed/v1`

`clinic doctor --on-port=’autocannon -c100 localhost:$PORT/seed/v1’ -- node index.js`

`clinic flame --on-port=’autocannon -c100 localhost:$PORT/seed/v1’ -- node index.js`

### References
* [Keeping Node.js Fast: Tools, Techniques, And Tips For Making High-Performance Node.js Servers](https://www.smashingmagazine.com/2018/06/nodejs-tools-techniques-performance-servers/)