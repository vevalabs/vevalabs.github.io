---
title: Format Reference
weight: 2
---

A `ryspec` document is a TOML file. Comments, strings, numbers, arrays, tables and arrays of tables follow TOML rules, so any TOML parser reads the document and any TOML editor formats it. The `ryspec`-specific part is a small vocabulary of root keys and the expression forms used in conditions.

## Document root

Eight keys may appear at the document root. Only `properties` is required.

| Key | Kind | Purpose |
| --- | --- | --- |
| `properties` | array of tables | The verdicts the document requires. Required. |
| `rules` | table | Named subformulas shared across properties. |
| `variables` | table | Declared names, their value space and metadata. |
| `monitor` | table | The monitor interface and the order of its partitions. |
| `features` | table | Format flags, such as enabling infix expressions. |
| `namespace` | string | A URI identifying this document. |
| `meta` | table | Facts about the document: title, description, author, license. |
| `extras` | table | Free-form data outside the document's metadata. |

Both `meta` and `extras` accept arbitrary keys beyond the conventional ones.

## Properties

Each `[[properties]]` entry is one requirement. Properties are evaluated independently while sharing the signals and rules they reference.

```toml
[[properties]]
name = "brakes_after_warning"
title = "Brakes engage after a warning"
criticality = "critical"
message = "brake was not engaged within 10 steps of a warning"
check = ["once", "brake", { max = 10 }]
```

| Field | Purpose |
| --- | --- |
| `name` | Identifier for the property, unique within the document. Required. |
| `check` | The condition that must hold. Required. |
| `given` | An antecedent: the property is evaluated only where this holds. |
| `title` | A human-readable restatement of the requirement. |
| `criticality` | Classification used by the consuming integration. |
| `message` | Text reported when the property fails. |
| `rules` | A `[properties.rules]` table of subformulas private to this property. |

`given` and `check` together say *where the property applies* and *what must then be true*. A property with a `given` produces a verdict only where the antecedent holds and is vacuously satisfied elsewhere, which is worth preferring over folding the antecedent into an implication inside `check`: the reason for the requirement stays visible, and a tool can report how often the property was actually exercised.

## Expression forms

Conditions are written in one of two interchangeable forms.

**Prefix form** is an array whose first element is an operator and whose remaining elements are its operands. There is no precedence and no grammar to parse, which makes it the form tools should generate:

```toml
check = ["implies", ["gt", "speed", 0], ["gt", "gear", 0]]
```

Bounds on a temporal operator are given as a trailing table:

```toml
cond1 = ["once", "q", { min = 3, max = 10 }]
```

**Infix form** is a string written the way the requirement reads, and a document must opt into it:

```toml
[features]
allow_expressions = true

[[properties]]
name = "my_property"
check = "({s} -> once[3:10] {p})"
```

Bounds follow the operator in brackets, and an omitted endpoint is left open: `once[3:10]` means between 3 and 10 steps ago, `once[:10]` means within the last 10.

Operators include the boolean connectives `not`, `and`, `or`, `implies` (written `->` in infix form) and `equiv`; the past-time temporal operators `once`, `always`, `since` and `prev`; and the comparisons used against numeric variables, such as `le` and `gt`.

### Referring to names

A string that stands alone — a whole `check`, a whole `given`, or a whole operand of a prefix array — is read as a reference to a variable, rule or property:

```toml
given = "cond1"
check = ["since", "not_p", "q"]
```

Inside an infix expression there is no such position, so a reference is marked with braces:

```toml
check = "({subexpr2} and {subexpr3})"
```

## Rules

A rule is a named subformula. Declared at the document root, it is shared by every property:

```toml
[rules]
subexpr1 = "(once[:10] {q})"
subexpr2 = "(not {p})"
subexpr3 = "(always {r})"

[[properties]]
name = "property1"
given = "subexpr1"
check = "({subexpr2} and {subexpr3})"
```

Rules may reference other rules, so a document can build a vocabulary of domain conditions once and phrase each property in those terms. A rule declared under `[properties.rules]` is scoped to its own property instead — the visibility boundary is the only difference.

## Variables

The `[variables]` table declares the names a document uses. Each entry is an inline table of attributes:

```toml
[variables]
speed = { type = "number", unit = "m/s", description = "vehicle speed" }
brake = { type = "bool" }
speed_max = { initial_value = 50.0, min = 0.0 }
speed_within_limit = { type = "bool" }
```

| Attribute | Purpose |
| --- | --- |
| `type` | `bool`, `number`, `text` or `binary`. |
| `unit` | Physical unit of the value, for documentation and tooling. |
| `description` | What the variable means. |
| `format` | Encoding of a structured value, such as `json` for a `text` variable. |
| `source` | A dotted path naming where the value is read from. |
| `initial_value` | Starting value, required for a parameter. |
| `min`, `max` | Bounds on the value space. |

### Partitions

Every variable belongs to one of three partitions. **Inputs** are fed from outside and are the default. **Outputs** are computed by the monitor and published. **Parameters** are set once at start-up and then held, which is why a parameter must carry an `initial_value`.

### Sources

A `source` reads a value out of somewhere else rather than claiming an input slot of its own. The most useful case is decoding a structured frame once and projecting fields out of it:

```toml
[variables]
vehicle_state = { type = "text", format = "json", description = "telemetry frame" }
speed = { type = "number", source = "vehicle_state.vehicle.speed" }
gear = { type = "number", source = "vehicle_state.drive.gear" }
```

Here `vehicle_state` is the only input. The head of a source path must be a declared variable, and a name cannot draw its value from two places at once.

## Monitor

The `[monitor]` table describes the interface of the monitor generated from the document. Listing a name fixes its partition, and the order of each list is the actual order of that partition — which matters when a generated monitor is called positionally:

```toml
[monitor]
inputs = ["speed", "brake"]
outputs = ["speed_within_limit"]
parameters = ["speed_max"]

[monitor.runtime]
allocation_size = 65536
buffer_size = 4096
```

`[monitor.runtime]` carries execution settings for the consuming tool, such as buffer and allocation sizes.

### Deduction

`[monitor]` and `[variables]` are both optional. A name in a rule that is not a rule, a property or a declared variable can only be an input, so it is deduced. A short document can therefore consist of nothing but properties, and the interface is still well defined.

## Identity and metadata

```toml
namespace = "https://example.com/ryspec/vehicle-braking"

[meta]
title = "Vehicle braking properties"
description = "Braking response checks for the reference vehicle model."
author = "John Doe"
license = "MIT"

[extras]
owner_team = "safety"
jira_project = "VEH"
```

`namespace` is a URI that identifies the document, which matters when properties from several documents are collected into one result set. `meta` records facts about the document; `extras` holds data that is not metadata about the specification but travels with it — ownership, ticket references, anything the surrounding process needs.

## Validation

Validation happens in two layers. The JSON Schema at `schemas/v0/ryspec.schema.json` catches shape errors: unknown keys, wrong types, malformed arrays. Referencing it from a `#:schema` comment at the top of a document gives an editor inline errors and completion while you type.

The loader then enforces the semantic rules a schema cannot express — duplicate names across partitions, invalid partition assignments, a parameter without an `initial_value`, a source path whose head is not declared, an output that also declares a source, an infix expression in a document that has not set `allow_expressions`, a bound whose `min` exceeds its `max`.

The `ryspec validate` command runs both layers over a file or a directory of files.
