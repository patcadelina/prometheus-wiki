## Metrics

A single metric

* should have a (single-word) application prefix relevant to the containing Prometheus domain
 * haproxy_foo_duration_milliseconds
 * indexer_request_count
 * processor_request_count
* must have a single unit
* should have a units suffix
 * foo_duration_nanoseconds
 * bar_egress_bytes
 * baz_count
* should represent the same logical thing-being-measured
 * request duration
 * bytes of data transfer
 * instantaneous resource usage as a percentage
* should name the thing-being-measured in singular form
 * foo_request_count rather than foo_requests

## Labels

Use labels to differentiate

* class of thing-being-measured
 * foo_http_request type={create,update,delete}
 * bar_request_duration_nanoseconds stage={extract,transform,load,overall}
* result of an operation
 * processor_request_{count,duration_milliseconds} result={success,failure}

Remember that every unique (label, value) pair represents a new axis of cardinality for 
the associated metric, which can dramatically increase the amount of data stored.