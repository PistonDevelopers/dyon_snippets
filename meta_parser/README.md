# Meta-Parser

Generates standalone parsers (no dependencies) in Dyon using a [Piston-Meta](https://github.com/pistondevelopers/meta) document and a DSL document.

### Self-Convert Rules

The DSL describes itself:

[Self-Convert](./src/self-convert.txt)

During generating the code for a new parser,
the Dyon data representing these self-convert rules are used to bootstrap the parser generator at runtime.
With other words, it generates itself first, to reduce the amount of overall code.

### Introduction

The DSL (Domain Specific Language) describes how to convert meta-data into Dyon data.

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
