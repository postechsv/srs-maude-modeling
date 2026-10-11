[IR Reverse Interpretation Verdict]
This step checks whether the IR is semantically correct by comparing ir_nl.yaml with the SRS and the report in both directions.
ir_nl.yaml is a natural-language explanation of the IR, written without seeing the SRS or the report.
The IR was built from the SRS by adding the decisions recorded in the report.

# Sub-steps
Run the following prompts in order. Run each one with a new subagent.
1. prompts/06a_ir_nl_in_srs.md
2. prompts/06b_srs_in_ir_nl.md
3. prompts/06c_ir_final_verdict.md

# Output
runs/ir_verdict_a.yaml, runs/ir_verdict_b.yaml, runs/ir_verdict.yaml

# Conditions
1. Do not perform the sub-steps yourself, and do not change their output files.
