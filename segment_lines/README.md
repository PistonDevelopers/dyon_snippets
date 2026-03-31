# Segment Lines

A tiny snippet to split a list of lines into lists of line segments.

The format of line data is `[vec4]` with the encoding `[x0, y0, x1, y1, x2, y2, ...]`.

Line segments need to be continuous in the input data.

Example:

```rust
segments := segment_lines([
    (0, 0), (1, 0),
    (1, 1), (2, 1),
    (0, 1), (1, 1),
    (1, 0), (2, 0),
])
println(segments)
```
