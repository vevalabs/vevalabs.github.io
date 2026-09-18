---
title: Shared rules
weight: 3
---

Once the same subformula appears in more than one property, lift it into `[rules]`.

```toml
#:schema ../schemas/v0/ryspec.schema.json

[rules]
subexpr1 = "(once[:10] {q})"
subexpr2 = "(not {p})"
subexpr3 = "(always {r})"

[[properties]]
name = "property1"
given = "subexpr1"
check = "({subexpr2} and {subexpr3})"

[[properties]]
name = "property2"
given = "({subexpr1} and {subexpr2})"
check = "subexpr3"
```

File-level rules are named subformulas shared across properties, useful for repeated temporal patterns and for common domain conditions that deserve a name of their own. Rules may reference other rules, so a document can build a vocabulary once and phrase each property in those terms.

The two properties also show both ways of writing a reference. `given = "subexpr1"` and `check = "subexpr3"` are bare names, because the whole field is the reference. Inside an infix expression there is no such position, so `{subexpr1}` and `{subexpr2}` take braces.

Note that `given` accepts either — a bare rule name in the first property, a full expression in the second.

Source: [`examples/shared_rules.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/shared_rules.toml)
