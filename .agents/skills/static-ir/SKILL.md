---
name: static-ir
description: Pipeline step 02 (Static IR Generation). Use only when the Master assigns step 02.
---

[Static IR Generation]
Read the SRS and build the Static IR.

# Input
inputs/srs_numbered.yaml, schemas/static_comp_schema.yaml, schemas/report_schema.yaml, the report-writing skill, runs/feedback.yaml (only if it exists)

For repair and regression checking, also read the following files when present in this experiment only: runs/report.yaml; history/vN/static_comp.yaml, history/vN/dynamic_comp.yaml, history/vN/report.yaml, history/vN/ir_nl.yaml, history/vN/report_check.yaml, history/vN/ir_verdict_a.yaml, history/vN/ir_verdict_b.yaml, history/vN/ir_verdict.yaml, history/vN/maude_result.yaml, history/vN/feedback.yaml. Here vN ranges over existing version directories, ordered by numeric N; these files are read-only.

# Output
runs/static_comp.yaml — follows the format of static_comp_schema.yaml.
runs/report.yaml — written according to the report-writing skill.

# Conditions
1. Number ids so that no id is duplicated within the file.
2. In trace, list the SRS sentences that directly support the element.
3. For each detail the SRS leaves open, retain a compatible prior decision when one exists; otherwise choose a concrete decision that preserves every applicable SRS obligation. Reflect it in the IR, including required declarations, and record its exact meaning according to the report-writing skill. Do not add restrictions that exclude behavior the SRS requires.
4. Put one blank line between the top-level keys (classes, relationships, enumerations, messages, constraints).
5. When a type name alone does not fully specify its meaning or representation, include the missing details in parentheses after the type name in the type field. (ex. Date (YYYY-MM-DD))
6. List the recipient as the first parameter of each message.
7. Values that the system needs to perform the behaviors in the SRS are declared: values obtained from outside are received as parameters of the request message, and values kept internally are declared as attributes; nothing is supplied by an execution environment outside the IR.
8. Do not declare results or records that the SRS does not require.
9. Apply the repair and regression checks below before completing the IR and its report.
10. Distinguish the following two kinds of conditions, and declare only constraints in constraints.
   - constraint: the uniqueness of a value that identifies an object among the objects of its class, which every state of the system must satisfy. Write it as a condition on the state, not as a check performed by an operation.
   - rule condition: any other condition, such as a limit on a value supplied by a request or a restriction on how objects may be related to each other. It is checked only when a request is accepted and is written in the Dynamic IR, not in constraints.

11. On a retry, inspect the prior Dynamic IR and its findings before changing declarations so the corrected rules can be expressed using this Static IR. Reconstruct the report entries for the current Static IR; leave Dynamic IR report reconciliation to step 03.

# Repair and regression checks
1. Before generating, read the listed report_check.yaml, ir_verdict.yaml, maude_result.yaml, and feedback.yaml across all historical versions, even if runs/feedback.yaml is absent. Consult the detailed verdict_a/verdict_b files only as needed to inspect evidence or prior supported behavior. Build an in-memory checklist of distinct issues: source version, affected SRS clause and IR element/field, violated obligation, counterexample or boundary, and current status. Resolve an N id through ir_nl.yaml from the same version; do not assume N ids or IR ids retain their meaning across versions. Merge repeated issues by meaning. Read the associated listed artifacts as needed to verify each issue and any claimed repair.
2. Treat runs/feedback.yaml as the newest observation, not the whole history. Check each issue against the current SRS and candidate; mark it resolved only with concrete evidence in the candidate, retain resolved obligations as regression checks, and distinguish contradictory or superseded feedback using the SRS and artifact contents. A missing finding, an unexecuted check, or an old pass alone does not prove correctness.
3. Use the newest snapshot containing this stage's prior IR and report as the repair baseline. If the newest snapshot is incomplete, use the nearest earlier suitable snapshot while retaining all newer findings. Generate from scratch only if no baseline exists. Repair only affected elements and their dependencies; preserve unaffected text, ids, names, ordering, and decisions. Changes to shared types, units, formulas, or messages require checking every dependent use. Produce the complete normal output, not a patch file.
4. Classify each finding before editing: an SRS contradiction or missing behavior requires an IR repair; a sound IR decision missing from the report requires a precise report repair; disagreement between IR and report requires reconciling both against the SRS. If a reverse statement misreads the IR, retain the correct semantics and clarify the affected IR wording. Do not remove required behavior, invent report justification for an SRS conflict, or redesign unrelated parts to silence a finding.
5. Before finishing, compare the candidate with the baseline and justify each semantic difference by an issue or its dependency. Recheck all accumulated obligations, previously supported behavior, and the affected boundary cases against the complete candidate. Keep the checklist in memory; write only the files listed in Output, and never modify history or feedback.
