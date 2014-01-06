## time()

`time()` returns the number of seconds since January 1, 1970 UTC.

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

## scalar()

Given a single-element input vector, `scalar(v instant-vector)` returns the sample value of that single element. If the input vector doesn't have exactly one element, `scalar` will return `NaN`.

## sort()

`sort(v instant-vector)` returns vector elements sorted by their sample values, in ascending order.

## sort_desc()

Same as `sort`, but sorts in descending order.