---
name: dynamic-ir
description: Pipeline step 03 (Dynamic IR Generation). Use only when the Master assigns step 03.
---

[Dynamic IR Generation]
Read the SRS and build the Dynamic IR.

Mode:
- Generation: feedback/ir.yaml is empty.
- Revision: otherwise.
Always follow Common Conditions, then the section for the mode.

# Input
inputs/srs_numbered.yaml, runs/static_comp.yaml, schemas/dynamic_comp_schema.yaml, schemas/report_schema.yaml, the report-writing skill, feedback/ir.yaml
- feedback/ir.yaml lists problems found in previous IRs. Each entry has its history version and failed step. Its ids and IR terms refer to that version.
- The latest history/vN/ is the one with the largest N.
- For revision, also: dynamic_comp.yaml and report.yaml in the latest history/vN/.
- To understand a feedback entry, if needed: the history/vN/ folder of its version.

# Output
runs/dynamic_comp.yaml — follows the format of dynamic_comp_schema.yaml.
runs/report.yaml — written according to the report-writing skill.

# Common Conditions
1. Each rule must describe one atomic state transition. Keep all effects that must occur together within the same rule. For the same message, use separate rules when different conditions lead to different effects.
   - When a rule acts on a collection attribute value, describe the change as one value of that attribute, so that the rule's result has a fixed shape.
   - When a rule sends messages to, creates, or removes a runtime-sized set of objects, write one rule that handles one object and fires once per object, and one rule that ends the processing. Declare what they need in static_comp.yaml.
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

# Generation
1. Build the Dynamic IR from the SRS.

# Revision
1. Take dynamic_comp.yaml and the DEC-DYNAMIC entries of report.yaml from the latest history/vN/, and edit them instead of writing new ones.
2. Make sure none of the problems in feedback/ir.yaml appear in the result.
3. Fix only the parts this requires, the parts those fixes affect, and the parts that no longer match runs/static_comp.yaml. Keep everything else unchanged.
