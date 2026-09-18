---
title: A single property
weight: 1
---

The smallest document that says something: one property, one condition.

```toml
#:schema ../schemas/v0/ryspec.schema.json

[[properties]]
name = "my_property"
check = "({s} -> once[3:10] {p})"
```

Whenever `s` holds, `p` must have held at some point in the last 3 to 10 steps.

`s` and `p` are never declared. Neither is a rule or a property, so both can only be inputs and are deduced as such — `[variables]` and `[monitor]` are optional, and a document this size has no need of them.

The condition is written as an infix expression, the notation that reads closest to the requirement itself. The same condition can equally be stated in prefix form, as an operator followed by its operands.

Source: [`examples/basic_property.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/basic_property.toml)
