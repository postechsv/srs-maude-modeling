---
name: initial-state
description: Pipeline step 12 (Initial State Generation). Use only when the Master assigns step 12.
---

[Initial State Generation]
Write the initial state of each ML test case in Maude.

# Input
runs/ml_tc.yaml, runs/model.maude, the maude-reference skill, runs/feedback.yaml (only if it exists)

For repair and regression checking, also read history/vN/ml_tc.yaml, history/vN/model.maude, history/vN/initial_state.maude, history/vN/maude_result.yaml, history/vN/maude_output.txt, history/vN/feedback.yaml when present in this experiment only. Order existing vN directories by numeric N; history is read-only.

# Output
runs/initial_state.maude — a module importing the module in model.maude, defining one initial state per test case named init-<test case id>.

# Conditions
1. Construct the initial state from the scenario, without assuming the expected outcome of the actions being tested.
2. Do not modify the model or add new behavior.
3. Apply the repair and regression checks below before completing this output.

# Repair and regression checks
1. Read the listed historical failures and relevant feedback across all versions, together with runs/feedback.yaml if present. In memory, track distinct errors by affected rule, declaration, or test-case id and cause, retaining fixed errors as regression checks. Verify the actual failing artifact and diagnostic in the same version; do not treat absent later feedback as proof of a fix.
2. Start from the newest snapshot containing this output and its upstream inputs, using the nearest suitable earlier snapshot if needed while retaining newer failures. Compare historical upstream inputs with the current inputs before reuse; current inputs are authoritative. Generate from scratch only if no baseline exists.
3. Repair only the failing fragment and dependencies affected by it or by changed upstream inputs. Preserve unaffected rules, declarations, test cases, names, and behavior. Do not change IR semantics or test expectations to hide an error, and do not copy incompatible historical code over current inputs. Write only this stage's normal complete output.
4. Before finishing, compare against the baseline, recheck accumulated errors and affected dependencies, and verify that previously correct behavior is preserved. Keep the repair checklist in memory; do not modify history, feedback, or upstream files. These checks do not replace step 14's execution or establish a test pass.
