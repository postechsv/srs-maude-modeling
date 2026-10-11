[ML TC Verdict C: Final Verdict]
Judge, from the correspondences written in 10a and 10b, whether ml_tc_nl.yaml and the high-level test cases with the ML TC report describe the same tests.

# Input
inputs/hl_tc.yaml, runs/ml_tc_report.yaml, runs/ml_tc_nl.yaml, runs/ml_tc_verdict_a.yaml, runs/ml_tc_verdict_b.yaml

# Output
runs/ml_tc_verdict.yaml — in the following format. Write findings only when pass is false.

pass: <true only if every result is supported; otherwise false>
results:
  - id: <N id or test case id>
    supported: <true | false>
    reason: <why the correspondences and differences written in the mapping are or are not acceptable>
findings:
  - id: <N id or test case id whose supported is false>
    evidence: [<as written in 10a or 10b>]
    reason: <as written in results>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the ML TC, the IR, and the SRS.
2. Judge every result in ml_tc_verdict_a.yaml and ml_tc_verdict_b.yaml. Check each mapping against the quoted test cases, entries, or statements.
3. A result is supported only if every part has a corresponding phrase and no described difference changes the meaning. A part with no corresponding phrase, or a difference that changes the meaning, makes it not supported.
4. Apply the same criterion to every result.
