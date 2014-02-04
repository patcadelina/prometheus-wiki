A Prometheus expression language query passes through the following stages and layers:

- Web UI
- Web API
- Query Parsing
- Query Analysis
- View Building
- Query Evaluation
- Result Serialization

## Web UI
Interactive queries usually originate from the Prometheus or PromDash web UIs. These UIs allow the user to specify an expression to query a Prometheus server for, together with some relevant parameters: the time range to be queried, as well as the query resolution.

Prometheus web UI code:
https://github.com/prometheus/prometheus/blob/master/web/static
https://github.com/prometheus/prometheus/blob/master/web/static/js/graph.js
https://github.com/prometheus/prometheus/blob/master/web/templates/graph.html

## Web API
Prometheus' web API layer takes incoming query requests and starts processing them. Itdoes some sanity and bounds checking on the query parameters and asks the AST (abstract syntax tree) layer to parse the provided expression. It then asks that layer to execute the expression against the local timeseries storage.

Range query code:
https://github.com/prometheus/prometheus/blob/master/web/api/query.go#L74

## Query Parsing
The expression parsing is realized via Yacc- and Lex-style parsers, defined here:

https://github.com/prometheus/prometheus/blob/master/rules/lexer.l
https://github.com/prometheus/prometheus/blob/master/rules/parser.y

These get automatically translated into Go code (with appended `.go` ending). The resulting parser takes a query language expression as an input, constructs an abstract syntax tree (AST) and returns a reference to the root node of the AST.

How this AST is evaluated to produce a final result is described in the Query Evaluation section.

## Query Analysis
The query analyzer recursively traverses the AST to look for any nodes which will need data from the datastore. These are usually leaf nodes containing direct timeseries references like `http_requests{code="200"}` or the corresponding ranges (`http_requests{code="200"}[5m]`). The analyzer takes the label pairs from these references and gets all matching timeseries fingerprints from the storage. These fingerprints, along with their time ranges, are collected and used to construct a request to the storage, asking it to build a view for the upcoming query evaluation. A view is a preloaded set of all data that the query evaluator will need to compute the final result of the user expression. The query analyser now waits for the storage to return this view, and then wraps it into a view adapter, which adds some wrappers which make it more convenient for the AST evaluation later to access the view later.

Query analysis and view adapter code:
https://github.com/prometheus/prometheus/blob/master/rules/ast/query_analyzer.go
https://github.com/prometheus/prometheus/blob/master/rules/ast/persistence_adapter.go

## View Building
TODO

## Query Evaluation
Each node in the AST has an `Eval()` method to evaluate its result value: usually, nodes will evaluate themselves by evaluating their child nodes recursively, up until the leaf nodes, which are direct references to groups of timeseries (`http_requests{code="200"}`), function calls (`time()`), or scalar literals (like `23`). In the case of timeseries references like `http_requests{code="200"}` the view adapter built in the previous step is used to retrieve the actual data from memory. Inner AST nodes can then aggregate or otherwise modify the results of the leaf nodes, until the final query result is returned by the `Eval()` method of the AST's root node. This result is either a scalar value (like `23`), a vector of timeseries instants (for each timeseries, a single value at a specific time), or a vector of timeseries ranges (for each timeseries, a range of values over time).

Query evaluation code:
https://github.com/prometheus/prometheus/blob/master/rules/ast/ast.go
https://github.com/prometheus/prometheus/blob/master/rules/ast/functions.go

## Result Serialization
Back in the web API, the result is marshalled into JSON and returned to the web UI or other client via HTTP.