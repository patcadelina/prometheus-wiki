## Arithmetic Binary Operators
The following binary arithmetic operators exist in Prometheus:

* `+` (addition)
* `-` (subtraction)
* `*` (multiplication)
* `/` (division)
* `%` (modulo)

Binary arithmetic operators are defined between scalar/scalar, vector/scalar, and vector/vector value pairs. 

**Between two scalars**, they do the obvious: they evaluate to another scalar that is the result of the operator applied to both scalar operands (i.e. "5 + 2 => 7").

**Between a point vector and a scalar**, the operator is applied to the value of every data point in the vector. E.g. if a timeseries point vector is multiplied by 2, the result is another vector in which every data point of the original vector is multiplied by 2.

**Between two point vectors**, a binary arithmetic operator only applies to vector elements that have identical sets of labels between the two vectors. Vector elements that don't find an exact label match on the other side get dropped from the result. The metric name of the result vector is carried over from the left hand side of the expression.

## Comparison / Filter Binary Operators

The following binary comparison/filter operators exist in Prometheus:

* `>` (greater-than)
* `<` (less-than)
* `>=` (greater-or-equal)
* `<=` (less-or-equal)

Comparison/filters operators are defined between scalar/scalar, vector/scalar, and vector/vector value pairs.

**Between two scalars**, these operators result in another scalar that is either `0` (`false`) or `1` (`true`), depending on the comparison result.

**Between a point vector and a scalar**, these operators are applied to the value of every data point in the vector, and vector elements between which the comparison result is `false` get dropped from the result vector.

**Between two point vectors**, these operators behave as a filter: They apply to vector elements that have identical sets of labels between the two vectors. Vector elements for which the expression evaluates to `false` or which don't find an exact label match on the other side of the expression get dropped from the result, while the others get carried over into a result vector with their original (left-hand-side) metric names and data values.

# Logical/Set Binary Operators

These logical/set binary operators are only defined between point vectors:

* `and` (intersection)
* `or` (union)

`vector1 and vector2` results in a vector consisting of the elements of `vector1` for which there are elements in `vector2` with exactly matching labelsets. Other elements are dropped. The metric name and values are carried over from the left-hand-side vector.

`vector1 or vector2` results in a vector that contains all original elements (labelsets + values) of `vector1` and additionally all elements of `vector2`, which don't have matching labelsets in `vector1`. The metric name is carried over from the left-hand-side vector in both cases.

# Aggregation Operators

Prometheus supports the following built-in aggregation operators that can be used to aggregate point vector elements into a vector of fewer elements with aggregated values:

* `sum` (calculate sum over dimensions)
* `min` (select minimum over dimensions)
* `max` (select maximum over dimensions)
* `avg` (calculate the average over dimensions)

These operators can either be used to aggregate over **all** label dimensions or preserve distinct dimensions by including a `by`-clause:

    <aggr-op>(<vector expression>) [by (<label list>)]

Example:

If the metric `http_requests_total` had timeseries that fan out by `application`, `instance`, and `group` labels, we could calculate the total number of seen HTTP requests per application and group over all instances via:

    sum(http_requests_total) by (application, group)

If we are just interested in the total of HTTP requests we've seen in **all** applications, we could simply write:

    sum(http_requests_total)