---
title: Getting Started
layout: docs
weight: 1
---

This page takes a single requirement from a sentence to a validated specification. It assumes nothing beyond a text editor and Python 3.12 or later, and it ends with a document that a runtime verification tool can consume.

## Install the validator

The `ryspec` reference implementation bundles the JSON Schema and the validator that enforces the rules a schema cannot express. Install it from the repository:

```sh
pip install git+https://github.com/vevalabs/ryspec
```

That provides a `ryspec` command. Nothing else is needed to write and check a specification.

## Write the first property

Start with a requirement stated in words:

> The vehicle never travels faster than 27.8 m/s.

Create a directory for your specifications and put one file in it:

```sh
mkdir specs
```

```toml {filename="specs/braking.toml"}
#:schema https://raw.githubusercontent.com/vevalabs/ryspec/main/schemas/v0/ryspec.schema.json

[[properties]]
name = "speed_within_limit"
check = ["le", "speed", 27.8]
```

This is a complete document. `properties` is the only root key a document must have, and a property needs only a `name` and a `check`.

The condition is written in **prefix form**: an array whose first element is an operator and whose remaining elements are its operands, here "`speed` is less than or equal to `27.8`". There is no precedence to remember and no grammar to parse, which is why prefix form is what tools generate and what the runtime executes today.

Nothing declares `speed`. A name in a condition that is not a rule, a property or a declared variable can only be an input, so it is deduced as one — a short document needs neither a `[variables]` nor a `[monitor]` table. The bare number `27.8` becomes an anonymous parameter, given a slot as if it had been declared.

The `#:schema` comment on the first line is a convention TOML-aware editors read: point it at the bundled schema and the editor gives you completion and inline errors while you type. Every tool that is not an editor reads it as an ordinary comment.

## Validate it

```sh
ryspec validate specs
```

```text
ok: 1 file(s) validated against .../ryspec/schemas/v0/ryspec.schema.json
```

`ryspec validate` takes a **directory**, not a file, and checks every `*.toml` under it recursively. It runs two layers: the JSON Schema catches shape errors — unknown keys, wrong types, malformed arrays — and the loader then catches the semantic errors a schema cannot see, such as a name listed in two partitions or a parameter with no starting value. Whichever fails first is reported.

A failing run names the file and the JSON Pointer to the offending value, and exits non-zero:

```text
FAIL specs/braking.toml: /monitor/parameters/0: parameter 'speed_max' has no initial_value

1 of 1 file(s) failed
```

Run it in CI over the directory holding your specifications and a malformed document stops the build rather than reaching a monitor.

## Say when the property applies

Most requirements are conditional. The second one reads:

> Whenever a collision warning is active, the driver brakes within 10 steps.

Conditions like this have two parts, and `ryspec` keeps them apart. `given` is the antecedent — where the property applies — and `check` is what must then be true:

```toml {filename="specs/braking.toml"}
[[properties]]
name = "brake_follows_warning"
title = "The driver brakes within 10 steps of a collision warning"
criticality = "critical"
message = "collision warning was not followed by braking"
given = "warning"
check = ["once", "brake", { max = 10 }]
```

The alternative is to fold the antecedent into an implication inside `check`. Prefer `given`: the reason for the requirement stays visible, and a tool can report how often the property was actually exercised rather than only how often it held. A property with a `given` is vacuously satisfied wherever the antecedent does not hold.

`given = "warning"` is a bare string, and a string that stands alone — a whole `given`, a whole `check`, a whole operand — is read as a reference to a variable, rule or property. The trailing `{ max = 10 }` bounds the temporal operator: `brake` must have held at some point within the last 10 steps.

The optional fields carry the requirement's intent into the verdict. `title` restates it for a human, `message` is reported on failure, and `criticality` — one of `info`, `warning`, `error` or `critical` — classifies the verdict without changing how it is evaluated.

## Name the recurring conditions

Once several properties talk about the same things, lift those conditions into `[rules]`. A rule is a named subformula shared by every property in the document:

```toml {filename="specs/braking.toml"}
[rules]
moving = ["gt", "speed", 0]
in_gear = ["gt", "gear", 0]

[[properties]]
name = "in_gear_while_moving"
title = "The vehicle is never in motion out of gear"
given = "moving"
check = "in_gear"
```

Rules may reference other rules, so a document builds a vocabulary of domain conditions once and phrases each requirement in those terms. A rule declared under `[properties.rules]` instead is private to its own property — the visibility boundary is the only difference between the two.

## Declare the interface

Deduction is enough while a specification is still moving. As it settles, two optional tables are worth adding.

`[variables]` says what each name means; `[monitor]` says which partition it belongs to and in what order:

```toml {filename="specs/braking.toml"}
[monitor]
inputs = ["speed", "gear", "brake", "warning"]
outputs = ["speed_within_limit"]
parameters = ["speed_max"]

[variables]
speed = { type = "number", unit = "m/s", description = "vehicle speed" }
gear = { type = "number" }
brake = { type = "bool" }
warning = { type = "bool" }
speed_max = { type = "number", unit = "m/s", initial_value = 27.8, min = 0.0 }
speed_within_limit = { type = "bool" }
```

Declaring buys three things over deduction: types and units that document intent, a fixed order for each partition — which matters when a generated monitor is called positionally — and starting values for parameters. A parameter is set once at start-up and then held, so it cannot be deduced and must carry an `initial_value`; that is the error shown in the failing run above.

With `speed_max` declared, the first property is phrased against the name rather than the literal, and its verdict is published by name:

```toml
check = ["le", "speed", "speed_max"]
```

## Write conditions the way they read

A condition may also be written as an **infix** string, the notation closest to the requirement itself. A document opts in:

```toml
[features]
allow_expressions = true

[[properties]]
name = "brake_follows_warning"
check = "({warning} -> once[:10] {brake})"
```

Inside an expression there is no standalone position, so a name is marked with braces. Bounds follow the operator in brackets, and an omitted endpoint is left open: `once[3:10]` means between 3 and 10 steps ago, `once[:10]` within the last 10.

Two limits are worth knowing before you commit to it. Infix is a subset of prefix form — comparisons such as `["gt", "speed", "speed_max"]`, `prev` and `equiv` have no infix spelling — and the runtime does not execute expression strings yet: a loader accepts the syntax and reports `not yet supported`. Prefix form is what runs today.

## Where to go next

The [format reference](/docs/ryspec/format/) covers every root key, field and expression form in one place. The [examples](/docs/ryspec/examples/) walk through the reference documents from the smallest useful specification to a fully declared monitor; each is checked by the project's test suite, so any of them is a reliable file to start from.

{{< cards >}}
{{< card title="Format Reference" subtitle="Every key, field and expression form" link="/docs/ryspec/format/" >}}
{{< card title="Examples" subtitle="Annotated specifications covering each feature of the format" link="/docs/ryspec/examples/" >}}
{{< card title="Ryspec" subtitle="Schema, validator and source on GitHub" link="https://github.com/vevalabs/ryspec" icon="github" >}}
{{< /cards >}}
