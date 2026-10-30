---
name: ml-tc-verdict
description: Pipeline step 10 (ML TC Reverse Interpretation Verdict). Use only when the Master assigns step 10.
---

[ML TC Reverse Interpretation Verdict]
This step checks whether the ML TC is semantically correct by comparing ml_tc_nl.yaml with the high-level test cases and the ML TC report in both directions.
ml_tc_nl.yaml is a natural-language explanation of the ML TC, written without seeing the high-level test cases or the ML TC report.
The ML TC was built from the high-level test cases by adding the decisions recorded in the ML TC report.

# Sub-steps
Run the following skills in order. Run each one with a new subagent.
1. the ml-tc-verdict-a skill
2. the ml-tc-verdict-b skill
3. the ml-tc-verdict-c skill

# Output
runs/ml_tc_verdict_a.yaml, runs/ml_tc_verdict_b.yaml, runs/ml_tc_verdict.yaml

# Conditions
1. Do not perform the sub-steps yourself, and do not change their output files.
2. Have each subagent also follow the output-format skill.
