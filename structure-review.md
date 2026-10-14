# Initial Source and Knowledge-Base Structure Review

Date: 2026-10-14  
Sources: S01 and S02 (see `source/manifest.md`)  
Status: assessment, not consolidated requirements or verified configuration guidance.

## Source assessment

**S01** is a compact, prescriptive three-page set of batch and analysis configuration notes. It distinguishes file-parsing-only setup from general batch setup, and uses ICP_OES / OSHA_1023 as its primary implementation example. Topics include analysis attributes, batch links, variations, component settings, batch results, instrument identification, file-path constants and parser naming/return arrays. It should not be treated as a complete guide to every analysis or import mode.

**S02** is a broader 21-page initial draft addressing OIS requests, OCD-based test assignment, sample receiving, analysis and batch configuration, limits, aliquoting, file parsing, QC, Empower and OIS reporting. Several workflow and analysis-specific headings have no substantive procedures. Its documented behavior is laboratory implementation guidance, not independently established vendor behavior.

## Recommended canonical outline

The user's proposed sections 0–14 remain suitable with modest scope refinements:

| Section | Canonical ownership |
|---|---|
| 0. Purpose, Usage and Evidence Rules | LLM reading rules, classifications, applicability, source traceability |
| 1. Configuration Architecture and Dependencies | Object relationships, exact identifiers, naming conventions, dependency map |
| 2. OIS Integration, Project and Sample Setup | File intake, OCD mapping, test assignment, receiving |
| 3. Analysis Configuration | Analysis/method/variation/component setup, attributes, aliquoting |
| 4. Batch Configuration | Batch templates, QC, stocks, required results, batch objects |
| 5. Instrument Configuration | Instrument groups and parameters; cross-reference parsing/Empower |
| 6. Data Import and Parsing | Paths, constants, scripts, parser contracts, validation and file lifecycle |
| 7. Calculations, Limits and Reporting | Limits, reporting, recovery, OIS result outputs |
| 8. Empower Integration | Empower custom fields, interface mapping, project matching |
| 9. Operational Workflows | Receiving, testing, review, release and closure |
| 10. End-to-End Analysis Implementation Procedure | Ordered checklist linking to canonical technical topics |
| 11. Established Configuration Examples | Evidence-scoped, method-specific examples and exceptions |
| 12. Troubleshooting and Verified Behaviors | Reproducible tests, observed errors, confirmed fixes |
| 13. Open Questions and Undocumented Areas | Conflicts, missing procedures, verification backlog |
| 14. Source Register and Change History | Source identifiers, provenance, material revisions |

Keep one file for the future canonical reference. The source manifest and this review are supporting project records, not alternative technical references.

## Initial topic mapping (not exhaustive crosswalk)

| Topic | S01 | S02 | Proposed owner |
|---|---|---|---|
| Batch link, analysis/variation/method mapping | pp. 1–2 | pp. 9–11, 14–16 | §§1,3; detailed rule in §3 |
| Component flags, reportable alias | pp. 1–2 | pp. 11–13 | §3; reporting in §7 |
| Batch template, QC, stocks, instrument results | p. 2 | pp. 13–16 | §4 |
| Instrument parameters | p. 2 | pp. 9, 15 | §5 |
| Data directory, parsing contract | p. 3 | pp. 8, 16–18 | §6 |
| Analysis limits, calculation sequencing | pp. 1–2 | pp. 10, 12–13 | §7 |
| OCD-based assignment, aliquoting | — | pp. 6–7, 11 | §§2,3 |
| Empower project fields and mapping | — | pp. 19–21 | §8 |
| Sample receiving, review, closure | — | pp. 8–9 (largely headings) | §9 |
| ICP_OES and other analysis-specific notes | ICP_OES examples throughout | p. 21 (headings only) | §11 |

## Apparent overlaps, tensions and open questions

These are investigation candidates, **not confirmed defects**:

- **Q01 — Instrument link field.** S01 p. 2 uses batch result alias matching instrument group. S02 p. 9 mentions Batch Result `X_STOCK` for an instrument, while S02 p. 14 uses `BATCH_COMPONENT.ALIAS_NAME = INSTRUMENT.INST_GROUP`. Verify whether these are distinct relationships or an inconsistent instruction.
- **Q02 — Required vs optional fields.** S01 pp. 1–2 discusses base component `Optional`, variation `Displayed`, and batch `Optional = F`. S02 pp. 11, 14 uses variation optionality and batch `Required = T`. Determine field-level semantics and applicable screens.
- **Q03 — Variation naming and method pairing.** S01 p. 1 gives `_QC` / `_QCSM` examples; S02 pp. 10–11 includes media/aliquot suffixes and S02 pp. 15–16 describes related-method exceptions. Avoid assuming every variation type is mandatory or method name always matches variation prefix.
- **Q04 — Parser dispatch.** S01 pp. 2–3 describes LabStation scripts or instrument-specific subroutines and `PARAMETER_1_VALUE`; S02 p. 9 treats Parameter 1 more broadly. Actual dispatch logic needs code evidence.
- **Q05 — Parser return interface.** S01 p. 3 specifies `resultDataArr` and conditional reorder variables; S02 p. 17 also describes `stopParse`, `validationStr`, `wrongBatch` and supplied context variables. Establish which outputs are always required versus conditional.
- **Q06 — Calculation applicability.** Both sources require `AutoCalculate = F` for final calculations in file-import batches; S02 p. 12 permits `T` for manual entry or Empower. Keep mode-specific applicability.
- **Q07 — Reporting flags.** S01 pp. 1–2 and S02 pp. 11–13 discuss base and variation reportability, review and outbound OIS output. Verify which level governs each outcome.
- **Q08 — File path constant granularity.** S01 p. 3 states current analysis-level behavior and suggests different granularity may be possible; do not describe alternatives as supported without evidence.
- **Q09 — Manual batch result checks.** S02 p. 16 says manual entry does not validate Batch Results at batch level; do not extrapolate the file-import preconditions to manual workflows.

## Missing or incomplete material

S02 headings without sufficient implementation detail include XML schema and staging/investigation logic, project/sample templates, receiving/seal/label workflows, batch review, project closure/OIS response generation, and analysis-specific sections for ICP_OES, SILICA_XRD, GRAV_PREWEIGH and GRAV_POSTWEIGH. File transfer script operations, parser failure/retry/rollback, calculation orchestration, and Empower error handling also need more evidence.

## Evidence priorities

1. `MNU_BM_SLTC_DATA_PARSE` and a working `BM_SLTC_DATA_PARSE_ICP_OES` implementation, including validation-only and commit behavior.
2. `CALC_ICP_FINAL` and related limit-format calculations.
3. Exported ANALYSIS, variation, method, batch template, component and instrument records for one known-working analysis.
4. Test evidence for file import, Empower, manual entry, review and OIS reporting.
5. Later, register the separately converted LabWare 8 technical manual as an independent vendor source; do not assume its contents now.

## Next deliverable

Create a **full statement-level crosswalk**, not just the topic map above. Recommended columns: `id`, `topic`, `source_statement`, `classification`, `applicability`, `source_id`, `source_location`, `canonical_section`, `implementation_evidence`, `verification_status`, `question_or_conflict`. Preserve separate source rows where wording or scope differs. Only then create and populate the canonical knowledge-base skeleton.
