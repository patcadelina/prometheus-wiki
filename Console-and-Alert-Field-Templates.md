Prometheus has support for templating in both the summary/description field of alerts and also exposed as html endpoints. Templates have the ability to run queries against the databases, iterate over data, use conditionals, format data etc. This is based on the [go templating](http://golang.org/pkg/text/template/) system.

If you're a first time user looking for simple consoles [PromDash](https://github.com/prometheus/promdash) is a good place to start, as while console templates are more powerful there's also a steeper learning curve.

**N.B.** Templates are a new feature that is being actively developed, and the API is subject to change.

# Functions

In addition to the [default functions](http://golang.org/pkg/text/template/#hdr-Functions) provided by go templating, prometheus provides functions to make writing templates easier.

If functions are used in a pipeline, the pipeline value is passed as the last argument. `[]sample` means a list of samples.

## Queries

| Name          | Arguments     | Returns  | Notes    |
| ------------- | ------------- | -------- | -------- |
| query         | query string  | []sample | Queries the databases, doesn't support returning matrixes.  |
| first         | []sample      | sample   | Equivalent to `index a 0`  |
| label         | label, sample | string   | Equivalent to `index sample.Labels label`  |
| value         | sample        | number   | Equivalent to `sample.Value`  |
| sortByLabel   | label, []samples | []sample | Sorts the samples by the given label. Is stable.  |

`first`, `label` and `value` are intended to make query results easily usable in pipelines.

## Numbers

| Name          | Arguments     | Returns |  Notes    |
| ------------- | --------------| --------| --------- |
| humanize      | number        | string  | Converts a number to a more readable format, using Metric Prefixes 
| humanize1024  | number        | string  | Like `humanize`, but uses 1024 as the base rather than 1000 |

Humanizing functions are intended to produce reasonable output for consumptions by humans, and are not guaranteed to return the same results between versions.

## Strings

| Name          | Arguments     | Returns |    Notes    |
| ------------- | ------------- | ------- | ----------- |
| title         | string        | string  | [strings.Title](http://golang.org/pkg/strings/#Title), capitalises first character of each word|
| match         | pattern, text | boolean | [regexp.MatchString](http://golang.org/pkg/regexp/#MatchString) Tests for a regexp match |
| reReplaceAll  | pattern, replacement, text | string | [Regexp.ReplaceAllString](http://golang.org/pkg/regexp/#Regexp.ReplaceAllString) Regexp substitution |


# Template Context

Each of the places templates provide different information that can be used to parameterise your template.

## Alert Field Templates

`.Value` and `.Labels` contain the alert value and labels.

They're also exposed as the `$value` and `$labels` variables for convenience, for example:

    ALERT InstanceDown
      IF up == 0
      FOR 5m
      WITH {
        severity="page"
      }
      SUMMARY "Instance {{$labels.instance}} down"
      DESCRIPTION "{{$labels.instance}} of job {{$labels.job}} has been down for more than 5 minutes."

Keep in mind that alert field templates will be executed for every rule iteration for every alert that fires, so keep any queries and templates lightweight. If you've a need for more complicated templates for alerts, it's better to link to a console.

## Console Templates

Consoles are exposed on /consoles/, and sourced from the directory pointed to by the `-consoleTemplates` flag.

URL parameters are available as a map in `.Params`. If you need to support multiple url parameters by the same name, `.RawParams` is a map of the list values for each parameter.