---
name: ml-tc-verdict-c
description: Pipeline step 10c (ML TC Verdict C: Final Verdict). Use only when the Master assigns step 10c.
---

[ML TC Verdict C: Final Verdict]
Combine the results of 10a and 10b into the final verdict.

# Input
runs/ml_tc_verdict_a.yaml, runs/ml_tc_verdict_b.yaml

# Output
runs/ml_tc_verdict.yaml — in the following format. Write findings only when pass is false.

pass: <true only if every result in both files has supported: true; otherwise false>
findings:
  - id: <N id or test case id>
    evidence: [<as written in the result>]
    reason: <as written in the result>

# Conditions
1. Read only the files listed in Input.
2. Collect every result whose supported is false from both files into findings. Copy id, evidence, and reason as they are. Do not add, remove, or change any judgment.
