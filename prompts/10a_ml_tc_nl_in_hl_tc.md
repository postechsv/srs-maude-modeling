[ML TC Verdict A: ml_tc_nl.yaml → HL TC + ML TC Report]
Describe how everything ml_tc_nl.yaml states corresponds to the high-level test cases or the ML TC report. Do not judge whether a statement is acceptable.
Treat the high-level test cases and the ML TC report together as one description of the intended tests.
A statement is each item under statements in ml_tc_nl.yaml.

# Input
inputs/hl_tc.yaml, runs/ml_tc_report.yaml, runs/ml_tc_nl.yaml

# Output
runs/ml_tc_verdict_a.yaml — in the following format.

results:
  - id: <N id>
    evidence: [<test case ids and report ids that the statement corresponds to>]
    mapping: <in detail: split the statement into its parts, and for each part, quote the exact phrase in its test case or a report entry for that test case that corresponds to it, name that test case or entry, and describe any difference in wording or scope; for a part with no corresponding phrase, state that nothing corresponds to it>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the ML TC, the IR, and the SRS.
2. Describe every statement of ml_tc_nl.yaml in order.
3. Only describe correspondences and differences. Do not decide whether the statement is supported, and do not write pass or fail.
4. If nothing corresponds to the statement, write evidence: [] and state so in mapping.
