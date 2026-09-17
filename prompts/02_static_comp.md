[Static IR Generation]
Read the SRS and build the Static IR.

# Input
inputs/srs_numbered.yaml, schemas/static_comp_schema.yaml, schemas/ir_report_schema.yaml, prompts/ir_report.md

# Output
runs/static_comp.yaml — follows the format of static_comp_schema.yaml.
runs/ir_report.yaml — written according to ir_report.md.

# Conditions
1. Number ids so that no id is duplicated within the file.
2. In trace, list the SRS sentences that directly support the element.
3. Whatever cannot be determined from the SRS alone, decide it yourself and record it according to ir_report.md.
