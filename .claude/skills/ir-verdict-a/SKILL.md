---
name: ir-verdict-a
description: Pipeline step 06a (IR Verdict A: ir_nl.yaml ⊆ SRS + Report). Use only when the Master assigns step 06a.
---

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
    unrelated: [<every other S id and report id, each compared with this statement and found not to support it>]
    supported: <true | false>
    reason: <in detail, whether supported is true or false: split the statement into its parts, and for each part, quote the exact phrase in an SRS sentence or report entry that supports it, name that sentence or entry, and explain how the phrase supports the part, including any difference in wording or scope; when supported is false, state which part nothing supports, or what the SRS or report states instead>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the IR.
2. Check every statement of ir_nl.yaml in order.
3. A statement is supported if what it states semantically matches what the SRS or the report states. List the SRS sentences or report entries it matches in evidence.
4. If nothing supports the statement, write evidence: [] and set supported to false.
5. Compare each statement with every SRS sentence and every report entry, one at a time. Put each id in evidence if it supports the statement, and in unrelated otherwise. Every S id and every report id must appear in exactly one of evidence and unrelated.
6. Evaluate each atomic claim using the allowed sources together. Exact wording need not match: accept a logical or arithmetic consequence only when all premises are stated in those sources, and show the derivation in reason. A report target id or "as declared in the IR" does not supply an unstated value, bound, formula, or ordering. Do not infer such choices from conventions or read the referenced artifact.
7. Retain evidence for supported parts when another part is unsupported, and keep supported false if any part lacks support or conflicts with the intended description. In reason, identify the exact missing or conflicting detail and distinguish missing report evidence from a contradiction of the SRS or report. Cite conflicting source ids in reason even when they are not supporting evidence. Do not prescribe a new design, treat a report decision as overriding the SRS, or relax the pass criterion.
