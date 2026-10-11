[IR Verdict A: ir_nl.yaml → SRS + Report]
Describe how everything ir_nl.yaml states corresponds to the SRS or the report. Do not judge whether a statement is acceptable.
Treat the SRS and the report together as one description of the intended system.
A statement is each item under statements in ir_nl.yaml.

# Input
inputs/srs_numbered.yaml, runs/report.yaml, runs/ir_nl.yaml

# Output
runs/ir_verdict_a.yaml — in the following format.

results:
  - id: <N id>
    evidence: [<S ids and report ids that the statement corresponds to>]
    mapping: <in detail: split the statement into its parts, and for each part, quote the exact phrase in an SRS sentence or report entry that corresponds to it, name that sentence or entry, and describe any difference in wording or scope; for a part with no corresponding phrase, state that nothing corresponds to it>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR.
2. Describe every statement of ir_nl.yaml in order.
3. Only describe correspondences and differences. Do not decide whether the statement is supported, and do not write pass or fail.
4. If nothing corresponds to the statement, write evidence: [] and state so in mapping.
