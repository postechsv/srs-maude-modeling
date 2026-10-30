---
name: ml-tc
description: Pipeline step 07 (ML TC Generation). Use only when the Master assigns step 07.
---

[ML TC Generation]
Rewrite each high-level test case in natural language, using the terms of the IR.

# Input
inputs/hl_tc.yaml, runs/static_comp.yaml, runs/dynamic_comp.yaml, schemas/ml_tc_report_schema.yaml, the ml-tc-report-writing skill, runs/feedback.yaml (only if it exists)
For revision context, also read this experiment's existing history/vN/ files (read-only): static_comp.yaml, dynamic_comp.yaml, ml_tc.yaml, ml_tc_report.yaml, ml_tc_nl.yaml, ml_tc_report_check.yaml, ml_tc_verdict*.yaml.

# Output
runs/ml_tc.yaml — in the following format.

- id: <id of the high-level test case>
  scenario: <the scenario, in the terms of the IR>
  expected: <the expected result, in the terms of the IR>

runs/ml_tc_report.yaml — written according to the ml-tc-report-writing skill.

# Conditions
1. Read only the files listed in Input.
2. Use only expressions defined in the IR. Do not create new expressions.
3. Use runs/feedback.yaml and every archived ML TC check and verdict to form an in-memory checklist of distinct problems, including previously fixed ones. Resolve findings against the ML TC, report, and reverse interpretation from the same version; do not match N ids across versions. Reconcile conflicting feedback against the high-level test cases and current IR.
   - If an archived ml_tc.yaml and ml_tc_report.yaml pair exists, use the latest such snapshot as the revision baseline, ordering N numerically. Compare its IR with the current IR before reusing tests. Repair only implicated test cases, decisions, and cases affected by changed IR definitions; preserve unaffected content and ids. Write the complete revised outputs to runs/ even when the previous files were deleted; do not modify history or create a checklist file.
   - Before finishing, check every applicable checklist item, including earlier fixes, and confirm that all high-level scenarios and expectations remain represented without conflicting additions.
