## Configuring Rules
Prometheus supports two types of rules which may be configured and then evaluated at regular intervals: recording rules and alerting rules. To include rules in Prometheus, create a file containing the necessary rule statements and have Prometheus load the file via the `rule_files` field in the [Prometheus configuration](https://github.com/prometheus/prometheus/blob/master/config/config.proto).

## Syntax-Checking Rules
To quickly check whether a rule file is syntactically correct without starting a Prometheus server, install and run Prometheus' `rule_checker` tool:

```bash
# If $GOPATH/github.com/prometheus/prometheus already exists, update it first:
go get -u github.com/prometheus/prometheus

go install github.com/prometheus/prometheus/tools/rule_checker
rule_checker -ruleFile=/path/to/example.rules
```

When the file is syntactically valid, the checker prints a textual representation of the parsed rules and then exits with an `0` return status.

If there are any syntax errors, it prints an error message and exits with a `255` return status.

## Recording Rules
Recording rules allow you to precompute frequently needed or computationally expensive expressions and save their result as a new set of timeseries. Querying the precomputed result will then often be much faster than executing the original expression every time it is needed. This is especially useful for dashboards, which need to query the same expression repeatedly every time they refresh.

To add a new recording rule, add a line of the following syntax to your rule file:

    <new timeseries name>[{<label overrides>}] = <expression to record>

Some examples:

    // Saving the per-job HTTP request count as a new set of timeseries:
    job:api_http_requests_total:sum = sum(api_http_requests_total) by (job)

    // Drop or rewrite labels in the result timeseries:
    new_timeseries{label_to_change="new_value",label_to_drop=""} = old_timeseries

Recording rules are evaluated at the interval specified by the `evaluation_interval` field in the Prometheus configuration. During each evaluation cycle, the right-hand-side expression of the rule statement is evaluated at the current instant in time and the resulting sample vector is stored as a new set of timeseries with the current timestamp and a new metric name (and perhaps an overridden set of labels).

## Alerting Rules
Alerting rules allow you to define alert conditions based on Prometheus expression language expressions and to send notifications about firing alerts to an external service. Whenever the alert expression results in one or more vector elements at a given point in time, the alert counts as active for these elements' label sets.

### Defining Alerting Rules
Alerting rules are defined in the following syntax:

    ALERT <alert name>
      IF <expression>
      [FOR <duration>]
      WITH <label set>
      SUMMARY "<summary template>"
      DESCRIPTION "<description template>"

The optional `FOR` clause causes Prometheus to wait for a certain duration between first encountering a new expression output vector element (like an instance with a high HTTP error rate) and counting an alert as firing for this element. Elements that are active, but not firing yet, are in pending state.

The `WITH` clause allows specifying a set of additional labels to be attached to the alert. Any existing conflicting labels will be overwritten.

The `SUMMARY` should be a short, human-readable summary of the alert (suitable for e.g. an email subject line), while the `DESCRIPTION` clause should provide a longer description. Both string fields allow the inclusion of template variables derived from the firing vector elements of the alert:

    // To insert a firing element's label values:
    {{$labels.<labelname>}}
    // To insert the numeric expression value of the firing element:
    {{$value}}

Examples:

    // Alert for any instance that is unreachable for >5 minutes.
    ALERT InstanceDown
      IF up == 0
      FOR 5m
      WITH {
        severity="page"
      }
      SUMMARY "Instance {{$labels.instance}} down"
      DESCRIPTION "{{$labels.instance}} of job {{$labels.job}} has been down for more than 5 minutes."

    // Alert for any instance that have a median request latency >1s.
    ALERT ApiHighRequestLatency
      IF api_http_request_latencies_ms{quantile="0.5"} > 1000
      FOR 1m
      WITH {}
      SUMMARY "High request latency on {{$labels.instance}}"
      DESCRIPTION "{{$labels.instance}} has a median request latency above 1s (current value: {{$value}})"

### Inspecting Alerts During Runtime
To manually inspect which alerts are active (pending or firing), navigate to the "Alerts" tab of your Prometheus instance. This will show you the exact label sets for which each defined alert is currently active.

### Sending Alert Notifications
Prometheus's alerting rules are good at figuring what is broken *right now*, but they are not a fully-fledged notification solution. Another layer is needed to add summarization, notification rate limiting, silencing and alert dependencies on top of the simple alert definitions. In Prometheus' ecosystem, the [Alert Manager](http://github.com/prometheus/alertmanager) takes on this role. Thus, Prometheus may be configured to periodically send information about alert states to an Alert Manager instance, which then takes care of dispatching the right notifications. The Alert Manager instance may be configured via the `-alertmanager.url` command line flag.