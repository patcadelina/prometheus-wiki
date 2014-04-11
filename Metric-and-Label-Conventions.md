The metric and label conventions presented in this document are not required for using Prometheus, but can serve as both a style-guide and collection of best practices. Individual organizations might want to approach e.g. naming conventions differently.

## Metric Names

A metric name

* should have a (single-word) application prefix relevant to the containing Prometheus domain
 * `prometheus_notifications_total`
 * `indexer_requests_latencies_ms`
 * `processor_requests_total`
* must have a single unit (i.e. don't mix seconds with milliseconds)
* should have a units suffix
 * `api_http_request_latency_ms`
 * `node_memory_usage_bytes`
 * `api_http_requests_total` (for a count)
* should represent the same logical thing-being-measured
 * request duration
 * bytes of data transfer
 * instantaneous resource usage as a percentage

## Labels

Use labels to differentiate

* class of thing-being-measured
 * `api_http_requests_total` - differentiate request types: `type={create,update,delete}`
 * `api_request_duration_nanoseconds` - differentiate request stages: `stage={extract,transform,load,overall}`
* result of an operation
 * `processor_request_{total,duration_milliseconds}`: differentiate outcome: `result={success,failure}`

Remember that every unique (label, value) pair represents a new axis of cardinality for 
the associated metric, which can dramatically increase the amount of data stored.