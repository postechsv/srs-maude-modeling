---
name: maude-model
description: Pipeline step 11 (Maude Conversion). Use only when the Master assigns step 11.
---

[Maude Conversion]
Convert the Static IR and the Dynamic IR into a Maude model.

# Input
runs/static_comp.yaml, runs/dynamic_comp.yaml, docs/maude_oo_rule_form.md, the maude-reference skill, runs/feedback.yaml (only if it exists)

For repair and regression checking, also read history/vN/static_comp.yaml, history/vN/dynamic_comp.yaml, history/vN/model.maude, history/vN/maude_result.yaml, history/vN/maude_output.txt, history/vN/feedback.yaml when present in this experiment only. Order existing vN directories by numeric N; history is read-only.

# Output
runs/model.maude — a Maude object-oriented module.

# Conditions
1. Read docs/maude_oo_rule_form.md first and follow it with priority. For any other syntax you need, refer to the maude-reference skill.
2. Convert each rule so that precondition.msgs and precondition.objects become the left-hand side, guard becomes the condition, and postcondition.msgs, postcondition.objects, and postcondition.common become the right-hand side. Use rl when guard is empty, and crl otherwise.
4. Use the rule id as the rule label.
5. Preserve the variable names used in the IR exactly. Do not rename or abbreviate them.
6. Represent the messages declared in static_comp.yaml as Maude messages of sort Msg.
7. Do not define an initial state. It is written separately.
8. The left-hand side and the right-hand side of each rule consist only of objects and messages written out explicitly; do not wrap them in function calls or produce them with function calls. Functions may appear only inside attribute values and conditions.
10. Write the constraints declared in static_comp.yaml as one operation valid : Configuration -> Bool defined only by equations, so that valid of a configuration is false exactly when the configuration violates one of the constraints. Do not use valid in any rule.
11. Apply the repair and regression checks below before completing this output.

# Repair and regression checks
1. Read the listed historical failures and relevant feedback across all versions, together with runs/feedback.yaml if present. In memory, track distinct errors by affected rule, declaration, or test-case id and cause, retaining fixed errors as regression checks. Verify the actual failing artifact and diagnostic in the same version; do not treat absent later feedback as proof of a fix.
2. Start from the newest snapshot containing this output and its upstream inputs, using the nearest suitable earlier snapshot if needed while retaining newer failures. Compare historical upstream inputs with the current inputs before reuse; current inputs are authoritative. Generate from scratch only if no baseline exists.
3. Repair only the failing fragment and dependencies affected by it or by changed upstream inputs. Preserve unaffected rules, declarations, test cases, names, and behavior. Do not change IR semantics or test expectations to hide an error, and do not copy incompatible historical code over current inputs. Write only this stage's normal complete output.
4. Before finishing, compare against the baseline, recheck accumulated errors and affected dependencies, and verify that previously correct behavior is preserved. Keep the repair checklist in memory; do not modify history, feedback, or upstream files. These checks do not replace step 14's execution or establish a test pass.
