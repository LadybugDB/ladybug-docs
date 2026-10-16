---
title: Aggregate Functions
description: Aggregate functions are used to compute a single result from a set of input values.
---

<div class="scroll-table">

| Function | Description | Example |
| ----------- | ----------- |  ----------- |
| `avg(arg)` | returns the average value of all tuples in arg | `avg(a.length)` |
| `count(arg)` | returns the number of tuples in arg | `count(a.ID)` |
| `min(arg)` | returns the minimum value of arg | `min(a.length)` |
| `max(arg)` | returns the maximum value of arg | `max(a.length)` |
| `sum(arg)` | returns the sum value of all tuples in arg | `sum(a.length)` |
| `collect(arg)` | returns a list of values returned by arg expression | `collect(a.age)` |
| `percentileDisc(arg, percentile)` | returns the value that corresponds to the given discrete percentile of the input values; `percentile` must be a literal between `0.0` and `1.0` | `percentileDisc(a.age, 0.5)` |

</div>