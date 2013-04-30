## Overview

This guide is a "Hello World"-style codelab which shows how to install, configure, and use Prometheus in a simple example setup.

## Getting Prometheus

First, fetch the latest Prometheus collector code:

```bash
git clone git@github.com:/prometheus/prometheus
```

## Building Prometheus

For building Prometheus under Linux or MacOS X, please follow the instructions in http://github.com/prometheus/prometheus/blob/master/README.md

## Configuring Prometheus to Monitor Itself

Prometheus collects telemetry from monitored targets by scraping metrics HTTP endpoints on these targets. Since Prometheus also exposes data in the same way about itself, it can actually be used to scrape and monitor its own health.

Though a Prometheus which collects only data about itself is not very useful in practice, it's a good example to get started. Save the following basic Prometheus configuration as a file named `prometheus.conf`:

```proto
// Global default settings.
global: <
  scrape_interval: "15s"     // By default, scrape targets every 15 seconds.
  evaluation_interval: "15s" // By default, evaluate rules every 15 seconds.

  // Attach these extra labels to all timeseries collected by this Prometheus instance.
  labels: <
    label: <
      name: "monitor"
      value: "codelab-monitor"
    >
  >
>

// A job definition containing exactly one endpoint to scrape: Prometheus itself.
job: <
  // The job name is added as a label `job=<job-name>` to any timeseries scraped from this job.
  name: "prometheus"
  // Override the global default and scrape targets from this job every 5 seconds.
  scrape_interval: "5s"

  // Let's define a group of targets to scrape for this job. In this case, only one.
  target_group: <
    // These endpoints are scraped via HTTP.
    target: "http://localhost:9090/metrics.json"
  >
>
```

As you might have noticed, Prometheus configuration is supplied in an ASCII form of protocol buffers. The protocol buffer schema definition has a complete documentation of all available configuration options: https://github.com/prometheus/prometheus/blob/master/config/config.proto

## Starting Prometheus

To start Prometheus with your newly created configuration file, change to your Prometheus build directory and run:

```bash
# By default, Prometheus stores its database in /tmp/metrics (flag "-metricsStoragePath").
# Create this directory if it doesn't exist yet:
mkdir -p /tmp/metrics

# Start Prometheus.
./prometheus.build -configFile=prometheus.conf
```

Prometheus should start up and it should show a status page about itself at [[http://localhost:9090/status]]. Give it a couple of seconds to start collecting data about itself from its own HTTP metrics endpoint.

You can also verify that Prometheus is serving metrics about itself by navigating to its metrics exposure endpoint: [[http://localhost:9090/metrics.json]]

## Using the Expression Browser

Let's try looking at some data that Prometheus has collected about itself. To use Prometheus' built-in expression browser, navigate to [[http://localhost:9090/]] and choose the "Console" instead of the "Graph" tab.

As you can gather from [[http://localhost:9090/metrics.json]], one metric that Prometheus exports about itself is called `prometheus_metric_disk_latency_microseconds`. Go ahead and enter this into the expression console:

```
prometheus_metric_disk_latency_microseconds
```

This should return a lot of different timeseries (along with the latest value recorded for each), all with the metric name `prometheus_metric_disk_latency_microseconds`, but with different labels. These labels designate different latency percentiles, operation types, and operation results (success, failure).

To count the number of returned timeseries, you could write:

```
count(prometheus_metric_disk_latency_microseconds)
```

If we were only interested in the 99th percentile latencies for e.g. `get_value_at_time` operations, we could use this query to retrieve that information:

```
prometheus_metric_disk_latency_microseconds{operation="get_value_at_time", percentile="0.990000"}
```

For further details about the expression language, see the [[Expression Language]] documentation.

## Using the Graphing Interface

To graph expressions, navigate to [[http://localhost:9090/]] and use the "Graph" tab.

For example, enter the following expression to graph all latency percentiles for `get_value_at_time` operations in Prometheus:

```
prometheus_metric_disk_latency_microseconds{operation="get_value_at_time"}
```

Experiment with the graph range parameters and other settings.

## Starting Up Some Sample Targets

Let's make this more interesting and start some example targets for Prometheus to scrape.

Download the Go client library for Prometheus, and run some random examples from it that export timeseries with random data:

```bash
# Fetch the client library code:
git clone git@github.com:/prometheus/client_golang

# You might also want to do this if you didn't download the above repo into your Go package path already:
go get github.com/prometheus/client_golang

# Start 3 example targets in screen sessions:
cd client_golang/examples/random
go run main.go -listeningAddress=:8080
go run main.go -listeningAddress=:8081
go run main.go -listeningAddress=:8082
```

You should now have example targets listening on [[http://localhost:8080/metrics.json]], [[http://localhost:8081/metrics.json]], and [[http://localhost:8082/metrics.json]].

## Configuring Prometheus to Monitor the Sample Targets

Now we'll configure Prometheus to scrape these new targets. Let's group these three endpoints into a job we call `random-example`. However, imagine that the first two endpoints are production targets, while the third one represents a canary instance. To model this in Prometheus, we can add several groups of endpoints to a single job, adding extra labels to each group of targets. In this example, we'll add the `group="production"` label to the first group of targets, while adding `group="canary"` to the second.

To achieve this, add the following job definition to your `prometheus.conf` and restart your Prometheus instance:

```proto
job: <
  name: "random-example"

  // The "production" targets for this job.
  target_group: <
    target: "http://localhost:8080/metrics.json"
    target: "http://localhost:8081/metrics.json"
    labels: <
      label: <
        name: "group"
        value: "production"
      >
    >
  >
  // The "canary" targets for this job.
  target_group: <
    target: "http://localhost:8082/metrics.json"
    labels: <
      label: <
        name: "group"
        value: "canary"
      >
    >
  >
>
```

Go to the expression browser and verify that Prometheus now has information about timeseries that these example endpoints expose, e.g. the `rpc_calls_total` metric.

## Configure Rules For Aggregating Scraped Data into New Timeseries

Manually entering expressions every you time you need them can get cumbersome and might also be slow to compute in some cases. Prometheus allows you to periodically record expressions into completely new timeseries via configured rules. Let's say we're interested in recording the per-second rate of `rpc_calls_total` averaged over all instances as measured over the last 5 minutes. We could write this as:

```
avg(rate(rpc_calls_total[5m]))
```

To record this expression as a new timeseries called `rpc_calls_rate`, create a file with the following recording rule and save it as `prometheus.rules`:

```
rpc_calls_rate_mean = avg(rate(rpc_calls_total[5m]))
```

To make Prometheus pick up this new rule, add a `rule_files` statement to the global configuration section in your `prometheus.conf`. The global section should now look like this:

```proto
// Global default settings.
global: <
  scrape_interval: "15s"     // By default, scrape targets every 15 seconds.
  evaluation_interval: "15s" // By default, evaluate rules every 15 seconds.

  // Attach these extra labels to all timeseries collected by this Prometheus instance.
  labels: <
    label: <
      name: "monitor"
      value: "codelab-monitor"
    >
  >
  // Load and evaluate rules in this file every 'evaluation_interval' seconds. This field may be repeated.
  rule_file "prometheus.rules"
>
```

Restart Prometheus with the new configuration and verify that a new timeseries with the metric name `rpc_calls_rate_mean` is now available by querying it through the expression browser or graphing it.