---
name: ir-report-writing
description: IR Report Writing rules. Use only when a pipeline step's Input lists this skill.
---

[IR Report Writing]
Record in ir_report.yaml what you decided yourself while building the IR because it could not be determined from the SRS alone.

# Output
runs/ir_report.yaml — follows the format of ir_report_schema.yaml.

# Conditions
1. Write one entry per decision.
2. Number ids according to the IR being built, such as DEC-STATIC-01 or DEC-DYNAMIC-01.
3. For a decision not reflected in the IR, leave target as an empty list.
4. If ir_report.yaml already exists, append entries to the end of the file without reading its contents.
5. Record every decision the IR makes that the SRS does not determine, however minor (for example, values, comparisons, orderings, defaults, and so on). Write each as concretely as the IR, with the exact details the IR uses for it.
