### Unit Test

A tiny snippet that makes it easy to write unit tests.

Example:

foo.dyon:
```rust
test_1() = \() = {
  1 + 1 == 2
}

// You can also generate a random unit test like this.
test_2() = \() = {
    (1 + grab random()) > grab random()
}
```

loader:
```rust
// The module to test.
foo := unwrap(load("foo.dyon"))

string := unwrap(load("string.dyon"))
unit_test := unwrap(load(source: "unit_test.dyon", imports: [string]))

_ := unwrap(call_ret(unit_test, "run_tests", [foo]))
```
