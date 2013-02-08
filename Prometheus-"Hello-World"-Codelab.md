## Overview

This guide is a "Hello World"-style codelab for installing, configuring, and using Prometheus for a very simple example setup.

## Getting Prometheus

First, fetch the latest Prometheus collector code:

```bash
git clone git@github.com/prometheus/prometheus
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
# By default, Prometheus stores its database in /tmp/metrics.
# Create this directory if it doesn't exist yet:
mkdir -p /tmp/metrics

# Start Prometheus.
./prometheus.build -configFile=prometheus.conf
```

Prometheus should start up and it should show a status page about itself at `http://localhost:9090/`. Give it a couple of seconds to start collecting data about itself from its own HTTP metrics endpoint.

## Using the Expression Browser

Let's try looking at some data that Prometheus has collected about itself. To use Prometheus' built-in expression browser, navigate to `http://localhost:9090/static/`.

...

## Using the Graphing Interface

## Starting Up Some Sample Targets

## Configuring Prometheus to Monitor the Sample Targets