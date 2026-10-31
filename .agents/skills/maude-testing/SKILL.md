---
name: maude-testing
description: Pipeline step 14 (Maude Testing). Use only when the Master assigns step 14.
---

[Maude Testing]
Run the Maude tests.

# Input
runs/model.maude, runs/initial_state.maude, runs/property.maude

# Output
runs/maude_output.txt — the output of the command below.

# Conditions
1. Run the following command inside runs/:
   maude -no-banner model.maude initial_state.maude property.maude < /dev/null > maude_output.txt 2>&1
