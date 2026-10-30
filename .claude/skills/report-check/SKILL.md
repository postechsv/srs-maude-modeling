---
name: report-check
description: Pipeline step 04 (Report Consistency Check). Use only when the Master assigns step 04.
---

[Report Consistency Check]
Check whether the decisions recorded in the report conflict with the SRS.

# Input
inputs/srs_numbered.yaml, runs/report.yaml

# Output
runs/report_check.yaml — in the following format.

pass: <true | false>
conflicts:
  - report_id: <id of the conflicting entry in report.yaml, e.g. DEC-STATIC-01>
    srs: [<S ids>]
    reason: <what the decision was, and how it makes these SRS sentences false>

# Conditions
1. Check every entry in report.yaml.
2. A decision conflicts with the SRS if applying the decision would make an SRS sentence false. Check only this direction; do not check whether the SRS implies the decision. A decision that makes something the SRS leaves open more specific is not a conflict.
3. Set pass to true only if there are no conflicts. If there are none, write conflicts: [].
