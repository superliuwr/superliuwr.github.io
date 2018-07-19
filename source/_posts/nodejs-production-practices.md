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

<!-- more -->

## Design

### Project Structure

Structure your solution by self-contained components.

Each component should contain 'layers' - a dedicated object for the web, logic and data access code. This not only draws a clean separation of concerns but also significantly eases mocking and testing the system. Though this is a very common pattern, API developers tend to mix layers by passing the web layer objects (Express req, res) to business logic and data layers - this makes your application dependant on and accessible by Express only.

In a large app that constitutes a large code base, cross-cutting-concern utilities like logger, encryption and alike, should be wrapped by your own code and exposed as private NPM packages. This allows sharing them among multiple code bases and projects.

Avoid the nasty habit of defining the entire Express app in a single huge file - separate your 'Express' definition to at least two files: the API declaration (app.js) and the networking concerns (WWW). For even better structure, locate your API declaration within components.

### Handle Different Environments and Configurations

When dealing with configuration data, many things can just annoy and slow down:

1. setting all the keys using process environment variables becomes very tedious when in need to inject 100 keys (instead of just committing those in a config file), however when dealing with files only the DevOps admins cannot alter the behavior without changing the code. A reliable config solution must combine both configuration files + overrides from the process variables

2. when specifying all keys in a flat JSON, it becomes frustrating to find and modify entries when the list grows bigger. A hierarchical JSON file that is grouped into sections can overcome this issue + few config libraries allow to store the configuration in multiple files and take care to union all at runtime. See example below

3. storing sensitive information like DB password is obviously not recommended but no quick and handy solution exists for this challenge. Some configuration libraries allow to encrypt files, others encrypt those entries during GIT commits or simply don't store real values for those entries and specify the actual value during deployment via environment variables.

4. some advanced configuration scenarios demand to inject configuration values via command line (vargs) or sync configuration info via a centralized cache like Redis so multiple servers will use the same configuration data.

Some configuration libraries can provide most of these features for free, have a look at NPM libraries like rc, nconf and config which tick many of these requirements.

### Error Handling


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

integrated with git

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
* [Node.js Best Practices](https://github.com/i0natan/nodebestpractices/blob/master/README.md#5-going-to-production-practices)