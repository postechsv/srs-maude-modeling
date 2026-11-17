---
name: maude-model
description: Pipeline step 11 (Maude Conversion). Use only when the Master assigns step 11.
---

[Maude Conversion]
Convert the Static IR and the Dynamic IR into a Maude model.

Mode:
- Revision: the last entry of feedback/maude_model.yaml has the same version as the latest history/vN/.
- Generation: otherwise.
Check only that version to choose the mode. Always follow Common Conditions, then the section for the mode.

# Input
runs/static_comp.yaml, runs/dynamic_comp.yaml, docs/maude_oo_rule_form.md, the maude-reference skill, feedback/maude_model.yaml
- feedback/maude_model.yaml: problems found in previous Maude models. Each entry has its history version and failed step. Its ids and IR terms refer to that version.
- The latest history/vN/ is the one with the largest N.
For revision, also:
- model.maude in the latest history/vN/.
- To understand a feedback entry, if needed: the history/vN/ folder of its version.

# Output
runs/model.maude — a Maude object-oriented module.

# Common Conditions
1. Read docs/maude_oo_rule_form.md first and follow it with priority. For any other syntax you need, refer to the maude-reference skill.
2. By default, convert each rule so that precondition.msgs and precondition.objects become the left-hand side, guard becomes the condition, and postcondition.msgs, postcondition.objects, and postcondition.common become the right-hand side. Use rl when guard is empty, and crl otherwise. When Additional Rule 5 in docs/maude_oo_rule_form.md applies, split the rule as it describes.
4. By default, use the rule id as the rule label. For a rule split by Additional Rule 5, use the labels it describes.
5. Keep the names used in the IR, but write them following the Maude naming conventions below. Change only the case of letters; do not otherwise rename or abbreviate any name.
   - Module names: all capital letters, with hyphens between words (for example, FOO-BAR).
   - Sort and class names: the first letter of each word in upper case, with words joined without spaces or hyphens (for example, FooBar).
   - View names: the first letter in upper case (for example, Foo).
   - Parameter names: a single capital letter (for example, X).
   - Variable names: the first letter in upper case (for example, fooBar becomes FooBar).
   - Operator names, including message names: the first letter in lower case, with each later word starting with an upper-case letter (for example, FooBar becomes fooBar).
   - The first argument of every message is the Oid of the object that receives it.
6. Represent the messages declared in static_comp.yaml as Maude messages of sort Msg.
7. Do not define an initial state. It is written separately.
8. At the top level of each side of a rule, write only objects and messages; do not wrap them in a function call. Inside attribute values and conditions, functions may be used.
10. Write the constraints declared in static_comp.yaml as one operation valid : Configuration -> Bool defined only by equations, so that valid of a configuration is false exactly when the configuration violates one of the constraints. Do not use valid in any rule.

# Generation
1. Build the Maude model from the current IR. Do not use feedback/maude_model.yaml.

# Revision
1. Take model.maude from the latest history/vN/, and edit it instead of writing a new one.
2. Make sure none of the problems in feedback/maude_model.yaml appear in the result.
3. Fix only the parts this requires and the parts those fixes affect. Keep everything else unchanged.
