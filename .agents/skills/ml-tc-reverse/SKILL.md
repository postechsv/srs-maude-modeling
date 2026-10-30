---
name: ml-tc-reverse
description: Pipeline step 09 (ML TC Reverse Interpretation). Use only when the Master assigns step 09.
---

[ML TC Reverse Interpretation]
Read the ML TC and explain each test case in detail in natural language.
Use the IR only to understand the terms used in the ML TC.

# Input
runs/ml_tc.yaml, runs/static_comp.yaml, runs/dynamic_comp.yaml

# Output
runs/ml_tc_nl.yaml — in the following format.

statements:
  - id: N-01
    test_case: <id of the test case>
    text: <one statement>
  - id: N-02
    test_case: <id of the test case>
    text: <one statement>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the high-level test cases, the SRS, and the reports.
2. Preserve each test's concrete initial conditions, input values, units, boundaries, request order, quantifiers, and expected observations. Use IR definitions only to explain expressions the ML TC actually uses; do not import additional preconditions, assertions, or behavior merely because they exist elsewhere in the IR.
3. Keep statements tied to their test_case and separate independent claims where practical without detaching conditions from their consequences. Check both that the statements are entailed by that ML TC with the permitted IR definitions and that every scenario and expected-result detail is represented. Do not read history or feedback to shape the interpretation.
