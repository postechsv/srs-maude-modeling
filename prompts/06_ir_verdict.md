[IR Reverse Interpretation Verdict]
This step checks whether the IR is semantically correct.
Treat the SRS and the report together as one description of the intended system.
Compare ir_nl.md with this combined description, and judge whether they describe the same system.
ir_nl.md is a natural-language explanation of the IR, written without seeing the SRS or the report.
The IR was built from the SRS by adding the decisions recorded in the report.

# Input
inputs/srs_numbered.yaml, runs/report.yaml, runs/ir_nl.md

# Output
runs/ir_verdict.yaml — in the following format. Write findings only when pass is false.

pass: <true only if there is nothing left to fix; otherwise false>
findings:
  - ids: [<S ids, report ids, or N ids involved>]
    reason: <what ir_nl.md says (or omits), and what the SRS or report says instead>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR.
