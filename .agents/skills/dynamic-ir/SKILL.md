---
name: dynamic-ir
description: Pipeline step 03 (Dynamic IR Generation). Use only when the Master assigns step 03.
---

[Dynamic IR Generation]
Read the SRS and build the Dynamic IR.

# Input
inputs/srs_numbered.yaml, runs/static_comp.yaml, schemas/dynamic_comp_schema.yaml, schemas/report_schema.yaml, the report-writing skill, runs/feedback.yaml (only if it exists)

For repair and regression checking, also read the following files when present in this experiment only: runs/report.yaml; history/vN/static_comp.yaml, history/vN/dynamic_comp.yaml, history/vN/report.yaml, history/vN/ir_nl.yaml, history/vN/report_check.yaml, history/vN/ir_verdict_a.yaml, history/vN/ir_verdict_b.yaml, history/vN/ir_verdict.yaml, history/vN/maude_result.yaml, history/vN/feedback.yaml. Here vN ranges over existing version directories, ordered by numeric N; these files are read-only.

# Output
runs/dynamic_comp.yaml — follows the format of dynamic_comp_schema.yaml.
runs/report.yaml — written according to the report-writing skill.

# Conditions
1. Each rule must describe one atomic state transition. Keep all effects that must occur together within the same rule. For the same message, use separate rules when different conditions lead to different effects.
2. Number ids so that no id is duplicated within the file.
3. In trace, list the SRS sentences that directly support the rule.
5. Use names, types, and terminology consistent with static_comp.yaml.
6. Use only the classes, attributes, relationships, enumerations, and messages declared in static_comp.yaml. Do not introduce new ones.
   - Within these elements, resolve any ambiguity in the SRS and make all additional decisions needed to translate the IR into Maude without further design decisions. Explicitly reflect these decisions in the IR and record them according to the report-writing skill.
8. Using the definitions in static_comp.yaml, write each field in natural language following the sentence templates in dynamic_comp_schema.yaml. Replace every bracketed placeholder with concrete information. Write precondition, guard, and postcondition in sufficient detail to construct the Maude LHS, condition, and RHS, respectively, without inferring missing information from the other fields.
9. For each object, mention only the attributes the rule reads or changes. Attributes not mentioned remain unchanged.
10. Keep the definitions of shared state structures and elements consistent across all rules.
11. Describe postcondition as the resulting state and its values, not as a sequence of processing steps.
12. If a category requested by the template has no applicable elements, explicitly state that there are none rather than inventing elements to fill it.
13. If there is no additional condition beyond the precondition match, leave guard as an empty string instead of filling its sentence template.
14. Use descriptive variable names that clearly indicate what each variable represents. Distinguish individual objects from collections, and distinguish input, existing, and resulting values where relevant. Avoid single-letter variable names and abbreviations.

15. Start each sentence on a new line within each field, keeping all lines of the field value at the same indentation level.
16. Do not introduce separate failure-handling rules unless explicitly required by the SRS. Express prohibitions and restrictions as guards of the corresponding operation rules, so that a prohibited request does not apply the rule. Do not restate or check the constraints declared in static_comp.yaml in any rule.
17. Put one blank line between rules.
18. In postcondition.common, write only how each calculated value used in the rule is calculated.
19. Apply the repair and regression checks below before completing the IR and its report.

20. Read the current report before choosing shared semantics. Treat current static_comp.yaml as the declaration authority; adapt the prior Dynamic IR to any current declaration changes. Use the same formulas, bounds, orderings, and representations in the rules and report, and preserve current Static IR report entries unless a correction is justified by both the SRS and current Static IR. Do not silently replace a static decision with a conflicting dynamic decision.

# Repair and regression checks
1. Before generating, read the listed report_check.yaml, ir_verdict.yaml, maude_result.yaml, and feedback.yaml across all historical versions, even if runs/feedback.yaml is absent. Consult the detailed verdict_a/verdict_b files only as needed to inspect evidence or prior supported behavior. Build an in-memory checklist of distinct issues: source version, affected SRS clause and IR element/field, violated obligation, counterexample or boundary, and current status. Resolve an N id through ir_nl.yaml from the same version; do not assume N ids or IR ids retain their meaning across versions. Merge repeated issues by meaning. Read the associated listed artifacts as needed to verify each issue and any claimed repair.
2. Treat runs/feedback.yaml as the newest observation, not the whole history. Check each issue against the current SRS and candidate; mark it resolved only with concrete evidence in the candidate, retain resolved obligations as regression checks, and distinguish contradictory or superseded feedback using the SRS and artifact contents. A missing finding, an unexecuted check, or an old pass alone does not prove correctness.
3. Use the newest snapshot containing this stage's prior IR and report as the repair baseline. If the newest snapshot is incomplete, use the nearest earlier suitable snapshot while retaining all newer findings. Generate from scratch only if no baseline exists. Repair only affected elements and their dependencies; preserve unaffected text, ids, names, ordering, and decisions. Changes to shared types, units, formulas, or messages require checking every dependent use. Produce the complete normal output, not a patch file.
4. Classify each finding before editing: an SRS contradiction or missing behavior requires an IR repair; a sound IR decision missing from the report requires a precise report repair; disagreement between IR and report requires reconciling both against the SRS. If a reverse statement misreads the IR, retain the correct semantics and clarify the affected IR wording. Do not remove required behavior, invent report justification for an SRS conflict, or redesign unrelated parts to silence a finding.
5. Before finishing, compare the candidate with the baseline and justify each semantic difference by an issue or its dependency. Recheck all accumulated obligations, previously supported behavior, and the affected boundary cases against the complete candidate. Keep the checklist in memory; write only the files listed in Output, and never modify history or feedback.
