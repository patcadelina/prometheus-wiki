# Binary Operators

## Arithmetic Operators
The following binary arithmetic operators exist in Prometheus:

* `+` (addition)
* `-` (subtraction)
* `*` (multiplication)
* `/` (division)
* `%` (modulo)

Binary arithmetic operators are defined between scalar/scalar, vector/scalar, and vector/vector value pairs. 

**Between two scalars**, they do the obvious: they evaluate to another scalar that is the result of the operator applied to both scalar operands (i.e. "5 + 2 => 7").

**Between a point vector and a scalar**, the operator is applied to the value of every data point in the vector. E.g. if a timeseries point vector is multiplied by 2, the result is another vector in which every data point of the original vector is multiplied by 2.

**Between two point vectors**, a binary arithmetic operator only applies to vector elements that have identical sets of labels between the two vectors. The metric name of the result vector is carried over from the left hand side of the expression. Vector elements that don't find an exact label match on the other side get dropped from the result.

## Filter Operators

# Aggregation Operators