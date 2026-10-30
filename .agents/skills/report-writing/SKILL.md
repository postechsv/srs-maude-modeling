---
name: report-writing
description: Report Writing rules. Use only when a pipeline step's Input lists this skill.
---

[Report Writing]
Record in report.yaml what you decided yourself while building the IR because it could not be determined from the SRS alone.

# Output
runs/report.yaml — follows the format of report_schema.yaml.

# Conditions
1. Write one entry per decision.
2. Number ids according to the IR being built, such as DEC-STATIC-01 or DEC-DYNAMIC-01.
3. For a decision not reflected in the IR, leave target as an empty list.
4. If report.yaml already exists, append entries to the end of the file without reading its contents.
