[Maude Property Generation]
Write the test of each ML test case as Maude search commands.

# Input
runs/ml_tc.yaml, runs/model.maude, runs/initial_state.maude, docs/Maude3.5.1-manual.pdf, runs/feedback.yaml (only if it exists)

# Output
runs/property.maude — search commands only.

# Conditions
1. For each test case, start from init-<test case id>. Construct the test using the initial state and the requests required by the scenario. Do not duplicate requests already included in the initial state, and preserve any ordering required by the scenario.
2. Before each search command, write a comment with the test case id and whether the test passes when a solution is found or when no solution is found.
3. Do not modify model.maude or initial_state.maude, and do not add new behavior.
4. Use =>* to search for violations of prohibitions or invariants. When checking results after all requests have been processed, explicitly require that no scenario request remains; use =>! only if the scenario also requires a terminal state.
5. If runs/feedback.yaml exists, it describes problems found in a previous property. Build the property so that none of these problems remain.
