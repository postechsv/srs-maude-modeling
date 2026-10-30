---
name: ml-tc
description: Pipeline step 07 (ML TC Generation). Use only when the Master assigns step 07.
---

[ML TC Generation]
Rewrite each high-level test case in natural language, using the terms of the IR.

# Input
inputs/hl_tc.yaml, runs/static_comp.yaml, runs/dynamic_comp.yaml, schemas/ml_tc_report_schema.yaml, the ml-tc-report-writing skill, runs/feedback.yaml (only if it exists), history/ (only if it exists)

# Output
runs/ml_tc.yaml — in the following format.

- id: <id of the high-level test case>
  scenario: <the scenario, in the terms of the IR>
  expected: <the expected result, in the terms of the IR>

runs/ml_tc_report.yaml — written according to the ml-tc-report-writing skill.

# Conditions
1. Read only the files listed in Input.
2. Use only expressions defined in the IR. Do not create new expressions.
3. If runs/feedback.yaml exists, it describes problems found in a previous ML TC. The failed check results in every history/vN/ (ml_tc_report_check.yaml, ml_tc_verdict.yaml) also describe problems found in earlier ML TCs. Build the ML TC so that none of these problems remain.
   - In this case, start from the ML TC in the latest history/vN/. Fix only the parts these problems require and the parts affected by those fixes; keep all other parts unchanged.
