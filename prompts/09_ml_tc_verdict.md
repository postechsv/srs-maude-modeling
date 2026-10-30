[ML TC Verdict]
This step checks whether the ML TC is semantically correct.
Compare each high-level test case with its explanation in ml_tc_nl.md, and judge whether they describe the same test.
ml_tc_nl.md explains the ML TC, which rewrote the high-level test cases in the terms of the IR. It was written without seeing the high-level test cases.

# Input
inputs/hl_tc.yaml, runs/ml_tc_nl.md

# Output
runs/ml_tc_verdict.yaml — in the following format. Write findings only when pass is false.

pass: <true only if there is nothing left to fix; otherwise false>
findings:
  - id: <id of the high-level test case>
    reason: <what the high-level test case says, and what ml_tc_nl.md says (or omits) instead>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the ML TC and the IR.
