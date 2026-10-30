---
name: ir-verdict
description: Pipeline step 06 (IR Reverse Interpretation Verdict). Use only when the Master assigns step 06.
---

[IR Reverse Interpretation Verdict]
This step checks whether the IR is semantically correct by comparing ir_nl.yaml with the SRS and the report in both directions.
ir_nl.yaml is a natural-language explanation of the IR, written without seeing the SRS or the report.
The IR was built from the SRS by adding the decisions recorded in the report.

# Sub-steps
Run the following skills in order. Run each one with a new subagent.
1. the ir-verdict-a skill
2. the ir-verdict-b skill
3. the ir-verdict-c skill

# Output
runs/ir_verdict_a.yaml, runs/ir_verdict_b.yaml, runs/ir_verdict.yaml

# Conditions
1. Do not perform the sub-steps yourself, and do not change their output files.
2. Have each subagent also follow the output-format skill.
