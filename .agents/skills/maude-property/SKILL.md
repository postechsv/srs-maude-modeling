---
name: maude-property
description: Pipeline step 13 (Maude Property Generation). Use only when the Master assigns step 13.
---

[Maude Property Generation]
Write the test of each ML test case as Maude search commands.

# Input
runs/ml_tc.yaml, runs/model.maude, runs/initial_state.maude, the maude-reference skill, runs/feedback.yaml (only if it exists)

For repair and regression checking, also read history/vN/ml_tc.yaml, history/vN/model.maude, history/vN/initial_state.maude, history/vN/property.maude, history/vN/maude_result.yaml, history/vN/maude_output.txt, history/vN/feedback.yaml when present in this experiment only. Order existing vN directories by numeric N; history is read-only.

# Output
runs/property.maude — search commands only.

# Conditions
1. For each test case, start from init-<test case id>. Construct the test using the initial state and the requests required by the scenario. Do not duplicate requests already included in the initial state, and preserve any ordering required by the scenario.
2. Before each search command, write a comment with the test case id and whether the test passes when a solution is found or when no solution is found.
3. Do not modify model.maude or initial_state.maude, and do not add new behavior.
4. Use =>* to search for violations of prohibitions or invariants. When checking results after all requests have been processed, explicitly require that no scenario request remains; use =>! only if the scenario also requires a terminal state.
5. Apply the repair and regression checks below before completing this output.

# Repair and regression checks
1. Read the listed historical failures and relevant feedback across all versions, together with runs/feedback.yaml if present. In memory, track distinct errors by affected rule, declaration, or test-case id and cause, retaining fixed errors as regression checks. Verify the actual failing artifact and diagnostic in the same version; do not treat absent later feedback as proof of a fix.
2. Start from the newest snapshot containing this output and its upstream inputs, using the nearest suitable earlier snapshot if needed while retaining newer failures. Compare historical upstream inputs with the current inputs before reuse; current inputs are authoritative. Generate from scratch only if no baseline exists.
3. Repair only the failing fragment and dependencies affected by it or by changed upstream inputs. Preserve unaffected rules, declarations, test cases, names, and behavior. Do not change IR semantics or test expectations to hide an error, and do not copy incompatible historical code over current inputs. Write only this stage's normal complete output.
4. Before finishing, compare against the baseline, recheck accumulated errors and affected dependencies, and verify that previously correct behavior is preserved. Keep the repair checklist in memory; do not modify history, feedback, or upstream files. These checks do not replace step 14's execution or establish a test pass.
