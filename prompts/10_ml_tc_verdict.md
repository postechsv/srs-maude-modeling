[ML TC Reverse Interpretation Verdict]
This step checks whether the ML TC is semantically correct by comparing ml_tc_nl.yaml with the high-level test cases and the ML TC report in both directions.
ml_tc_nl.yaml is a natural-language explanation of the ML TC, written without seeing the high-level test cases or the ML TC report.
The ML TC was built from the high-level test cases by adding the decisions recorded in the ML TC report.

# Sub-steps
Run the following prompts in order. Run each one with a new subagent.
1. prompts/10a_ml_tc_nl_in_hl_tc.md
2. prompts/10b_hl_tc_in_ml_tc_nl.md
3. prompts/10c_ml_tc_final_verdict.md

# Output
runs/ml_tc_verdict_a.yaml, runs/ml_tc_verdict_b.yaml, runs/ml_tc_verdict.yaml

# Conditions
1. Do not perform the sub-steps yourself, and do not change their output files.
2. Give each subagent, together with its prompt, the instructions under "# Instructions from the Master".
