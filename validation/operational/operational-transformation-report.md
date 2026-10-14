# Operational Transformation Report

## Scope and method

Input: `labware-configuration-kb.md` only. Output: `labware-configuration-operational.md`. The canonical input was read but not modified. Sections 1–13 retain their technical hierarchy. The original document-governance introduction was replaced by a standalone status, usage, and evidence-classification section; the source register, crosswalk traceability, change history, and maintenance procedures in section 14 were removed. Document identifiers, page citations, and links to unavailable supporting files were removed or replaced with neutral descriptions. Distinct laboratory accounts are retained where they bear on an unresolved configuration decision.

## Automated comparison

- Stable technical rule identifiers in sections 1–13: 123 in input, 123 in output; missing: 0. IDs found only in the removed crosswalk traceability register are intentionally excluded.
- Technical headings (sections 1–13 and Q/G items): 98 in input, 98 in output; changed or missing: 6.
- Technical Markdown tables: 18 in input sections 1–13, 18 in output sections 1–13; one separate evidence-classification table is present in replacement section 0.
- Code fences: 0 fence delimiters in technical input, 0 in output. The removed governance example in section 0 is intentionally omitted.
- Internal Markdown anchor targets unresolved: 0 (none).
- Residual source/file references flagged: 1.
- Exception rows: 7.

## Semantic review and limitations

The transformation preserves the full substantive text of technical sections as its starting point and applies editorial provenance removal and terminology substitutions rather than recreating technical requirements. Exact field names, tables, routine identifiers, parameter values, method examples, conditional branches, open Q01–Q09 and G01–G04 questions, and technical tables were retained in place. No external knowledge or LabWare assumptions were used.

**Complete semantic equivalence has not been demonstrated.** Automated identifier, heading, table, and anchor checks do not establish preservation of every requirement, qualifier, or distinction. Removing source attribution can weaken the ability to distinguish competing laboratory accounts; ambiguous substitutions and remaining prose should be manually reviewed against the canonical input before operational reliance. Runtime behavior, schema, configuration values, and integration outcomes were not independently verified. See the exceptions CSV for detected structural issues; an empty exception file would not establish semantic equivalence.

## Deviations

The source register and source-maintenance section 14, and governance-only parts of section 0, were intentionally removed. The status/evidence classification table in the replacement section 0 is editorial, not a technical configuration table. Source citation columns in technical tables were retained structurally where necessary to avoid changing the surrounding technical table; their remaining content should be reviewed for clarity. No external file is needed to navigate the operational reference.
