# Meta-Parser

Generates standalone parsers (no dependencies) in Dyon using a [Piston-Meta](https://github.com/pistondevelopers/meta) document and a DSL (Domain Specific Language) document.

Piston-Meta is built into Dyon's standard library.

This script ([src/meta_parser.dyon](./src/meta_parser.dyon)) can be used to generate a parser in Dyon for any language using Piston-Meta.

### Example

Assume that you want to parse the following data:

```text
0 1 2
3 4 5
6 7 8
9 10
```

Each line consists of 2 or 3 numbers, separated by space.

In Piston-Meta, we can describe this syntax as following:

"syntax.txt"
```text
1 pos = [.w? .$_:"x" .w! .$_:"y" ?[.w! .$_:"z"] .w?]
0 doc = .l(pos:"pos")
```

Now, we use the DSL to specify converter rules from meta-data to Dyon data:

"convert.txt"
```text
meta {
  pos := [x: f64, y: f64, z: opt[f64]]
    => (x, y, if z == none() { 0 } else { unwrap(z) });
  doc := repeat pos:"pos";
  -----------------------
  doc
}
```

Next step is to write a Dyon script that generates a parser.

First, using a loader script to add "meta_parser.dyon" in the context:

"parser_loader.dyon"
```dyon
fn main() {
    meta := unwrap(load("meta_parser.dyon"))
    main := unwrap(load(source: "parser_main.dyon", imports: [meta]))
    call(main, "main", [])
}
```

Next, generate the parser using "syntax.txt" (Piston-Meta format) and "convert.txt" (DSL):

"parser_main.dyon"
```dyon
fn main() {
    res := gen_parser(
        meta: "syntax.txt",
        from: "convert.txt",
        to: "parser.dyon"
    )
    if is_err(res) {
        eprintln("ERROR:")
        eprintln(unwrap_err(res))
        return
    }
}
```

To run, type the following in the Terminal:

```text
dyonrun parser_loader.dyon
```

**Notice:** If you do not have `dyonrun` installed, you can install it using:

```
cargo install --example dyonrun dyon
```

Now, write a loader script to put the generated parser in the context:

"loader.dyon"
```dyon
fn main() {
    m := unwrap(load("output.dyon"))
    main := unwrap(load(source: "main.dyon", imports: [m]))
    call(main, "main", [])
}
```

Finally, the main script:

```dyon
fn main() {
    println(convert(file: "data.txt"))
}
```

To run the main script:

```text
dyonrun loader.dyon
```

This should print:

```text
ok([(0, 1, 2), (3, 4, 5), (6, 7, 8), (9, 10)])
```

### Rules

The DSL is a text document with a "meta" root node:

```text
meta {
   <rules>
   ----------
   <start>
}
```

The start specifies which rule to use for the whole document.

Remember that Piston-Meta outputs an array meta-data:

```rust
pub enum MetaData {
    StartNode(Arc<String>),
    EndNode(Arc<String>),
    Bool(Arc<String>, bool),
    F64(Arc<String>, f64),
    String(Arc<String>, Arc<String>),
}
```

The rules specify how to match against meta-data and output Dyon data.

**IMPORTANT!** Remember to add a semicolon `;` at the end of each rule.
If you get a strange error, then it is probably just a missing semicolon.

#### Subrule

A subrule is referenced by its name:

```text
<rule name>
```

If meta-data generates a node name, you can specify it as a string after the name of the subrule:

```text
<rule name>:"<meta data node name>"
```

#### Set Rule

```text
<name> := [<patterns>] => <code>;
```

Here a pattern can be:

- `<name>: bool` or `<name>: opt[bool]` (optional)
- `<name>: f64` or `<name>: opt[f64]` (optional)
- `<name>: str` or `<name>: opt[str]` (optional)
- `<name> <- <subrule>` (see [Subrule](#subrule))

The code to the right of `=>` is Dyon code.
This code can depend on the variables bound by names.

#### Select Rule

```text
<name> := select {<subrules>};
```

#### Repeat Rule

```text
<name> := repeat <subrule>;
```

### Self-Convert Rules

The DSL describes itself:

[Self-Convert](./src/self-convert.txt)

During generating the code for a new parser,
the Dyon data representing these self-convert rules are used to bootstrap the parser generator at runtime.
With other words, it generates itself first, to reduce the amount of overall code.

### Introduction

The DSL describes how to convert meta-data into Dyon data.

Remember that Piston-Meta breaks up parsing into two steps:

```
f : text -> data
```

Is broken into:

```
f <=> f2 . f1
f1 : text -> meta data
f2 : meta data -> data
```

Piston-Meta handles `f1`, but `f2` is usually written in code:

```
f1 <=> piston_meta(syntax)
piston_meta : syntax -> (text -> meta data)
```

Generating `f2` from a DSL that outputs a converter in the form of Dyon code:

```
f2 <=> dsl(convert)
dsl : convert -> (meta data -> data)
```

Together, they form a complete meta-parser:

```
meta_parser <=> (piston_meta x dsl)
meta_parser : syntax x convert -> (text -> meta data, meta data -> data)
```

### Overview

"src/meta_parser.dyon":

```dyon
// Generate parser.
fn gen_parser__meta_from_to(syntax_file: str, convert_file: str, output_file: str) -> res { ... }

// The syntax of the DSL used to convert meta-data into Dyon data.
fn syntax() -> str { ... }

// Dyon data representing the rules for the parser generator.
//
// These rules are what you get when bootstrapping the parser generator,
// by using the parser generator to parse its own rules from "src/self-convert.txt".
fn self_meta() -> {} { ... }

// Generates parser code.
fn to_code__syntax_meta(syntax: str, meta: {}) -> res[str]
```
