---
title: Variables and a monitor
weight: 5
---

As a specification stabilises, it is worth saying what the referenced names are and what the monitor's interface looks like.

```toml
#:schema ../schemas/v0/ryspec.schema.json

[monitor]
inputs = ["speed", "brake"]
outputs = ["speed_within_limit"]
parameters = ["speed_max"]

[monitor.runtime]
allocation_size = 65536
buffer_size = 4096

[variables]
speed = { type = "number", unit = "m/s", description = "vehicle speed" }
brake = { type = "bool" }
velocity = { source = "speed", type = "number", unit = "m/s" }
speed_max = { initial_value = 50.0, min = 0.0 }
speed_within_limit = { type = "bool" }

[[properties]]
name = "speed_within_limit"
check = ["le", "speed", "speed_max"]
```

`[monitor]` fixes the order of each partition, which matters when a generated monitor is called positionally. `[monitor.runtime]` carries execution settings for the consuming tool, here buffer and allocation sizes.

`[variables]` describes each name — its type, unit, and for a parameter its starting value. A parameter is set once at start-up and then held, which is why `speed_max` must carry an `initial_value`.

`velocity` is an alias: it takes its value from `speed` through `source` rather than occupying an input slot, so the same signal can be referred to under a second name without being fed twice.

Both tables remain optional. Declaring them buys three things over deduction: types and units that document intent, a fixed order for each partition, and starting values for parameters.

Source: [`examples/variables_and_monitor.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/variables_and_monitor.toml)
