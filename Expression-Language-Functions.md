## time()

`time()` returns the number of seconds since January 1, 1970 UTC.

## count()

`count(v sample-vector)` returns the number of elements (timeseries) in a sample vector `v` as a scalar value.

Example which returns the number of timeseries with the metric name `http_requests` and the `job` label set to `api-server`:

```
count(http_requests{job="api-server"})
```

## delta()

`delta(v range-vector, counter bool)` calculates the difference between the first and last value of each timeseries element in a range vector `v`, returning a sample vector with the given deltas and equivalent labels. If `counter` is set to `1` (`true`), the timeseries in the range vector are treated as monotonically increasing counters. Breaks in monotonicity (such as counter resets due to target restarts) are automatically adjusted for. Setting `counter` to `0` (`false`) turns this behavior off.

Example which returns the total number of HTTP requests counted within the last 5 minutes, per timeseries in the range vector:

```
delta(http_requests{job="api-server"}[5m], 1)
```

Example which returns the difference in CPU temperature between now and 2 hours ago:

```
delta(cpu_temp_celsius{host="zeus"}[2h], 0)
```

## rate()

`rate(v range-vector)` behaves like `delta()`, with two differences:
* the returned delta is converted into a per-second rate, according to the respective interval
* the `counter` argument is implicitly set to `1` (`true`)

Example call which returns the per-second rate of HTTP requests as measured over the last 5 minutes, per timeseries in the range vector:

```
rate(http_requests{job="api-server"}[5m])
```