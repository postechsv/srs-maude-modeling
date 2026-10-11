[ML TC Verdict A: ml_tc_nl.yaml ⊆ HL TC + ML TC Report]
Check whether everything ml_tc_nl.yaml states is supported by the high-level test cases or the ML TC report.
Treat the high-level test cases and the ML TC report together as one description of the intended tests.
A statement is each item under statements in ml_tc_nl.yaml.

# Input
inputs/hl_tc.yaml, runs/ml_tc_report.yaml, runs/ml_tc_nl.yaml

# Output
runs/ml_tc_verdict_a.yaml — in the following format.

pass: <true only if every statement is supported; otherwise false>
results:
  - id: <N id>
    evidence: [<test case ids and report ids that support this statement>]
    supported: <true | false>
    reason: <in detail, whether supported is true or false: what the statement states, and how the test case or report entries in evidence state it or fail to state it>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the ML TC, the IR, and the SRS.
2. Check every statement of ml_tc_nl.yaml in order.
3. A statement is supported if what it states semantically matches what its test case in hl_tc.yaml or the ML TC report entries for that test case state. List the test case or report entries it matches in evidence.
4. If nothing supports the statement, write evidence: [] and set supported to false.
