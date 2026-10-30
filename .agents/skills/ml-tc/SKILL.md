---
name: ml-tc
description: Pipeline step 07 (ML TC Generation). Use only when the Master assigns step 07.
---

[ML TC Generation]
Rewrite each high-level test case in natural language, using the terms of the IR.

# Input
inputs/hl_tc.yaml, runs/static_comp.yaml, runs/dynamic_comp.yaml, schemas/ml_tc_report_schema.yaml, the ml-tc-report-writing skill, runs/feedback.yaml (only if it exists)

For repair and regression checking, also read these files when present in this experiment only: history/vN/ml_tc.yaml, history/vN/ml_tc_report.yaml, history/vN/ml_tc_nl.yaml, history/vN/ml_tc_report_check.yaml, history/vN/ml_tc_verdict_a.yaml, history/vN/ml_tc_verdict_b.yaml, history/vN/ml_tc_verdict.yaml, history/vN/feedback.yaml, history/vN/static_comp.yaml, history/vN/dynamic_comp.yaml. Order existing vN directories by numeric N; history is read-only.

# Output
runs/ml_tc.yaml — in the following format.

- id: <id of the high-level test case>
  scenario: <the scenario, in the terms of the IR>
  expected: <the expected result, in the terms of the IR>

runs/ml_tc_report.yaml — written according to the ml-tc-report-writing skill.

# Conditions
1. Read only the files listed in Input.
2. Use only expressions defined in the IR. Do not create new expressions.
3. Apply the repair and regression checks below before completing the ML TC and its report.

# Repair and regression checks
1. Read ml_tc_report_check.yaml, ml_tc_verdict.yaml, and relevant feedback across all historical versions, not only runs/feedback.yaml. Consult the detailed verdict_a/verdict_b files as needed to inspect evidence or prior supported behavior. In memory, group issues by test-case id and semantic obligation; retain their version, failing detail, boundary example, and current status. Resolve N ids using the same version's ml_tc_nl.yaml. Verify repairs in the actual artifacts and retain resolved obligations as regression checks; absence from later feedback or an unexecuted check is not evidence of resolution.
2. Start with the newest snapshot containing the prior ML TC and report; fall back to the nearest suitable earlier snapshot if necessary, retaining newer findings. Compare that snapshot's IR with the current IR and include affected test cases in the repair scope. Current high-level tests and current IR take precedence over historical artifacts; use only the current IR vocabulary. Generate from scratch only when no baseline exists.
3. Modify only failing test cases, affected IR dependencies, and their report entries; retain unrelated cases, ids, wording, and decisions. For a valid concretization missing from the report, repair the report instead of replacing the scenario. Do not change expected outcomes, weaken assertions, or narrow a high-level scenario to make the model pass. Reconcile actual semantic conflicts against the high-level test and current IR.
4. Before finishing, check every high-level scenario and expected result against the candidate, all accumulated issues and boundary cases, and the semantic differences from the baseline. Produce the complete normal outputs; keep repair notes in memory and do not modify history or feedback.
