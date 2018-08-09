---
title: Metrics and Performance Monitoring with Prometheus
date: 2018-06-29 23:09:56
categories:
- Microservices
tags:
- Microservices
- Metrics
- DevOps
- Prometheus
---
# Monitoring in general

## What is Monitoring?

> The term "service monitoring", means tasks of collecting, processing, aggregating, and displaying real-time quantitative data about a system.

<!-- more -->

To analyze the data, first, you need to extract metrics from your system - like the Memory usage of a particular application instance. We call this extraction **instrumentation**.

## What should be monitored?

There are different layers where an APM tool should collect data from. The more of them covered, the more insights you'll get about your system's behavior.

* Service level
* Host level
* Instance (or process) level

The list you can find below collects the most crucial problems you'll run into while you maintain a Node.js application in production.

* Service Downtimes
* Error Rate: Because errors are user facing and immediately affect your customers.
* Response time: Because the latency directly affects your customers and business.
* Throughput: The traffic helps you to understand the context of increased error rates and the latency too.
* Saturation: It tells how "full" your service is. If the CPU usage is 90%, can your system handle more traffic?
* Memory Usage: It can be used to recognize a leak.

# Prometheus

Prometheus is an open-source solution for monitoring and alerting. It provides powerful data compressions and fast data querying for time series data.

> The core concept of @PrometheusIO is that it stores all data in a time series format.

> Time series is a stream of immutable timestamped values that belong to the same metric and the same labels. The labels cause the metrics to be multi-dimensional.

## Data collection and metrics types

Prometheus uses the HTTP **pull** model, which means that every application needs to expose a **GET /metrics** endpoint that can be periodically fetched by the Prometheus instance.

Prometheus has four metrics types:

* Counter: cumulative metric that represents a single numerical value that only ever goes up
* Gauge: represents a single numerical value that can arbitrarily go up and down
* Histogram: samples observations and counts them in configurable buckets
* Summary: similar to a histogram, samples observations, it calculates configurable quantiles over a sliding time window

### Pushgateway

Prometheus offers an alternative, called the Pushgateway to monitor components that cannot be scrapped because they live behind a firewall or are short-lived jobs.

Before a job gets terminated, it can push metrics to this gateway, and Prometheus can scrape the metrics from this gateway later on.

## Monitoring an application

When we want to monitor our application with Prometheus, we need to solve the following challenges:

* Instrumentation: Safely instrumenting our code with minimal performance overhead
* Metrics exposition: Exposing our metrics for Prometheus with an HTTP endpoint
* Hosting Prometheus: Having a well configured Prometheus running
* Extracting value: Writing queries that are statistically correct
* Visualizing: Building dashboards and visualizing our queries
* Alerting: Setting up efficient alerts
* Paging: Get notified about alerts with applying escalation policies for paging

## Node.js Metrics Exporter

To collect metrics from our Node.js application and expose it to Prometheus we can use the [prom-client](https://github.com/siimon/prom-client) npm library.

```javascript
// Init
const Prometheus = require('prom-client')
const httpRequestDurationMicroseconds = new Prometheus.Histogram({
  name: 'http_request_duration_ms',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['route'],
  // buckets for response time from 0.1ms to 500ms
  buckets: [0.10, 5, 15, 50, 100, 200, 300, 400, 500]
})
```

```javascript
// After each response
httpRequestDurationMicroseconds
  .labels(req.route.path)
  .observe(responseTimeInMs)
```

```javascript
// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.set('Content-Type', Prometheus.register.contentType)
  res.end(Prometheus.register.metrics())
})
```

## Queries

Prometheus provides a functional expression language that lets the user select and aggregate time series data in real time.

The Prometheus dashboard has a built-in query and visualization tool.

## Alerting

Prometheus comes with a built-in alerting feature where you can use your queries to define your expectations, however, Prometheus alerting doesn't come with a notification system. To set up one, you need to use the **Alert manager** or an other external process.

## Kubernetes integration

Prometheus offers a built-in Kubernetes integration. It's capable of discovering Kubernetes resources like Nodes, Services, and Pods while **scraping** metrics from them.

It's an extremely powerful feature in a containerized system, where instances born and die all the time. With a use-case like this, HTTP endpoint based scraping would be hard to achieve through manual configuration.

You can also provision Prometheus easily with Kubernetes and [Helm](https://github.com/helm/charts/tree/master/stable/prometheus).

## Grafana

As you can see, the built-in visualization method of Prometheus is great to inspect our queries output, but it's not configurable enough to use it for dashboards.

As Prometheus has an API to run queries and get data, you can use many external solutions to build dashboards. One of my favorite is **Grafana**.

Grafana is an open-source, pluggable visualization platform. It can process metrics from many types of systems, and it has built-in Prometheus data source support.

# References

* [Node.js Performance Monitoring with Prometheus](https://blog.risingstack.com/node-js-performance-monitoring-with-prometheus/)
* [Example Prometheus Monitoring](https://github.com/RisingStack/example-prometheus-nodejs)
