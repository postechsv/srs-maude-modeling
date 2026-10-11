[IR Verdict C: Final Verdict]
Judge, from the correspondences written in 06a and 06b, whether ir_nl.yaml and the SRS with the report describe the same system.

# Input
inputs/srs_numbered.yaml, runs/report.yaml, runs/ir_nl.yaml, runs/ir_verdict_a.yaml, runs/ir_verdict_b.yaml

# Output
runs/ir_verdict.yaml — in the following format. Write findings only when pass is false.

pass: <true only if every result is supported; otherwise false>
results:
  - id: <N id or S id>
    supported: <true | false>
    reason: <why the correspondences and differences written in the mapping are or are not acceptable>
findings:
  - id: <N id or S id whose supported is false>
    evidence: [<as written in 06a or 06b>]
    reason: <as written in results>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR.
2. Judge every result in ir_verdict_a.yaml and ir_verdict_b.yaml. Check each mapping against the quoted sentences, entries, or statements.
3. A result is supported only if every part has a corresponding phrase and no described difference changes the meaning. A part with no corresponding phrase, or a difference that changes the meaning, makes it not supported.
4. Apply the same criterion to every result.
