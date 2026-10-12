[IR Verdict A: ir_nl.yaml ⊆ SRS + Report]
Check whether everything ir_nl.yaml states is supported by the SRS or the report.
Treat the SRS and the report together as one description of the intended system.
A statement is each item under statements in ir_nl.yaml.

# Input
inputs/srs_numbered.yaml, runs/report.yaml, runs/ir_nl.yaml

# Output
runs/ir_verdict_a.yaml — in the following format.

pass: <true only if every statement is supported; otherwise false>
results:
  - id: <N id>
    evidence: [<S ids and report ids that support this statement>]
    supported: <true | false>
    reason: <in detail, whether supported is true or false: split the statement into its parts, and for each part, quote the exact phrase in an SRS sentence or report entry that supports it, name that sentence or entry, and explain how the phrase supports the part, including any difference in wording or scope; when supported is false, state which part nothing supports, or what the SRS or report states instead>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR.
2. Check every statement of ir_nl.yaml in order.
3. A statement is supported if what it states semantically matches what the SRS or the report states. List the SRS sentences or report entries it matches in evidence.
4. If nothing supports the statement, write evidence: [] and set supported to false.
