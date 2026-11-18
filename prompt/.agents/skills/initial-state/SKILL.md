---
name: initial-state
description: Pipeline step 12 (Initial State Generation). Use only when the Master assigns step 12.
---

[Initial State Generation]
Write the initial state of each ML test case in Maude.

Mode:
- Revision: the last entry of feedback/initial_state.yaml has the same version as the latest history/vN/.
- Generation: otherwise.
Check only that version to choose the mode. Always follow Common Conditions, then the section for the mode.

# Input
runs/ml_tc.yaml, runs/model.maude, the maude-reference skill, feedback/initial_state.yaml
- feedback/initial_state.yaml: problems found in previous initial states. Each entry has its history version and failed step. Its ids and IR terms refer to that version.
- The latest history/vN/ is the one with the largest N.
For revision, also:
- initial_state.maude in the latest history/vN/.
- To understand a feedback entry, if needed: the history/vN/ folder of its version.

# Output
runs/initial_state.maude — a module importing the module in model.maude, defining one initial state per test case named init-<test case id>.

# Common Conditions
1. Construct the initial state from the scenario, without assuming the expected outcome of the actions being tested.
2. Do not modify the model or add new behavior.

# Generation
1. Build the initial states from the current ML TC and model.maude. Do not use feedback/initial_state.yaml.

# Revision
1. Take initial_state.maude from the latest history/vN/, and edit it instead of writing a new one.
2. Make sure none of the problems in feedback/initial_state.yaml appear in the result.
3. Fix only the parts this requires and the parts those fixes affect. Keep everything else unchanged.
