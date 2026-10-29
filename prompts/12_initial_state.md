[Initial State Generation]
Write the initial state of each ML test case in Maude.

# Input
runs/ml_tc.yaml, runs/model.maude, docs/Maude3.5.1-manual.pdf, runs/feedback.yaml (only if it exists)

# Output
runs/initial_state.maude — a module importing the module in model.maude, defining one initial state per test case named init-<test case id>.

# Conditions
1. Construct the initial state from the scenario, without assuming the expected outcome of the actions being tested.
2. Do not modify the model or add new behavior.
3. If runs/feedback.yaml exists, it describes problems found in a previous initial state. Build the initial state so that none of these problems remain.
