[Initial State Generation]
Write the initial state of each ML test case in Maude.

# Input
runs/ml_tc.yaml, runs/maude_types.yaml, docs/Maude3.5.1-manual.pdf

# Output
runs/initial_state.maude — a module importing the module named in maude_types.yaml, defining one initial state per test case named init-<test case id>.

# Conditions
1. Construct the initial state from the scenario, without assuming the expected outcome of the actions being tested.
2. Do not add new behavior.
