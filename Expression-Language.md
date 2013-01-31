# Prometheus Expression Language

## Overview

Prometheus provides a functional expression language that lets the user select and aggregate timeseries data in real-time. The result of an expression can either be shown as a graph, viewed as data in the expression browser, or consumed and further processed by external systems via the HTTP API.

## Data Types

### Timeseries

Data in Prometheus is stored as timeseries, which are uniquely identified by a metric name and a set of arbitrary label/value pairs. Each time series can have one or more data points attached to it. Data points are timestamp/value pairs.

#### Metric name
The metric name of a timeseries (e.g. "http_requests_total") specifies the general feature of a system that is measured. It may contain alpha-numeric characters, plus underscores and colons.

#### Labels
The label/value pairs attached to a timeseries add dimensions that allow later filtering and aggregation by these dimensions. Label keys are identifiers (alpha-numeric characters, plus underscores and colons), while their values may be arbitrary strings.

#### Data points
Each timeseries can have one or more data points attached to it. Data points are timestamp/value pairs, where values are always floating-point numbers (currently 32-bit precision).

## Operators

## Functions

## Gotchas