---
name: static-ir
description: Pipeline step 02 (Static IR Generation). Use only when the Master assigns step 02.
---

[Static IR Generation]
Read the SRS and build the Static IR.

# Input
inputs/srs_numbered.yaml, schemas/static_comp_schema.yaml, schemas/report_schema.yaml, the report-writing skill, runs/feedback.yaml (only if it exists)
For revision context, also read existing runs/report.yaml and this experiment's history/vN/ files (read-only): static_comp.yaml, dynamic_comp.yaml, report.yaml, ir_nl.yaml, report_check.yaml, ir_verdict*.yaml, maude_result.yaml.

# Output
runs/static_comp.yaml — follows the format of static_comp_schema.yaml.
runs/report.yaml — written according to the report-writing skill.

# Conditions
1. Number ids so that no id is duplicated within the file.
2. In trace, list the SRS sentences that directly support the element.
3. Whatever cannot be determined from the SRS alone, decide it yourself, explicitly reflect the decision in the IR, including the declarations it requires, and record it according to the report-writing skill.
4. Put one blank line between the top-level keys (classes, relationships, enumerations, messages, constraints).
5. When a type name alone does not fully specify its meaning or representation, include the missing details in parentheses after the type name in the type field. (ex. Date (YYYY-MM-DD))
6. List the recipient as the first parameter of each message.
7. Values that the system needs to perform the behaviors in the SRS are declared: values obtained from outside are received as parameters of the request message, and values kept internally are declared as attributes; nothing is supplied by an execution environment outside the IR.
8. Do not declare results or records that the SRS does not require.
9. Use runs/feedback.yaml and the available checks, verdicts, and Maude failures from every history/vN/ to form an in-memory checklist of distinct problems, including previously fixed ones. Resolve each finding against its own version's artifacts; N ids and element ids may refer to different content across versions. Reconcile conflicting feedback against the SRS rather than blindly combining fixes.
   - If an archived static_comp.yaml and report.yaml pair exists, use the latest such snapshot as the revision baseline, ordering N numerically. Repair only the implicated declarations, decisions, and their dependencies; preserve unaffected content and ids. Use archived Dynamic IR to identify declarations needed by affected rules. Write the complete revised outputs to runs/ even when RUN.md has deleted the previous files; do not modify history or create a checklist file.
   - Before finishing, check the revised IR and report against every applicable checklist item, including earlier fixes, and confirm that unaffected SRS requirements remain satisfied. A previous pass is evidence to preserve, not permission to retain a discovered error.
10. Distinguish the following two kinds of conditions, and declare only constraints in constraints.
   - constraint: the uniqueness of a value that identifies an object among the objects of its class, which every state of the system must satisfy. Write it as a condition on the state, not as a check performed by an operation.
   - rule condition: any other condition, such as a limit on a value supplied by a request or a restriction on how objects may be related to each other. It is checked only when a request is accepted and is written in the Dynamic IR, not in constraints.
