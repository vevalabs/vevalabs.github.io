---
title: Several properties
weight: 2
---

A document holds as many properties as the system requires.

```toml
#:schema ../schemas/v0/ryspec.schema.json

[[properties]]
name = "my_property"
check = "({s} -> once[3:10] {p})"

[[properties]]
name = "other_property"
check = "((once[:10] {q}) -> ((not {p}) since {q}))"
```

Each property is evaluated independently while sharing the signals referenced by its formula. `p` here is the same input in both, so the two requirements are stated over one vocabulary rather than two.

That sharing is the reason to group requirements in a single document: they describe the same trace. Requirements over different signals, or with different owners and release cycles, belong in separate documents.

Keep each property narrow enough that a failing verdict points at one requirement. That is what makes the property name useful as a diagnostic.

Source: [`examples/multiple_properties.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/multiple_properties.toml)
