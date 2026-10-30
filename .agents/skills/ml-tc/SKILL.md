---
name: ml-tc
description: Pipeline step 07 (ML TC Generation). Use only when the Master assigns step 07.
---

[ML TC Generation]
Rewrite each high-level test case in natural language, using the terms of the IR.

Mode:
- Revision: the last entry of feedback/ml_tc.yaml has the same version as the latest history/vN/.
- Generation: otherwise.
Check only that version to choose the mode. Always follow Common Conditions, then the section for the mode.

# Input
inputs/hl_tc.yaml, runs/static_comp.yaml, runs/dynamic_comp.yaml, schemas/ml_tc_report_schema.yaml, the ml-tc-report-writing skill
For revision, also:
- feedback/ml_tc.yaml: problems found in previous ML TCs. Each entry has its history version and failed step. Its ids and IR terms refer to that version.
- ml_tc.yaml and ml_tc_report.yaml in the latest history/vN/.
- To understand a feedback entry, if needed: the history/vN/ folder of its version.

# Output
runs/ml_tc.yaml — in the following format.

- id: <id of the high-level test case>
  scenario: <the scenario, in the terms of the IR>
  expected: <the expected result, in the terms of the IR>

runs/ml_tc_report.yaml — written according to the ml-tc-report-writing skill.

# Common Conditions
1. Read only the files listed in Input.
2. Use only expressions defined in the IR. Do not create new expressions.

# Generation
1. Build the ML TC from the high-level test cases and the current IR. Do not use feedback/ml_tc.yaml.

# Revision
1. Take ml_tc.yaml and ml_tc_report.yaml from the latest history/vN/, and edit them instead of writing new ones.
2. Make sure none of the problems in feedback/ml_tc.yaml appear in the result.
3. Fix only the parts this requires and the parts those fixes affect. Keep everything else unchanged.
