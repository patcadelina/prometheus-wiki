## time()

`time()` returns the number of seconds since January 1, 1970 UTC.

## count()

`count(v sample-vector)` returns the number of elements (timeseries) in a sample vector `v` as a scalar value.

## delta()

`delta(v range-vector, counter bool)` calculates the difference between the first and last value of each timeseries element in a range vector `v`, returning a sample vector with the given deltas and equivalent labels. If `counter` is set to `1` (`true`), the timeseries in the range vector are treated as monotonically increasing counters. Breaks in monotonicity (such as counter resets due to target restarts) are automatically adjusted for. Setting `counter` to `0` (`false`) turns this behavior off.

## rate()

`rate(v range-vector)` behaves like `delta()`, with two differences:
* the returned delta is converted into a per-second rate, according to the respective interval
* the `counter` argument is implicitly set to `1` (`true`)