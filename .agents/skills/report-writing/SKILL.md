---
name: report-writing
description: Report Writing rules. Use only when a pipeline step's Input lists this skill.
---

[Report Writing]
Record in report.yaml what you decided yourself while building the IR because it could not be determined from the SRS alone.

# Output
runs/report.yaml — follows the format of report_schema.yaml.

# Conditions
1. Write one entry per decision.
2. Number ids according to the IR being built, such as DEC-STATIC-01 or DEC-DYNAMIC-01.
3. For a decision not reflected in the IR, leave target as an empty list.
4. If a current report exists, read it before updating it. On revision, retain applicable baseline decisions and their ids; update or remove superseded decisions rather than appending contradictory replacements. Step 02 writes the static decisions only; step 03 preserves the current static entries and merges the revised dynamic decisions, checking their consistency with the current IR.
5. Make each decision self-contained for a reviewer who cannot read the IR. State the exact chosen values, units, bounds and inclusivity, representations, formulas and constants, ordering, defaults, and behavioral scope wherever the SRS leaves them open. A name or a reference such as "the declared value" is not a substitute for its definition. Set target to all current IR ids implementing the decision; check that the IR and report state the same choice.
6. Before finishing, check every added IR detail against the SRS or an explicit report decision, and check that every retained decision is actually reflected in its targets. If feedback identifies missing report evidence for an intentional, SRS-consistent IR detail, complete the report without redesigning that detail. If the IR contradicts the SRS, repair the IR and its affected decisions; do not justify the contradiction by adding report text.
