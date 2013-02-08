## Overview

This guide is a "Hello World"-style codelab for installing, configuring, and using Prometheus in a very simple example setup.

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

```
// Global default settings.
global {
  scrape_interval = "15s"     // By default, scrape targets every 15 seconds.
  evaluation_interval = "15s" // By default, evaluate rules every 15 seconds.

  // Attach these extra labels to all timeseries collected by this Prometheus instance.
  labels {
    monitor = "codelab-monitor"
  }
}

// A job definition containing exactly one endpoint to scrape: Prometheus itself.
job {
  // The job name is added as a label `job=<job-name>` to any timeseries scraped from this job.
  name = "prometheus"
  // Override the global default and scrape targets from this job every 5 seconds.
  scrape_interval = "5s"

  // Let's define a group of targets to scrape for this job. In this case, only one.
  targets {
    // These endpoints are scraped via HTTP.
    endpoints = [
      "http://localhost:9090/metrics.json"
    ]
  }
}
```

## Starting Prometheus

To start Prometheus with your newly created configuration file, change to your Prometheus build directory and run:

```bash
# By default, Prometheus stores its database in /tmp/metrics (flag "-metricsStoragePath").
# Create this directory if it doesn't exist yet:
mkdir -p /tmp/metrics

# Start Prometheus.
./prometheus.build -configFile=prometheus.conf
```

Prometheus should start up and it should show a status page about itself at `http://localhost:9090/`. Give it a couple of seconds to start collecting data about itself from its own HTTP metrics endpoint.

You can also verify that Prometheus is serving metrics about itself by navigating to its metrics exposure endpoint: `http://localhost:9090/metrics.json`

## Using the Expression Browser

Let's try looking at some data that Prometheus has collected about itself. To use Prometheus' built-in expression browser, navigate to `http://localhost:9090/`

As you can gather from `http://localhost:9090/metrics.json`, one metric that Prometheus exports about itself is called `prometheus_metric_disk_latency_microseconds`. Go ahead and enter this into the expression browser:

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

To graph expressions, navigate to `http://localhost:9090/static/graph.html`.

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

You now have example targets listening on ports 8080, 8081, and 8082.

## Configuring Prometheus to Monitor the Sample Targets

...