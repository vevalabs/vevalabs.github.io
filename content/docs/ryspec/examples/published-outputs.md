---
title: Published outputs
weight: 7
---

An output is a name the monitor publishes rather than keeps internal.

```toml
#:schema ../schemas/v0/ryspec.schema.json

[monitor]
inputs = ["q", "r"]
outputs = ["guard"]

[variables]
q = { type = "bool" }
r = { type = "bool" }
guard = { type = "bool" }

[rules]
once_q = ["once", "q"]
r_and_not_q = ["and", "r", ["not", "q"]]
guard = ["and", "r_and_not_q", "once_q"]

[[properties]]
name = "guard_holds"
check = "guard"
```

`guard` appears three times, and each occurrence does a different job: `[variables]` gives it a type, `monitor.outputs` places it in the output partition, and `[rules]` defines how it is computed. The property then checks it.

Because `guard` is an output, its value is published at every step rather than only contributing to a verdict — useful when a downstream component needs the intermediate condition and not just whether the property held.

The example also shows rules composing: `guard` is built from `r_and_not_q` and `once_q`, which are themselves rules.

Source: [`examples/published_outputs.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/published_outputs.toml)
