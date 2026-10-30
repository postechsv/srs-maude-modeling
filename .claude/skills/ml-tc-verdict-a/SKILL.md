---
name: ml-tc-verdict-a
description: Pipeline step 10a (ML TC Verdict A: ml_tc_nl.yaml ⊆ HL TC + ML TC Report). Use only when the Master assigns step 10a.
---

[ML TC Verdict A: ml_tc_nl.yaml ⊆ HL TC + ML TC Report]
Check whether everything ml_tc_nl.yaml states is supported by the high-level test cases or the ML TC report.
Treat the high-level test cases and the ML TC report together as one description of the intended tests.
A statement is each item under statements in ml_tc_nl.yaml.

# Input
inputs/hl_tc.yaml, runs/ml_tc_report.yaml, runs/ml_tc_nl.yaml

# Output
runs/ml_tc_verdict_a.yaml — in the following format.

pass: <true only if every statement is supported; otherwise false>
results:
  - id: <N id>
    evidence: [<test case ids and report ids that support this statement>]
    supported: <true | false>
    reason: <in detail, whether supported is true or false: split the statement into its parts, and for each part, quote the exact phrase in the test case or a report entry that supports it, name that test case or entry, and explain how the phrase supports the part, including any difference in wording or scope; when supported is false, state which part nothing supports, or what the test case or report states instead>

# Conditions
1. Read only the files listed in Input. Do not read or refer to any other file, including the ML TC, the IR, and the SRS.
2. Check every statement of ml_tc_nl.yaml in order.
3. A statement is supported if what it states semantically matches what its test case in hl_tc.yaml or the ML TC report entries for that test case state. List the test case or report entries it matches in evidence.
4. If nothing supports the statement, write evidence: [] and set supported to false.
5. Evaluate each atomic claim using the allowed sources together. Exact wording need not match: accept a logical or arithmetic consequence only when all premises are stated in those sources, and show the derivation in reason. A report target id or "as declared in the ML TC or IR" does not supply an unstated value, bound, formula, or ordering. Do not infer such choices from conventions or read the referenced artifact.
6. Retain evidence for supported parts when another part is unsupported, and keep supported false if any part lacks support or conflicts with the intended description. In reason, identify the exact missing or conflicting detail and distinguish missing report evidence from a contradiction of the high-level test case or report. Cite conflicting source ids in reason even when they are not supporting evidence. Do not prescribe a new design, treat a report decision as overriding the high-level test case, or relax the pass criterion.
