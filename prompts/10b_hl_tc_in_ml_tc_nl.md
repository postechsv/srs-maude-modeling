[ML TC Verdict B: HL TC ⊆ ml_tc_nl.yaml]
Check whether everything the high-level test cases state is contained in ml_tc_nl.yaml.
A statement is each item under statements in ml_tc_nl.yaml.

# Input
inputs/hl_tc.yaml, runs/ml_tc_nl.yaml

# Output
runs/ml_tc_verdict_b.yaml — in the following format.

pass: <true only if every test case is supported; otherwise false>
results:
  - id: <test case id>
    evidence: [<N ids that this test case matches>]
    supported: <true | false>
    reason: <in detail, whether supported is true or false: what the test case states, and which statements state each part of it, or which part no statement states>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the ML TC, the IR, the SRS, and the ML TC report.
2. Check every test case of hl_tc.yaml in order.
3. A test case is supported if everything its scenario and expected state semantically matches what the statements for that test case state. List all the statements it matches in evidence. If any part of the test case matches no statement, set supported to false.
4. If no statement matches the test case, write evidence: [] and set supported to false.
