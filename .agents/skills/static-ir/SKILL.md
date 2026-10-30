---
name: static-ir
description: Pipeline step 02 (Static IR Generation). Use only when the Master assigns step 02.
---

[Static IR Generation]
Read the SRS and build the Static IR.

Mode:
- Generation: feedback/ir.yaml is empty.
- Revision: otherwise.
Always follow Common Conditions, then the section for the mode.

# Input
inputs/srs_numbered.yaml, schemas/static_comp_schema.yaml, schemas/report_schema.yaml, the report-writing skill, feedback/ir.yaml
- feedback/ir.yaml lists problems found in previous IRs. Each entry has its history version and failed step. Its ids and IR terms refer to that version.
- The latest history/vN/ is the one with the largest N.
- For revision, also: static_comp.yaml and report.yaml in the latest history/vN/.
- To understand a feedback entry, if needed: the history/vN/ folder of its version.

# Output
runs/static_comp.yaml — follows the format of static_comp_schema.yaml.
runs/report.yaml — written according to the report-writing skill.

# Common Conditions
1. Number ids so that no id is duplicated within the file.
2. In trace, list the SRS sentences that directly support the element.
3. Whatever cannot be determined from the SRS alone, decide it yourself, explicitly reflect the decision in the IR, including the declarations it requires, and record it according to the report-writing skill.
4. Put one blank line between the top-level keys (classes, relationships, enumerations, messages, constraints).
5. When a type name alone does not fully specify its meaning or representation, include the missing details in parentheses after the type name in the type field. (ex. Date (YYYY-MM-DD))
6. List the recipient as the first parameter of each message.
7. Values that the system needs to perform the behaviors in the SRS are declared: values obtained from outside are received as parameters of the request message, and values kept internally are declared as attributes; nothing is supplied by an execution environment outside the IR.
8. Do not declare results or records that the SRS does not require.
9. Distinguish the following two kinds of conditions, and declare only constraints in constraints.
   - constraint: the uniqueness of a value that identifies an object among the objects of its class, which every state of the system must satisfy. Write it as a condition on the state, not as a check performed by an operation.
   - rule condition: any other condition, such as a limit on a value supplied by a request or a restriction on how objects may be related to each other. It is checked only when a request is accepted and is written in the Dynamic IR, not in constraints.

# Generation
1. Build the Static IR from the SRS.

# Revision
1. Take static_comp.yaml and the DEC-STATIC entries of report.yaml from the latest history/vN/, and edit them instead of writing new ones.
2. Fix only the parts the feedback requires and the parts those fixes affect. Keep everything else unchanged.
3. Make sure none of the feedback problems remain.
