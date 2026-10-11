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
