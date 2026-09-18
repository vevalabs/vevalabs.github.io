---
title: Structured input
weight: 6
---

A `text` variable decoded as JSON lets other variables read fields out of it through a dotted `source` path, without claiming an input slot of their own.

```toml
#:schema ../schemas/v0/ryspec.schema.json

[monitor]
inputs = ["vehicle_state"]
outputs = ["in_gear_while_moving"]

[variables]
vehicle_state = { type = "text", format = "json", description = "telemetry frame" }
speed = { type = "number", source = "vehicle_state.vehicle.speed" }
gear = { type = "number", source = "vehicle_state.drive.gear" }
in_gear_while_moving = { type = "bool" }

[[properties]]
name = "in_gear_while_moving"
check = ["implies", ["gt", "speed", 0], ["gt", "gear", 0]]
```

The monitor takes one input — the whole telemetry frame — while the properties are still written against `speed` and `gear` as if they were ordinary signals. This is what keeps requirements readable when a system publishes structured frames rather than flat signals.

The head of every source path, `vehicle_state` here, must itself be a declared variable, and a name cannot draw its value from two places at once.

The check is a single `implies`. The same requirement can be stated with the antecedent in `given` — `given = ["gt", "speed", 0]` with `check = ["gt", "gear", 0]` — which is usually the better phrasing, since the property then reports a verdict only on the steps where the vehicle is actually moving.

Source: [`examples/sources_json.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/sources_json.toml)
