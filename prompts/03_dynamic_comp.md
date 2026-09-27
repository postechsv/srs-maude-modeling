[Dynamic IR Generation]
Read the SRS and build the Dynamic IR.

# Input
inputs/srs_numbered.yaml, runs/static_comp.yaml, schemas/dynamic_comp_schema.yaml, schemas/ir_report_schema.yaml, prompts/ir_report.md

# Output
runs/dynamic_comp.yaml — follows the format of dynamic_comp_schema.yaml.
runs/ir_report.yaml — written according to ir_report.md.

# Conditions
1. Each rule must describe one atomic state transition. Keep all effects that must occur together within the same rule. For the same event, use separate rules when different conditions lead to different effects.
2. Number ids so that no id is duplicated within the file.
3. In trace, list the SRS sentences that directly support the rule.
4. Resolve any ambiguity in the SRS and make all additional decisions needed to translate the IR into Maude without further design decisions. Explicitly reflect these decisions in the IR and record them according to ir_report.md.
5. Use names, types, and terminology consistent with static_comp.yaml.
6. Include rules required by the SRS even if they require elements not defined in static_comp.yaml.
7. Record any elements required by these rules but missing from static_comp.yaml in ir_report.yaml. Use the format "RULE-M-01" for the id of each rule requiring elements absent from static_comp.yaml.
8. Using the definitions in static_comp.yaml, write each field in natural language. In precondition, explicitly specify the event and its inputs, and the objects to be matched, including their classes, attribute values or variable bindings, and relationships. In guard, specify the conditions on those values. In postcondition, explicitly specify the resulting state, including what is created, changed, deleted, or preserved. When translating the IR into Maude, it must be possible to write the LHS, condition, and RHS from precondition, guard, and postcondition, respectively, without inferring missing information from the other fields.