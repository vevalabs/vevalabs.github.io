---
title: Prefix form
weight: 4
---

Prefix form represents a rule as an operator followed by its operands, with no infix syntax and no precedence rules — the form to generate when a tool writes specifications rather than a person.

```toml
#:schema ../schemas/v0/ryspec.schema.json

[[properties]]
name = "p1"
given = "cond1"
check = ["since", "not_p", "q"]

[properties.rules]
cond1 = ["once", "q", { min = 3, max = 10 }]
not_p = ["not", "p"]
```

Bounds on a temporal operator come as a trailing table, so `["once", "q", { min = 3, max = 10 }]` is the prefix spelling of `once[3:10] {q}`.

`[properties.rules]` holds subformulas private to this property. The table is identical to a document-level `[rules]`; only the visibility differs, which keeps a name like `cond1` from leaking into a document-wide vocabulary.

The reference rule is the same in both notations. `given = "cond1"` and the operands `"not_p"` and `"q"` are bare names, because a string that is a whole field or a whole operand is already a reference. Braces are only needed to interpolate a name inside an infix expression.

Source: [`examples/prefix_form.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/prefix_form.toml)
