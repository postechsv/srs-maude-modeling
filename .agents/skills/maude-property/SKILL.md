---
name: maude-property
description: Pipeline step 13 (Maude Property Generation). Use only when the Master assigns step 13.
---

[Maude Property Generation]
Write the test of each ML test case as Maude search commands.

Mode:
- Revision: the last entry of feedback/property.yaml has the same version as the latest history/vN/.
- Generation: otherwise.
Check only that version to choose the mode. Always follow Common Conditions, then the section for the mode.

# Input
runs/ml_tc.yaml, runs/model.maude, runs/initial_state.maude, the maude-reference skill
For revision, also:
- feedback/property.yaml: problems found in previous properties. Each entry has its history version and failed step. Its ids and IR terms refer to that version.
- property.maude in the latest history/vN/.
- To understand a feedback entry, if needed: the history/vN/ folder of its version.

# Output
runs/property.maude — search commands only.

# Common Conditions
1. For each test case, start from init-<test case id>. Construct the test using the initial state and the requests required by the scenario. Do not duplicate requests already included in the initial state, and preserve any ordering required by the scenario.
2. Before each search command, write a comment with the test case id and whether the test passes when a solution is found or when no solution is found.
3. Do not modify model.maude or initial_state.maude, and do not add new behavior.
4. Use =>* to search for violations of prohibitions or invariants. When checking results after all requests have been processed, explicitly require that no scenario request remains; use =>! only if the scenario also requires a terminal state.

# Generation
1. Build the search commands from the current ML TC, model.maude, and initial_state.maude. Do not use feedback/property.yaml.

# Revision
1. Take property.maude from the latest history/vN/, and edit it instead of writing a new one.
2. Make sure none of the problems in feedback/property.yaml appear in the result.
3. Fix only the parts this requires and the parts those fixes affect. Keep everything else unchanged.
