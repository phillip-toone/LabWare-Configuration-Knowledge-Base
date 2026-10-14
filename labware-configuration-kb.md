# LabWare Configuration Knowledge Base

> **Status:** Structural skeleton (Phase 3), 2026-10-14. **Not yet an implementation specification.** The source documents have not been consolidated into this file. No live LabWare behavior has been independently verified.
>
> **Audience:** Laboratory implementers and LLM assistants supporting the OSHA Technical Center LabWare LIMS implementation.

## 0. Purpose, Usage, and Evidence Rules

### 0.1 Purpose and boundaries

This is the intended **single canonical location** for accepted laboratory-specific configuration guidance. It will cover analysis implementation, batch testing, instrument imports, Empower, OIS-related workflows, and evidence-backed examples. Until a section is populated and reviewed, its heading **does not** imply that a procedure or requirement is established.

The original Word documents remain immutable evidence snapshots in [`source/`](source/); see the [source manifest](source/manifest.md). The [requirements crosswalk](crosswalk/requirements.csv) is an extraction and reconciliation workspace, **not** a second canonical technical reference. The [quality review](crosswalk/quality-review.md) documents extraction limitations. The [structure review](structure-review.md) records the initial taxonomy assessment.

### 0.2 Instructions for LLM use

1. Determine the requested workflow and its applicability (file import, manual entry, Empower, method, media, QC, or analysis-specific exception) **before** applying a rule.
2. Answer from populated canonical sections, with section anchors and source IDs where available. Do not treat headings, examples, or unresolved questions as instructions.
3. Distinguish laboratory requirements from vendor-documented behavior and from observed or verified implementation behavior. **No vendor technical manual is currently registered.**
4. If a statement is missing, conditional, contradicted, or unverified, say so explicitly; cite the relevant open question and request the missing evidence rather than inventing configuration values.
5. Use the crosswalk to audit provenance and find source statements, not to silently override the canonical text. When sources disagree, preserve both claims until reconciled with evidence.
6. Do not infer that an existing configuration is correct, that a method-specific example is universal, or that a recommendation is mandatory.
7. Prefer links to the owning technical section instead of repeating instructions elsewhere in this document.

### 0.3 Evidence classification

| Label | Meaning | Use restriction |
|---|---|---|
| `DOCUMENTED_REQUIREMENT` | Explicit instruction in a registered laboratory source | Does not by itself establish correctness or runtime verification. |
| `DOCUMENTED_DESCRIPTION` | Source describes implementation or behavior | Do not elevate to a universal requirement. |
| `IMPLEMENTATION_EXAMPLE` | Specific method, analysis, instrument, or workflow illustration | Do not generalize without support. |
| `RECOMMENDATION` | Suggested practice | Not mandatory unless independently adopted as a requirement. |
| `OBSERVED` | Configuration or behavior evidenced by a supplied record/export | Observation is not proof of correctness. |
| `VERIFIED` | Behavior supported by a recorded reproducible test | Include environment, date, test, outcome, and evidence. |
| `VENDOR_DOCUMENTED` | Statement grounded in an identified vendor reference | Must cite registered vendor edition and location. |
| `UNRESOLVED` | Missing, ambiguous, or conflicting information | Do not resolve by inference. |

The crosswalk also contains `CONTEXT` and `CONFLICT` working classifications; these are extraction/review markers, not validated technical requirements. A topic may have several types of evidence at once.

### 0.4 Provenance and entry conventions

When substantive guidance is added, identify its **stable topic/requirement ID**, scope, statement, evidence class, source location, verification status, and any linked open question. A compact entry pattern is:

```text
**ID:** [stable ID]  **Class:** [classification]  **Applies to:** [conditions]
**Guidance:** [source-supported statement]
**Evidence:** [S01/S02 and precise page/heading or future evidence ID]
**Verification:** [not tested / evidence record]
**See also:** [owning section / question ID]
```

Use existing crosswalk IDs when suitable; do not manufacture source locations or verification records. Source citations should use the registered IDs and precise locations, e.g., `S02, p. 17, Data File Batches`; for extraction traceability, reference `crosswalk/requirements.csv` IDs. When evidence changes, update the canonical statement in its **one owning section**, adjust crosswalk status if needed, and record the material change in §14.

### 0.5 Topic ownership and duplication rule

Sections 1–9 own reusable technical guidance; §10 is a linked implementation sequence, §11 owns analysis-specific examples and exceptions, §12 owns reproducible troubleshooting, §13 tracks uncertainty, and §14 tracks sources and changes. A cross-reference may summarize why another section matters, but must not copy its full technical rule.

## 1. Configuration Architecture and Dependencies

**Owner:** LabWare object relationships, key fields and identifiers, dependency map, naming and link conventions.

**To develop:** ANALYSIS / analysis methods / T_ANALYSIS_METHOD distinctions; TEST-to-batch relationships; batch link matching; variation and parser naming; configuration prerequisites. Preserve distinctions among similarly named tables.

**Sources to review:** S01 pp. 1–3; S02 pp. 9–11, 14–16. **Open:** [Q01](#q01-instrument-reference-fields), [Q03](#q03-methodvariation-relationships).

## 2. OIS Integration, Project, and Sample Setup

**Owner:** Inbound OIS request processing, OCD data model, media/analyte-to-test assignment, project/sample templates, receiving dependencies.

**To develop:** SFTP/shell-script intake boundaries, XML validation/staging, `X_OCD_MONITORING` relationships, sample media, project and sample setup. Undocumented template and validation steps must remain marked as gaps.

**Sources to review:** S02 pp. 6–8. **Open:** [G01](#g01-xml-and-projectsample-setup).

## 3. Analysis Configuration

**Owner:** Analysis record, master methods, method/variation links, variation and component configuration, attributes, aliquoting.

**To develop:** Conditional variation types; base versus variation component flags; reportable components' analysis-side setup; attributes; destructive flag; method mapping and aliquot logic. External reporting behavior belongs to §7.

**Sources to review:** S01 pp. 1–2; S02 pp. 9–13. **Open:** [Q02](#q02-component-requiredness), [Q03](#q03-methodvariation-relationships), [Q07](#q07-reportability-semantics).

## 4. Batch Configuration

**Owner:** Batch templates, protocols, QC and stock records, batch components, required pre-analysis results, test membership, compatible methods.

**To develop:** Template properties; batch link records; stock/standard/QC components; QC sample and CCV handling; manual versus file-driven batch prerequisites. Parser internals belong to §6.

**Sources to review:** S01 p. 2; S02 pp. 13–16. **Open:** [Q01](#q01-instrument-reference-fields), [Q02](#q02-component-requiredness), [Q09](#q09-manual-batch-checks).

## 5. Instrument Configuration

**Owner:** Instrument records, instrument group identity, instrument parameters and links to batch/analysis configuration.

**To develop:** `INSTRUMENTS` configuration, Parameter 1 and Parameter 2 usage and scope. Parser dispatch details belong to §6; Empower-specific project checks belong to §8.

**Sources to review:** S01 p. 2; S02 pp. 9, 15, 19. **Open:** [Q01](#q01-instrument-reference-fields), [Q04](#q04-parser-dispatch).

## 6. Data Import and Parsing

**Owner:** Instrument file intake, import directory constants, parser invocation, interfaces, validation-only and commit phases, reordering, file lifecycle.

**To develop:** `SLTC_DATA_IMPORT_{analysisName}` and `BM_SLTC_DATA_PARSE_{analysisName}` conventions, `resultDataArr` interface, conditional parser context variables, validation and retry behavior. Keep file-import-only settings explicitly scoped.

**Sources to review:** S01 pp. 2–3; S02 pp. 8, 16–18. **Open:** [Q04](#q04-parser-dispatch), [Q05](#q05-parser-contract), [Q08](#q08-path-constant-granularity), [G02](#g02-parser-failure-and-recovery).

## 7. Calculations, Limits, and Reporting

**Owner:** Analysis limits, reporting-limit transformations, result calculations and execution order, recovery, reportability and OIS outbound results.

**To develop:** `ANALYSIS_LIMITS.MIN_LIMIT`, `RESULT.X_REPORTING_LIMIT`, lower-limit formatting, `AutoCalculate` by workflow, analyte-code alias and outbound XML. Do not infer runtime enforcement without evidence.

**Sources to review:** S01 pp. 1–2; S02 pp. 10, 12–13. **Open:** [Q06](#q06-calculation-mode), [Q07](#q07-reportability-semantics), [G03](#g03-release-and-ois-response).

## 8. Empower Integration

**Owner:** Empower project setup, custom fields, interface mapping, expected project validation, Empower-specific data types.

**To develop:** Empower project custom fields and sample/result mapping, instrument Parameter 2, `ND` value handling, interface workflow. Do not generalize from an existing project mapping without verification.

**Sources to review:** S02 pp. 12, 19–21. **Open:** [G04](#g04-empower-exception-behavior).

## 9. Operational Workflows

**Owner:** Sample receiving, batch creation, batch testing, review/release, project closure, end-to-end operational sequence.

**To develop:** Documented entry points and prerequisites; mark receiving, review, release, and closure procedures as incomplete where only headings exist. Detailed object configuration stays in §§2–8.

**Sources to review:** S02 pp. 6–9, 16–18. **Open:** [G03](#g03-release-and-ois-response).

## 10. End-to-End Analysis Implementation Procedure

**Owner:** Ordered implementation checklist that links to established rules rather than reproducing them.

**Status:** **Not yet established.** Populate after §§1–9 have evidence-backed content. The sequence must branch for manual entry, instrument-file import, and Empower, and distinguish necessary setup from optional QC/aliquot features.

**Planned checkpoints:** Identify analysis/method/media → establish dependencies → configure analysis/variation/components → configure batch/instrument/integration as applicable → configure calculations/reporting → test assignment, result processing and release → capture evidence and unresolved issues. **This is a proposed organization, not a validated operating procedure.**

## 11. Established Configuration Examples

**Owner:** Method-specific records, deviations, observed choices, and verified examples. Every example must identify its analysis, method, configuration snapshot, evidence, and applicability.

### 11.1 ICP_OES / OSHA_1023

**Status:** Source example identified, **not yet consolidated or independently verified**. See S01 pp. 1–3; S02 pp. 11–12 and related references. Link reusable rules to §§3–7 rather than duplicating them.

### 11.2 Other named analyses

**Status:** S02 p. 21 lists `SILICA_XRD`, `GRAV_PREWEIGH`, and `GRAV_POSTWEIGH` headings without substantive analysis-specific instructions. **Do not infer configuration details.**

## 12. Troubleshooting and Verified Behaviors

**Owner:** Reproducible failures, tests, root causes, confirmed workarounds, and environmental limitations.

**Status:** No independent verification records supplied. Described behavior in S01/S02 is not automatically `VERIFIED`.

**Future record fields:** ID; symptom; environment/version; preconditions; reproduction steps; expected/actual outcome; evidence reference; resolution; verification date; applicable analyses.

## 13. Open Questions and Undocumented Areas

The entries below are **investigation prompts, not confirmed defects**. Full initial discussion: [structure-review.md](structure-review.md). Record resolutions with evidence and update the owning section; retain the question history.

### Q01 — Instrument reference fields

S01 and S02 describe batch component alias matching instrument group; S02 also mentions `X_STOCK` in an instrument-related context. Determine the actual role of each field. **Unresolved.**

### Q02 — Component requiredness

Clarify base/variation component `Optional`, batch `Optional`, and batch `Required` semantics across screens. **Unresolved.**

### Q03 — Method/variation relationships

Reconcile naming patterns with media-specific variations and related-method exceptions without assuming all suffixes or method pairings are universal. **Unresolved.**

### Q04 — Parser dispatch

Verify LabStation versus instrument-specific subroutine dispatch and Parameter 1 dependencies from code. **Unresolved.**

### Q05 — Parser contract

Determine which context variables and return values are mandatory, optional, or conditional, including `stopParse`, `validationStr`, `wrongBatch`, and reorder arrays. **Unresolved.**

### Q06 — Calculation mode

Verify `AutoCalculate` behavior and sequencing separately for file-import, manual, and Empower workflows. **Unresolved.**

### Q07 — Reportability semantics

Determine base versus variation reportability effects on review/release and OIS XML. **Unresolved.**

### Q08 — Path constant granularity

S01 describes an analysis-level path constant and suggests other granularity may be possible; verify before documenting any alternative as supported. **Unresolved.**

### Q09 — Manual batch checks

Determine what validations exist in manual workflows; do not assume file-import preconditions apply. **Unresolved.**

### G01 — XML and project/sample setup

S02 has headings for XML schema, staging, investigations, and templates without sufficient procedures. **Undocumented.**

### G02 — Parser failure and recovery

Partial failure, rollback, reimport, and retry behavior are not fully established. **Undocumented.**

### G03 — Release and OIS response

Review, release, closure, and outbound response generation need operational and configuration evidence. **Undocumented.**

### G04 — Empower exception behavior

Mapping and setup are described, but runtime errors and recovery need evidence. **Undocumented.**

## 14. Source Register and Change History

### 14.1 Registered source snapshots

| ID | Source | Role |
|---|---|---|
| `S01` | [`source/Batch-and-Analysis-Configuration-Requirements.docx`](source/Batch-and-Analysis-Configuration-Requirements.docx) | Laboratory batch/analysis and file-parsing guidance; ICP_OES examples. |
| `S02` | [`source/Sample-Testing-Configuration.docx`](source/Sample-Testing-Configuration.docx) | Broader laboratory OIS-to-LabWare implementation draft; contains incomplete sections. |

For SHA-256 checksums, version information, and source status, consult [`source/manifest.md`](source/manifest.md). **No vendor manual, configuration export, or independently verified test record is currently registered.** A future LabWare 8 technical reference must be added as a distinct source, not treated as already available.

### 14.2 Change history

| Date | Change | Evidence/qualification |
|---|---|---|
| 2026-10-14 | Created Phase 3 canonical-document skeleton and topic ownership; registered existing S01/S02 by reference. | Structure based on `structure-review.md`, `source/manifest.md`, and reviewed crosswalk. No technical requirements consolidated or verified. |

### 14.3 Maintenance procedure

For each material addition or correction: identify the source/evidence → classify the claim and applicability → update its one owning section → add links to examples/questions as appropriate → update source manifest or crosswalk when needed → record the change here → commit to Git. No separate approval process is required for this single-maintainer project.
