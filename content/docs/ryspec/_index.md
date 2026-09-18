---
title: Reelay Specification Format
linkTitle: Specification Format
layout: docs
weight: 2
---

VevaLabs develops the Reelay Specification (`ryspec`) format, a declarative specification format for runtime verification and neuro-symbolic reasoning. Based on the widely used TOML format, the `ryspec` format provides a simple, human-readable representation of multiple temporal logic specifications that can be consumed by runtime verification tools and reasoning agents.

A `ryspec` document brings the requirements of a runtime system together in one place. Each requirement is expressed as a named property with a temporal logic condition. A document may also declare the variables referenced by those properties, the interface of the monitor that evaluates them, and metadata describing the specification.

The following example defines three properties for a braking system. It declares a telemetry input, extracts variables from the input, defines shared subformulas, and exposes a Boolean output:

```toml
[meta]
title = "Vehicle braking properties"
description = "Braking response requirements for the reference vehicle model."

# The monitor interface is declared as follows.
# The monitor observes a vehicle state object and evaluates three properties.
[monitor]
inputs = ["vehicle_state"]
outputs = ["brake_follows_warning", "in_gear_while_moving", "speed_within_limit"]
parameters = ["speed_max"]

# Variable declarations used in the properties
[variables]
vehicle_state = { type = "text", format = "json", description = "telemetry frame" }
speed = { type = "number", unit = "m/s", source = "vehicle_state.vehicle.speed" }
gear = { type = "number", source = "vehicle_state.drive.gear" }
brake = { type = "bool", source = "vehicle_state.driver.brake" }
warning = { type = "bool", source = "vehicle_state.adas.collision_warning" }
# A parameter is set once at start-up and held, so it needs a starting value.
speed_max = { type = "number", unit = "m/s", initial_value = 27.8, min = 0.0 }
# An output is published by the monitor rather than kept internal.
speeding = { type = "bool" }

# Named subformulas shared by every property below, in prefix form: an
# operator followed by its operands, with no precedence rules to apply.
[rules]
moving = ["gt", "speed", 0]
in_gear = ["gt", "gear", 0]
speeding = ["gt", "speed", "speed_max"]

# `given` is the antecedent: evaluated only while a warning is active, and
# then requiring that the brake was engaged within the last 10 steps.
[[properties]]
name = "brake_follows_warning"
title = "The driver brakes within 10 steps of a collision warning"
criticality = "critical"
message = "collision warning was not followed by braking"
given = "warning"
check = "(once[:10] {brake})"

# A string that stands alone is already a reference; braces interpolate one
# into an expression, as `{brake}` above and `{speeding}` below do.
[[properties]]
name = "in_gear_while_moving"
title = "The vehicle is never in motion out of gear"
given = "moving"
check = "in_gear"

# No antecedent, so this one must hold at every step.
[[properties]]
name = "speed_within_limit"
title = "Speed stays under the configured limit"
criticality = "warning"
check = "(not {speeding})"
```

The format is specified by a JSON Schema and comes with a validator that also checks the semantic rules a schema cannot express. Both live in the reference repository, alongside a set of annotated example documents covering each feature of the format.

{{< cards >}}
{{< card title="vevalabs/ryspec" subtitle="Schema, validator and documentation on GitHub" link="https://github.com/vevalabs/ryspec" icon="github" >}}
{{< card title="Examples" subtitle="Annotated specifications covering each feature of the format" link="/docs/ryspec/examples/" icon="book-open" >}}
{{< /cards >}}
