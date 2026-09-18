---
title: Examples
weight: 1
---

The pages in this section walk through the reference examples from the [`ryspec` repository](https://github.com/vevalabs/ryspec/tree/main/examples), ordered from the smallest useful specification to a fully declared monitor. Each document is valid on its own and is checked by the project's test suite, so any of them is a reliable starting point for a new specification.

Every example opens with a `#:schema` comment pointing at the bundled JSON Schema. A TOML-aware editor uses it for completion and inline errors; other tools read it as a comment.

{{< cards >}}
{{< card title="A single property" subtitle="One property, one condition, everything else deduced" link="basic-property/" >}}
{{< card title="Several properties" subtitle="Independent verdicts over a shared vocabulary" link="multiple-properties/" >}}
{{< card title="Shared rules" subtitle="Named subformulas lifted out of the properties" link="shared-rules/" >}}
{{< card title="Prefix form" subtitle="Operator-first arrays and property-private rules" link="prefix-form/" >}}
{{< card title="Variables and a monitor" subtitle="Declared types, units, parameters and interface order" link="variables-and-monitor/" >}}
{{< card title="Structured input" subtitle="Projecting signals out of a JSON telemetry frame" link="sources-json/" >}}
{{< card title="Published outputs" subtitle="Exposing a computed condition, not only a verdict" link="published-outputs/" >}}
{{< card title="Metadata and extras" subtitle="Identifying a document and carrying process data" link="meta-and-extras/" >}}
{{< /cards >}}
