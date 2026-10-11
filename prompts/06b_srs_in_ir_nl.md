[IR Verdict B: SRS ⊆ ir_nl.yaml]
Check whether everything the SRS states is contained in ir_nl.yaml.
A statement is each item under statements in ir_nl.yaml.

# Input
inputs/srs_numbered.yaml, runs/ir_nl.yaml

# Output
runs/ir_verdict_b.yaml — in the following format.

pass: <true only if every sentence is supported; otherwise false>
results:
  - id: <S id>
    evidence: [<N ids that this sentence matches>]
    supported: <true | false>
    reason: <only when supported is false: which part of the sentence is missing or stated differently in ir_nl.yaml>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR and the report.
2. Check every sentence of the SRS in order.
3. A sentence is supported if everything it states semantically matches what ir_nl.yaml states. List all the statements it matches in evidence. If any part of the sentence matches no statement, set supported to false.
4. If no statement matches the sentence, write evidence: [] and set supported to false.
