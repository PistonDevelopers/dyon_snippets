### Date

This is a tiny snippet that lets you create a date, and to compute the difference in days between two dates.

```
[year: f64, month: f64, day: f64]
```

Example:

```rust
// January 1, 2017
a := date(year: 2017, month: 1, 1)
// December 31, 2017
b := date(year: 2017, month: 12, 31)
println(sub_days(b, a))
```
