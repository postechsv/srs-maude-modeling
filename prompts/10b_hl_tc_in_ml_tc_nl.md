[ML TC Verdict B: HL TC → ml_tc_nl.yaml]
Describe how everything the high-level test cases state corresponds to ml_tc_nl.yaml. Do not judge whether a test case is acceptably covered.
A statement is each item under statements in ml_tc_nl.yaml.

# Input
inputs/hl_tc.yaml, runs/ml_tc_nl.yaml

# Output
runs/ml_tc_verdict_b.yaml — in the following format.

results:
  - id: <test case id>
    evidence: [<N ids that the test case corresponds to>]
    mapping: <in detail: split the scenario and expected of the test case into their parts, and for each part, quote the exact phrase in a statement for that test case that corresponds to it, name that statement, and describe any difference in wording or scope; for a part with no corresponding phrase, state that nothing corresponds to it>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the ML TC, the IR, the SRS, and the ML TC report.
2. Describe every test case of hl_tc.yaml in order.
3. Only describe correspondences and differences. Do not decide whether the test case is supported, and do not write pass or fail.
4. If no statement corresponds to the test case, write evidence: [] and state so in mapping.
