---
name: ir-reverse
description: Pipeline step 05 (IR Reverse Interpretation). Use only when the Master assigns step 05.
---

[IR Reverse Interpretation]
Read the IR and explain it in detail in natural language.

# Input
runs/static_comp.yaml, runs/dynamic_comp.yaml

# Output
runs/ir_nl.yaml — in the following format.

statements:
  - id: N-01
    text: <one statement>
  - id: N-02
    text: <one statement>

# Conditions
1. Read only the IR files listed in Input. Do not read or refer to any other file, including the SRS and the report.
2. Preserve every system-relevant detail explicitly present in the IR: actors, conditions, quantifiers, constants, units, formats, inclusive or exclusive bounds, formulas, ordering, message consumption and production, and unchanged state. Do not summarize a precise value into a vague parameter name or infer extra behavior from domain conventions.
3. Write separately assessable statements where practical, keeping each condition attached to the effect it limits. Expand definitions only from the allowed IR and distinguish an explicit restriction from an operation merely absent from these files. Describe system semantics rather than pipeline metadata such as trace ids; do not infer universal prohibitions from missing rules.
4. Before finishing, check both that every statement is entailed by the IR and that the statements cover all explicit IR semantics. If static and dynamic definitions disagree, describe the conflicting definitions accurately instead of silently selecting or repairing one. Do not read history or feedback to shape the interpretation.
