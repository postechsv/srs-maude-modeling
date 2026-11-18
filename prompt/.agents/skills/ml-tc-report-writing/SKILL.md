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
4. Record every decision the ML TC makes that the high-level test case does not determine, however minor (for example, values, initial states, expected states, orderings, and so on). Write each as concretely as the ML TC, with the exact details the ML TC uses for it, and set target to every test case that uses it.
