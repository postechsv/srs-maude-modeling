[Maude Conversion]
Convert the Static IR and the Dynamic IR into a Maude model.

# Input
runs/static_comp.yaml, runs/dynamic_comp.yaml, docs/maude_oo_rule_form.md, docs/Maude3.5.1-manual.pdf, runs/feedback.yaml (only if it exists)

# Output
runs/model.maude — a Maude object-oriented module.

# Conditions
1. Read docs/maude_oo_rule_form.md first and follow it with priority. For any other syntax you need, refer to docs/Maude3.5.1-manual.pdf.
2. Convert each rule so that precondition.msgs and precondition.objects become the left-hand side, guard becomes the condition, and postcondition.msgs, postcondition.objects, and postcondition.common become the right-hand side. Use rl when guard is empty, and crl otherwise.
4. Use the rule id as the rule label.
5. Preserve the variable names used in the IR exactly. Do not rename or abbreviate them.
6. Represent the messages declared in static_comp.yaml as Maude messages of sort Msg.
7. Do not define an initial state. It is written separately.
8. The left-hand side and the right-hand side of each rule consist only of objects and messages written out explicitly; do not wrap them in function calls or produce them with function calls. Functions may appear only inside attribute values and conditions.
10. Write the constraints declared in static_comp.yaml as one operation valid : Configuration -> Bool defined only by equations, so that valid of a configuration is false exactly when the configuration violates one of the constraints. Do not use valid in any rule.
11. If runs/feedback.yaml exists, it describes problems found in a previous Maude model. Build the Maude model so that none of these problems remain.
