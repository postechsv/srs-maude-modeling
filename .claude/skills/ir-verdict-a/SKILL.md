---
name: ir-verdict-a
description: Pipeline step 06a (IR Verdict A: ir_nl.yaml ⊆ SRS + IR Report). Use only when the Master assigns step 06a.
---

[IR Verdict A: ir_nl.yaml ⊆ SRS + IR Report]
Check whether everything ir_nl.yaml states is supported by the SRS or the IR report.
Treat the SRS and the IR report together as one description of the intended system.
A statement is each item under statements in ir_nl.yaml.

# Input
inputs/srs_numbered.yaml, runs/ir_report.yaml, runs/ir_nl.yaml

# Output
runs/ir_verdict_a.yaml — in the following format.

pass: <true only if every statement is supported; otherwise false>
results:
  - id: <N id>
    evidence: [<S ids and IR report ids that support this statement>]
    unrelated: [<every other S id and IR report id, each compared with this statement and found not to support it>]
    supported: <true | false>
    reason: <in detail, whether supported is true or false: split the statement into its parts, and for each part, quote the exact phrase in an SRS sentence or IR report entry that supports it, name that sentence or entry, and explain how the phrase supports the part, including any difference in wording or scope; when supported is false, state which part nothing supports, or what the SRS or IR report states instead>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR.
2. Check every statement of ir_nl.yaml in order.
3. A statement is supported if what it states semantically matches what the SRS or the IR report states. List the SRS sentences or IR report entries it matches in evidence.
4. If nothing supports the statement, write evidence: [] and set supported to false.
5. Compare each statement with every SRS sentence and every IR report entry, one at a time. Put each id in evidence if it supports the statement, and in unrelated otherwise. Every S id and every IR report id must appear in exactly one of evidence and unrelated.
