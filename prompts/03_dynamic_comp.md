[Dynamic IR Generation]
Read the SRS and build the Dynamic IR.

# Input
inputs/srs_numbered.yaml, runs/static_comp.yaml, schemas/dynamic_comp_schema.yaml, schemas/ir_report_schema.yaml, prompts/ir_report.md

# Output
runs/dynamic_comp.yaml — follows the format of dynamic_comp_schema.yaml.
runs/ir_report.yaml — written according to ir_report.md.

# Conditions
1. Each rule must describe one atomic state transition. Keep all effects that must occur together within the same rule. For the same message, use separate rules when different conditions lead to different effects.
2. Number ids so that no id is duplicated within the file.
3. In trace, list the SRS sentences that directly support the rule.
5. Use names, types, and terminology consistent with static_comp.yaml.
6. Use only the classes, attributes, relationships, enumerations, and messages declared in static_comp.yaml. Do not introduce new ones.
   - Within these elements, resolve any ambiguity in the SRS and make all additional decisions needed to translate the IR into Maude without further design decisions. Explicitly reflect these decisions in the IR and record them according to ir_report.md.
8. Using the definitions in static_comp.yaml, write each field in natural language following the sentence templates in dynamic_comp_schema.yaml. Replace every bracketed placeholder with concrete information. Write precondition, guard, and postcondition in sufficient detail to construct the Maude LHS, condition, and RHS, respectively, without inferring missing information from the other fields.
9. For each object, mention only the attributes the rule reads or changes. Attributes not mentioned remain unchanged.
10. Keep the definitions of shared state structures and elements consistent across all rules.
11. Describe postcondition as the resulting state and its values, not as a sequence of processing steps.
12. If a category requested by the template has no applicable elements, explicitly state that there are none rather than inventing elements to fill it.
13. If there is no additional condition beyond the precondition match, leave guard as an empty string instead of filling its sentence template.
14. Use descriptive variable names that clearly indicate what each variable represents. Distinguish individual objects from collections, and distinguish input, existing, and resulting values where relevant. Avoid single-letter variable names and abbreviations.

15. Start each sentence on a new line within each field, keeping all lines of the field value at the same indentation level.
16. Do not introduce separate failure-handling rules unless explicitly required by the SRS. Express prohibitions and restrictions as preconditions or guards of the corresponding operation rules.
17. Put one blank line between rules.
18. In postcondition.common, write only how each calculated value used in the rule is calculated.
