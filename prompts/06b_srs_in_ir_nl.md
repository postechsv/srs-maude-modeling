[IR Verdict B: SRS → ir_nl.yaml]
Describe how everything the SRS states corresponds to ir_nl.yaml. Do not judge whether a sentence is acceptably covered.
A statement is each item under statements in ir_nl.yaml.

# Input
inputs/srs_numbered.yaml, runs/ir_nl.yaml

# Output
runs/ir_verdict_b.yaml — in the following format.

results:
  - id: <S id>
    evidence: [<N ids that the sentence corresponds to>]
    mapping: <in detail: split the sentence into its parts, and for each part, quote the exact phrase in a statement that corresponds to it, name that statement, and describe any difference in wording or scope; for a part with no corresponding phrase, state that nothing corresponds to it>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR and the report.
2. Describe every sentence of the SRS in order.
3. Only describe correspondences and differences. Do not decide whether the sentence is supported, and do not write pass or fail.
4. If no statement corresponds to the sentence, write evidence: [] and state so in mapping.
