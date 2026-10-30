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
4. When repairing, merge decisions from the invoking stage's permitted baseline report by test-case id and meaning. Preserve unchanged ids and text, update changed entries in place, allocate unused ids for new decisions, and remove obsolete or conflicting entries. Describe only the current ML TC, not the retry history.
5. Make each decision self-contained for a reviewer who cannot read the ML TC or IR. Explicitly state chosen initial values, units, boundaries, freshness assumptions, request ordering, concrete sort keys and directions, and the meaning of any IR-specific behavior used by the test. "As defined in RULE-XX" is not evidence for details omitted from this report.
6. Audit every ML TC detail not entailed by its high-level test, including details imported from the IR, against a decision at the same precision. Set target to every affected test-case id; evidence for one case does not automatically apply to another. Check the reverse direction too: each targeted decision must match the current scenario or expected result. For target: [], state that the decision is not implemented without promising test behavior.
7. Preserve the high-level test's intent and coverage. A report may explain a compatible concretization, but cannot justify weakened assertions, changed expected results, or a restriction that excludes a required scenario. Keep these explanations within the existing issue and decision fields.
