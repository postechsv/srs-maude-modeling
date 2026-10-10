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
3. Whatever cannot be determined from the SRS alone, decide it yourself, explicitly reflect the decision in the IR, including the declarations it requires, and record it according to ir_report.md.
4. Put one blank line between the top-level keys (classes, relationships, enumerations, messages).
5. When a type name alone does not fully specify its meaning or representation, include the missing details in parentheses after the type name in the type field. (ex. Date (YYYY-MM-DD))
6. List the recipient as the first parameter of each message.
7. Values that the system obtains from outside are received as parameters of the request message, and values that the system keeps internally are declared as attributes; nothing is supplied by an execution environment outside the IR.
