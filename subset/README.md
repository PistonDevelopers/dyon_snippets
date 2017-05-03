### Subset

Detects whether an object or a sorted array is a subset of another.

Example:

```rust
a := {x: 1, y: 2, z: 3}
b := {x: 2, y: 3}
obj_subset_of(b, a)
```

Example:

```rust
a := [1, 2, 3]
b := [1, 2]
subset_of(b, a, \(x, y) = x < y)
```
