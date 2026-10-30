---
name: ml-tc-report-writing
description: ML TC Report Writing rules. Use only when a pipeline step's Input lists this skill.
---

[ML TC Report Writing]
Record in ml_tc_report.yaml what you wrote in the ML TC because it could not be determined from the high-level test cases alone.

# Output
runs/ml_tc_report.yaml — follows the format of ml_tc_report_schema.yaml.

# Conditions
1. Write one entry per decision.
2. Number ids as DEC-MLTC-01, DEC-MLTC-02, ...
3. For a decision not reflected in the ML TC, leave target as an empty list.
4. On revision, retain applicable baseline entries and their ids, and update or remove superseded decisions instead of regenerating unaffected entries or appending contradictory replacements. Keep target aligned with the affected test case ids.
5. State every choice beyond the high-level test case explicitly enough for a reviewer who cannot read the IR or ML TC: concrete values and bounds, initial conditions, operation choices, ordering, and added observations or expectations. Do not rely on an IR rule name alone to explain its behavior. Check that every added ML TC detail has high-level test case support or a matching report decision, and that every retained decision is reflected in its target cases.
6. If feedback identifies missing report evidence for an intentional detail consistent with the high-level test case and current IR, complete the report without changing the test. If the test conflicts with either input, repair the affected test and decision; do not justify the conflict by adding report text.
