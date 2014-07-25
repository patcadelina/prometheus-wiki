Prometheus has support for templating in both the summary/description field of alerts and also exposed as html endpoints. Templates have the ability to run queries against the databases, iterate over data, use conditionals, format data etc. This is based on the [go templating](http://golang.org/pkg/text/template/) system.

If you're a first time user looking for simple consoles [PromDash](https://github.com/prometheus/promdash) is a good place to start, as while console templates are more powerful there's also a steeper learning curve.

**N.B.** Templates are a new feature that is being actively developed, and the API is subject to change.

# Examples

## Simple alert field templates

    ALERT InstanceDown
      IF up == 0
      FOR 5m
      WITH {
        severity="page"
      }
      SUMMARY "Instance {{$labels.instance}} down"
      DESCRIPTION "{{$labels.instance}} of job {{$labels.job}} has been down for more than 5 minutes."


Alert field templates will be executed for every rule iteration for every alert that fires, so keep any queries and templates lightweight. If you've a need for more complicated templates for alerts, it's better to link to a console.

## Simple iteration

This displays a list hosts, and whether they're up
````
{{ range query "up" }}
  {{ .Labels.instance }} {{ .Labels.Value }}
{{ end }}
````

`.` is given each sample in turn.

## Display one value

````
{{ with query "some_metric{instance='someinstance'}" }}
{{ . | first | value | humanize }}
{{ end }}
````

Go and go's templating language are both strongly typed, so you have to check that samples were returned to avoid an execution error. This could happen if a scrape/rule evaluation hasn't run yet, or a host was down for example.

## Using console url parameters

````
{{ with printf "node_memory_MemTotal{job='node',instance='%s'}" .Params.instance | query }}
{{ . | first | value | humanize1024}}B
{{ end }}
````

If accessed as `console.html?instance=hostname`, `.Params.instance` will evaluate to `hostname`.

## Advanced iteration

````html
<table>
{{ range printf "node_network_receive_bytes{job='node',instance='%s',device!='lo'}" .Params.instance | query | sortByLab
el "device"}}
  <tr><th colspan=2>{{ .Labels.device }}</th></tr>
  <tr>
    <td>Received</td>
    <td>{{ with printf "rate(node_network_receive_bytes{job='node',instance='%s',device='%s'}[5m])" .Labels
.instance .Labels.device | query }}{{ . | first | value | humanize }}B/s{{end}}</td>
  </tr>
  <tr>
    <td>Transmitted</td>
    <td>{{ with printf "rate(node_network_transmit_bytes{job='node',instance='%s',device='%s'}[5m])" .Labels
.instance .Labels.device | query }}{{ . | first | value | humanize }}B/s{{end}}</td>
  </tr>{{ end }}
<table>
````

Here we iterate over the network devices, then for each of them display the network traffic. 

As the range doesn't specify a variable, `.Params.instance` isn't available inside the loop as `.` is now the loop variable.

## Basic templates

There's support for defining templates that you can reuse. This is particularly powerful when combined with console library support, allowing you to share templates across consoles.
````
{{define "myTemplate"}}do something{{end}}
{{template "MyTemplate"}}
````

## Templates with multiple arguments

Templates are limited to one argument, the `args` function can be used to wrap multiple arguments.
````
{{define "myMultiArgTemplate"}}First argument: {{.arg0}} Second argument: {{.arg1}}{{end}}
{{template "myMultiArgTemplate" (args 1 2)}}
````


# Functions

In addition to the [default functions](http://golang.org/pkg/text/template/#hdr-Functions) provided by go templating, prometheus provides functions to make writing templates easier.

If functions are used in a pipeline, the pipeline value is passed as the last argument. 

## Data Structures

The primary structure is the sample, defined as:
````go
type sample struct {
        Labels map[string]string
        Value  float64
}
````

The `__name__` label is the name of the metric.

`[]sample` means a list of samples.

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
| humanizeDuration | number        | string  | Converts a duration in seconds to a more readable format. |

Humanizing functions are intended to produce reasonable output for consumptions by humans, and are not guaranteed to return the same results between versions.

## Strings

| Name          | Arguments     | Returns |    Notes    |
| ------------- | ------------- | ------- | ----------- |
| title         | string        | string  | [strings.Title](http://golang.org/pkg/strings/#Title), capitalises first character of each word|
| match         | pattern, text | boolean | [regexp.MatchString](http://golang.org/pkg/regexp/#MatchString) Tests for a regexp match |
| reReplaceAll  | pattern, replacement, text | string | [Regexp.ReplaceAllString](http://golang.org/pkg/regexp/#Regexp.ReplaceAllString) Regexp substitution |
| graphLink  | expr | string | Returns path to graph view on /graph for the expression |
| tableLink  | expr | string | Returns path to tabular view on /graph for the expression |


## Others
| Name          | Arguments     | Returns |    Notes    |
| ------------- | ------------- | ------- | ----------- |
| args          | []interface{} | map[string]interface{} | This converts a list of objects to a map with keys arg0, arg1 etc. This is intended to allow multiple arguments to be passed to templates |
| safeHtml      | string        | string  | Marks string as HTML not requiring auto-escaping |

`interface{}` in Go is similar to a void pointer in C.

# Template Type Differences

Each of the types templates provide different information that can be used to parameterise your template, and have a few other differences

## Alert Field Templates

`.Value` and `.Labels` contain the alert value and labels. They're also exposed as the `$value` and `$labels` variables for convenience.

## Console Templates

Consoles are exposed on `/consoles/`, and sourced from the directory pointed to by the `-consoleTemplates` flag. 

Console templates are rendered with [html/template](http://golang.org/pkg/html/template/), which provides auto-escaping. To bypass the auto-escaping use the `safe*` functions.

URL parameters are available as a map in `.Params`. If you need to support multiple url parameters by the same name, `.RawParams` is a map of the list values for each parameter. The URL path is available in `.Path`, excluding the `/consoles/` prefix.

Consoles also have access to all the templates (i.e. things defined with `{{define "templateName"}}...{{end}}` found in `*.lib` files in the directory pointed to by the `-consoleLibraries` flag. As this is a shared namespace, you should take care to avoid clashing with other users. Template names beginning with `prom`, `_prom`, and `__` are reserved for use by Prometheus, as are the functions listed above.


