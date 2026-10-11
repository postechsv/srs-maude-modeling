[ML TC Generation]
Rewrite each high-level test case in natural language, using the terms of the IR.

# Input
inputs/hl_tc.yaml, runs/static_comp.yaml, runs/dynamic_comp.yaml, schemas/ml_tc_report_schema.yaml, prompts/ml_tc_report.md, runs/feedback.yaml (only if it exists)

# Output
runs/ml_tc.yaml — in the following format.

- id: <id of the high-level test case>
  scenario: <the scenario, in the terms of the IR>
  expected: <the expected result, in the terms of the IR>

runs/ml_tc_report.yaml — written according to ml_tc_report.md.

# Conditions
1. Read only the files listed in Input.
2. Use only expressions defined in the IR. Do not create new expressions.
3. If runs/feedback.yaml exists, it describes problems found in a previous ML TC. Build the ML TC so that none of these problems remain.
