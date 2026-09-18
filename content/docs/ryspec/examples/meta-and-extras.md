---
title: Metadata and extras
weight: 8
---

A document can identify itself and carry data belonging to the process around it.

```toml
#:schema ../schemas/v0/ryspec.schema.json

namespace = "https://example.com/ryspec/vehicle-braking"

[meta]
title = "Vehicle braking properties"
description = "Braking response checks for the reference vehicle model."
author = "John Doe"
license = "MIT"

[extras]
owner_team = "safety"
jira_project = "VEH"

[[properties]]
name = "brakes_after_warning"
title = "Brakes engage after a warning"
criticality = "critical"
message = "brake was not engaged within 10 steps of a warning"
check = ["once", "brake", { max = 10 }]
```

`namespace` is a URI identifying the document, which matters when properties from several documents are collected into one result set.

`meta` records facts about the document itself. `extras` is free-form data outside the document's metadata — ownership, ticket references, anything the surrounding process needs to travel with the specification. Both take arbitrary keys beyond the ones shown here.

The property adds its own reporting fields: `title` restates the requirement for a human reader, `criticality` classifies it for the consuming integration, and `message` is the text reported when the check fails. Together with `name`, these are what a monitor has to work with when it reports a violation, so they are worth filling in on any property that matters.

Source: [`examples/meta_and_extras.toml`](https://github.com/vevalabs/ryspec/blob/main/examples/meta_and_extras.toml)
