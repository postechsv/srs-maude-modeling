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
