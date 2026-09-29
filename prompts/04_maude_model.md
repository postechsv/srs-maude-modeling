[Maude Conversion]
Convert the Static IR and the Dynamic IR into a Maude model.

# Input
runs/static_comp.yaml, runs/dynamic_comp.yaml, docs/Maude3.5.1-manual.pdf

# Output
runs/model.maude — a Maude object-oriented module.

# Conditions
1. Write the model in Maude 3.5.1 syntax as described in docs/Maude3.5.1-manual.pdf. Read only the parts you need.
2. Convert each rule so that event and existing become the left-hand side, guard becomes the condition, and postcondition becomes the right-hand side. Use rl when guard is empty, and crl otherwise.
3. Use the rule id as the rule label.
4. Preserve the variable names used in the IR exactly. Do not rename or abbreviate them.
