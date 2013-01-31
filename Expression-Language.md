# Prometheus Expression Language

## Overview

Prometheus provides a functional expression language that lets the user select and aggregate timeseries data in real-time. The result of an expression can either be shown as a graph, viewed as data in the expression browser, or consumed and further processed by external systems via the HTTP API.

## Basic Concepts

### Timeseries

Data in Prometheus is stored as timeseries, which are uniquely identified by a metric name and a set of arbitrary label/value pairs. Each timeseries can have one or more data points attached to it. Data points are timestamp/value pairs.

#### Metric name
The metric name of a timeseries (e.g. "http_requests_total") specifies the general feature of a system that is measured. It may contain alpha-numeric characters, plus underscores and colons.

#### Labels
The label/value pairs attached to a timeseries add dimensions that allow later filtering and aggregation by these dimensions (e.g. "endpoint", "response_code"). Label keys are identifiers (alpha-numeric characters, plus underscores and colons), while their values may be arbitrary strings.

#### Data points
Each timeseries can have one or more data points attached to it, which are timestamp/value pairs. Values are always encoded as floating-point numbers (currently 32-bit precision).

## Expression Language Data Types

In Prometheus' expression language, an expression or sub-expression can evaluate to one of four types:

* string
* scalar (float)
* sample vector (vector of single data points in time from multiple timeseries)
* range vector / matrix (vector of data point streams from multiple timeseries)

Depending on the use-case (e.g. when graphing vs. displaying the output of an expression), only some of these types are legal as the result from a user-specified expression. For example, an expression that returns a point vector is the only type that can be directly graphed.

## Literals

### String Literals

Strings may be specified as literals in single or double quotes.

Example:

    "this is a string"

### Float Literals

Scalar float values can be literally written as numbers of the form `[-](digits)[.(digits)]`.

    -2.43

### Sample Vector Literals

Sample vector literals allow the selection of a set of timeseries: in the simplest form, only a metric name is specified. This results in a sample vector containing elements for all timeseries that have this metric name.

This example selects all timeseries that have the "http_requests_total" metric name:

    http_requests_total

It is possible to filter these timeseries further by appending a set of labels to match in curly braces (`{}`).

This example selects only those timeseries with the "http_requests_total" metric name that also have the `job` label set to `prometheus` and their `group` label set to `canary`:

    http_requests_total{job="prometheus", group="canary"}

### Range Vector Literals

Range vector literals work like sample vector literals, except that a range duration is appended in square brackets (`[]`) at the end of the literal that specifies how far back in time values should be fetched for each resulting range vector element.

Time durations are specified as a number, followed immediately by one of the following units:

* `s` - seconds
* `m` - minutes
* `h` - hours
* `d` - days
* `w` - weeks
* `y` - years

In this example, we select all the values we have recorded within the last 5 minutes for all timeseries that have the metric name `http_requests_total` and a `job` label set to `prometheus`:

    http_requests_total{job="prometheus"}[5m]

## Operators

Prometheus supports many binary and aggregation operators. These are described in detail in the [[Expression Language Operators]] page.

## Functions

Prometheus supports several binary and aggregation operators. These are described in detail in the [[Expression Language Functions]] page.

## Gotchas

TODO:
* staleness and interpolation
* ...