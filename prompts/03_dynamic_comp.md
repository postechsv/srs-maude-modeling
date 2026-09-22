[Dynamic IR Generation]
Read the SRS and build the Dynamic IR.

# Input
inputs/srs_numbered.yaml, runs/static_comp.yaml, schemas/dynamic_comp_schema.yaml, schemas/ir_report_schema.yaml, prompts/ir_report.md

# Output
runs/dynamic_comp.yaml — follows the format of dynamic_comp_schema.yaml.
runs/ir_report.yaml — written according to ir_report.md.

# Conditions
1. Split rules into units that cannot be divided further (atomic).
2. Number ids so that no id is duplicated within the file.
3. In trace, list the SRS sentences that directly support the rule.
4. Whatever cannot be determined from the SRS alone, decide it yourself and record it according to ir_report.md.
5. Use names, types, and terminology consistent with static_comp.yaml.
6. Include rules required by the SRS even if they require elements not defined in static_comp.yaml.
7. Record any elements required by these rules but missing from static_comp.yaml in ir_report.yaml.