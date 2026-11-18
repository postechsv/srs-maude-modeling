---
name: ml-tc-report-check
description: Pipeline step 08 (ML TC Report Consistency Check). Use only when the Master assigns step 08.
---

[ML TC Report Consistency Check]
Check whether the decisions recorded in the ML TC report conflict with the high-level test cases.

# Input
inputs/hl_tc.yaml, runs/ml_tc_report.yaml

# Output
runs/ml_tc_report_check.yaml — in the following format.

pass: <true | false>
conflicts:
  - report_id: <id of the conflicting entry in ml_tc_report.yaml, e.g. DEC-MLTC-01>
    test_cases: [<test case ids>]
    reason: <what the decision was, and how it makes these high-level test cases false>

# Conditions
1. Check every entry in ml_tc_report.yaml.
2. A decision conflicts with the high-level test cases if applying the decision would make a high-level test case false. Check only this direction; do not check whether the high-level test cases imply the decision. A decision that makes something the high-level test cases leave open more specific is not a conflict.
3. Set pass to true only if there are no conflicts. If there are none, write conflicts: [].
