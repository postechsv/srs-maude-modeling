---
name: maude-verdict
description: Pipeline step 15 (Maude Test Verdict). Use only when the Master assigns step 15.
---

[Maude Test Verdict]
Decide the result of each test case from the Maude output.

# Input
runs/model.maude, runs/initial_state.maude, runs/property.maude, runs/maude_output.txt

# Output
runs/maude_result.yaml — in the following format. Write failures only when pass is false.

pass: <true | false>
failures:
  - id: <id of the high-level test case>
    kind: <violation | inconclusive | error>
    source: <model.maude | initial_state.maude | property.maude | model content>   # only for error
    reason: <what the test expected, and what happened instead>

# Conditions
1. Decide each test by the pass criterion written in the comment before its search in property.maude.
2. For a test that passes when no solution is found, require exhaustive completion of the search without a depth limit. A timeout, interruption, or depth-limited search with no solution does not establish a pass.
3. List each test that does not pass in failures with one of the following kinds:
   - violation: the search was executed, and its result contradicts the pass criterion.
   - inconclusive: the search was executed but did not complete, for example because of a timeout, interruption, or depth limit.
   - error: Maude did not execute the search, for example because of a parse error, or property.maude has no search for the test. In source, write the file that caused it. If property.maude has no search because the model has nothing that shows the expected result, write model content.
