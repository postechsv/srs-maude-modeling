[Maude Conversion]
Convert the Static IR and the Dynamic IR into a Maude model.

# Input
runs/static_comp.yaml, runs/dynamic_comp.yaml, schemas/model_report_schema.yaml

# Output
runs/model.maude — a Maude object-oriented module.
runs/model_report.yaml — follows the format of model_report_schema.yaml.

# Conditions
1. Convert each rule so that event and existing become the left-hand side, guard becomes the condition, and postcondition becomes the right-hand side. Use rl when guard is empty, and crl otherwise.
2. Use the rule id as the rule label.
3. Whatever the IR does not determine, decide it yourself and record it in model_report.yaml.
