[Maude Conversion]
Convert the Static IR and the Dynamic IR into a Maude model.

# Input
runs/static_comp.yaml, runs/dynamic_comp.yaml, docs/maude_oo_rule_form.md, docs/Maude3.5.1-manual.pdf

# Output
runs/model.maude — a Maude object-oriented module.

# Conditions
1. Read docs/maude_oo_rule_form.md first and follow it with priority. For any other syntax you need, refer to docs/Maude3.5.1-manual.pdf.
2. Convert each rule so that precondition.msgs and precondition.objects become the left-hand side, guard becomes the condition, and postcondition.msgs, postcondition.objects, and postcondition.common become the right-hand side. Use rl when guard is empty, and crl otherwise.
4. Use the rule id as the rule label.
5. Preserve the variable names used in the IR exactly. Do not rename or abbreviate them.
6. Represent the messages declared in static_comp.yaml as Maude messages of sort Msg.
