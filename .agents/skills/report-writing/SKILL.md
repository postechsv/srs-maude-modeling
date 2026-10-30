---
name: report-writing
description: Report Writing rules. Use only when a pipeline step's Input lists this skill.
---

[Report Writing]
Record in report.yaml what you decided yourself while building the IR because it could not be determined from the SRS alone.

# Input
The invoking stage's listed SRS, schemas, current IR, and permitted history files; runs/report.yaml when it exists. Inspect the invoking stage's candidate IR as it is built.

# Output
runs/report.yaml — follows the format of report_schema.yaml.

# Conditions
1. Write one entry per decision.
2. Number ids according to the IR being built, such as DEC-STATIC-01 or DEC-DYNAMIC-01.
3. For a decision not reflected in the IR, leave target as an empty list.
4. Read the existing report before updating it. Step 02 reconstructs the current static decisions from its repair baseline; step 03 preserves those entries and merges the current dynamic decisions. Update a changed decision in place, preserve unchanged decision ids and text, and allocate unused ids only for new decisions. Remove obsolete or contradictory entries rather than appending a competing decision; the report describes the current IR, not a revision log.
5. Make each decision self-contained for a reviewer who cannot read the IR. State the exact chosen values and their units, inclusive or exclusive bounds, defaults, formats, formulas and constants, ordering and tie-breaking rules, message effects, and scope wherever applicable. Phrases such as "use the declared constant" or "as defined in RULE-XX" and target ids are not substitutes for the actual decision. A value uniquely derivable from stated facts may be explained by that derivation; an arbitrary choice must be explicit.
6. Audit each concrete IR detail not entailed by the SRS against a report entry at the same precision, including details inside type descriptions, guards, and calculations. Set target to the actual affected IR ids. Conversely, check that each targeted decision is implemented consistently in the candidate IR. A target: [] entry must explicitly say it is not implemented and must not promise behavior absent from the IR.
7. Resolve disagreements among static decisions, dynamic decisions, and their implementations against the SRS before finishing. If the IR already implements a permissible choice, fill the report gap without changing that choice. Documenting a decision does not make an SRS contradiction permissible. Keep all explanations within the existing issue and decision fields.
