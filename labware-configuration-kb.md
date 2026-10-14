# LabWare Configuration Knowledge Base

> **Status:** Incremental consolidation (Phase 4), 2026-10-14; §§1–2 populated from registered sources, other technical sections remain skeletons. **Not yet a complete implementation specification.** The source documents have not been consolidated into this file. No live LabWare behavior has been independently verified.
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

**Scope:** This section describes the laboratory-documented relationships among configuration objects and the dependencies that must be understood before configuring an analysis. Detailed field settings and procedures remain in their owning sections (§§2–8). **Evidence status:** source-documented only; no configuration export, routine code, vendor manual, or executed test has yet been used to verify these relationships.

### 1.1 Object model and relationship boundaries

| Object / table | Documented role or relationship | Evidence and classification | Detail owner |
|---|---|---|---|
| `ANALYSIS` | Analysis configuration is associated with instrument group, batch link, attributes, variations, components, and method/variation assignments. | `DOCUMENTED_DESCRIPTION` — S01 p. 1; S02 pp. 9–12 | [§3](#3-analysis-configuration) |
| `T_ANALYSIS_METHOD` | Separate master method table. A method record is described as a prerequisite to associating methods and variations with an analysis. It also carries `X_RELATED_METHOD` for the documented compatible-method scenario. | `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT` — S02 pp. 10, 15–16 | [§3](#3-analysis-configuration), [§4](#4-batch-configuration) |
| `ANALYSIS_METHOD` / `ANALYSIS_METHODS` | Child relationship of `ANALYSIS`, used to pair a method with an analysis variation. S02 uses both singular and plural spellings; the exact physical table name must be checked before writing queries. It is **not** the same object as `T_ANALYSIS_METHOD`. | `DOCUMENTED_DESCRIPTION` — S02 pp. 10–11, 15–16 | [§3](#3-analysis-configuration); [Q03](#q03-methodvariation-relationships) |
| Analysis Variation | Identifies the variation used for testing; has an associated method and component-level settings. Media, QC, QCSM and aliquot variation types are documented, conditionally. | `DOCUMENTED_DESCRIPTION` — S01 p. 1; S02 pp. 10–11 | [§3](#3-analysis-configuration) |
| `TEST` | The object assigned to a batch, rather than the `SAMPLE` directly. | `DOCUMENTED_DESCRIPTION` — S02 p. 15, Batch Tests | [§4](#4-batch-configuration) |
| Batch Test Template / Batch Link | Batch templates are generally organized by OSHA method; an Analysis Link Key / Batch Link record associates eligible analyses with batches. | `DOCUMENTED_REQUIREMENT` with conditional scope — S01 pp. 1–2; S02 pp. 13–15 | [§4](#4-batch-configuration) |
| `INSTRUMENTS` | Instrument group and parameters support instrument selection, parsing and, for some workflows, Empower project identity. | `DOCUMENTED_DESCRIPTION` — S01 p. 2; S02 pp. 9, 19 | [§5](#5-instrument-configuration) |
| `X_OCD_MONITORING` and `X_ANALYTE_CODE` | OCD records relate requested analyte codes and sample media to monitoring methods used in automated test assignment. | `DOCUMENTED_DESCRIPTION` — S02 pp. 6–7 | [§2](#2-ois-integration-project-and-sample-setup) |
| LIMS Constant and parsing subroutines | Additional objects for instrument-file imports; not generally required for manual batches without file parsing. | `DOCUMENTED_DESCRIPTION` — S01 pp. 1, 3; S02 pp. 16–18 | [§6](#6-data-import-and-parsing) |

**Boundary:** These are relationships described by the laboratory sources, not an independently verified LabWare schema. In particular, do not conflate the analysis child method mapping with the master `T_ANALYSIS_METHOD` table.

### 1.2 Batch membership and analysis link identity

**[ARCH-BATCH-01] — `DOCUMENTED_DESCRIPTION` — Applies to test batches.** S02 states that `TEST.BATCH` holds the batch name and that `BATCH_OBJECTS.OBJECT_ID = TEST.TEST_NUMBER`; samples are not directly attached to batches. This is the documented membership model, not a claim that every possible LabWare batch class uses the same relationship. **Evidence:** S02 p. 15, *Batch Tests*. **Details:** [§4](#4-batch-configuration).

**[ARCH-LINK-01] — `DOCUMENTED_REQUIREMENT` — Applies when an analysis is batched.** S01 directs that the analysis Batch Link and batch template Analysis Link Key match the analysis name exactly, with a new Batch Link record for each analysis to be batched. S02 describes a Batch Link of type **Analysis Link Key** matching `ANALYSIS.NAME`, and separately states that the analysis Batch Link must match the batch template. Read these as linked laboratory configuration instructions; the exact underlying field names and lookup mechanics are not independently verified. **Evidence:** S01 pp. 1–2, *Analyses* and *Batch*; S02 pp. 9, 14–15, *Required Configuration* and *Batch Tests*. **Details:** [§3](#3-analysis-configuration), [§4](#4-batch-configuration).

### 1.3 Method, variation and test-assignment dependencies

**[ARCH-METHOD-01] — `DOCUMENTED_REQUIREMENT` — Applies to method/variation assignment.** S02 states that master methods must have a record in `T_ANALYSIS_METHOD` before the method/variation link can be configured, and that the analysis's method-to-variation pairs are used for test assignment at sample login. S01 likewise requires methods on associated variations to match the OSHA method identifier in the method table. **Evidence:** S01 p. 1; S02 pp. 10–11, *T_Analysis_Method* and *Methods*. **Details:** [§3](#3-analysis-configuration).

**[ARCH-ASSIGN-01] — `DOCUMENTED_DESCRIPTION` — Applies to the documented OIS automated assignment workflow.** Assignment uses the requested analyte code and sample medium: S02 specifies a matching `X_OCD_MONITORING` record for analyte code and media, together with an analysis variation associated with the corresponding method. This describes the documented assignment prerequisites, not the complete implementation of inbound XML processing. **Evidence:** S02 pp. 6–7, *Test Assignment*. **Details:** [§2](#2-ois-integration-project-and-sample-setup).

**[ARCH-MULTI-01] — `DOCUMENTED_DESCRIPTION` with `IMPLEMENTATION_EXAMPLE` — Applies only when compatible methods share a batch/variation.** S02 describes a related-method mechanism using `T_ANALYSIS_METHOD.X_RELATED_METHOD` to point from child to parent method, alongside the analysis method/variation association. Its GC example pairs method `OSHA_111` with variation `OSHA_5000_PASSIVE` whose variation method is `OSHA_5000`; the child method's related method is `OSHA_5000`. **Do not infer** that every method pair may be combined. **Evidence:** S02 pp. 15–16, *Multiple Methods*. **Details:** [§4](#4-batch-configuration); [Q03](#q03-methodvariation-relationships).

### 1.4 Naming and matching conventions (architecture index)

These are **laboratory-documented conventions**, not general vendor naming rules. The owning sections specify the fields and workflow conditions.

| Convention or match | Documented scope | Evidence / destination |
|---|---|---|
| Analysis Batch Link / template Analysis Link Key ↔ `ANALYSIS.NAME` | Batched analyses; exact match required in source instructions | S01 pp. 1–2; S02 pp. 14–15; [§4](#4-batch-configuration) |
| Batch template name or name prefix ↔ OSHA method name | Normal single-method batch setup; S01 requires exact template name, while S02 explicitly names the prefix | S01 p. 2; S02 pp. 13–14; [§4](#4-batch-configuration) |
| Variation name `{method}_{variation type}` | Documented automated assignment and variation patterns, e.g. `_AIR`, `_QC`, `_QCSM`, `_ALIQUOT`; actual required variations depend on workflow | S01 p. 1; S02 pp. 10–11; [§3](#3-analysis-configuration) |
| Batch instrument result Alias Name ↔ instrument group ↔ analysis instrument group | Instrument-file import or Empower as described; `X_STOCK` wording remains unresolved | S01 pp. 1–2; S02 pp. 9, 14; [§5](#5-instrument-configuration), [Q01](#q01-instrument-reference-fields) |
| `SLTC_DATA_IMPORT_{analysisName}` | Documented instrument-file import constant | S01 p. 3; [§6](#6-data-import-and-parsing) |
| `BM_SLTC_DATA_PARSE_{analysisName}` | Documented analysis-specific file parser name | S01 p. 3; S02 p. 17; [§6](#6-data-import-and-parsing) |
| `BM_SLTC_DATA_PARSE_{analysisName}_{instrumentName}` | Instrument-specific parser when LabStation parsing is not used; instrument name ties to Parameter 1 value in S01 | S01 pp. 2–3; [§6](#6-data-import-and-parsing), [Q04](#q04-parser-dispatch) |
| Reportable component Alias Name ↔ OIS analyte code | Outbound reporting of reportable field-sample results | S01 p. 2; S02 p. 13; [§7](#7-calculations-limits-and-reporting) |

### 1.5 Conditional dependency map

The following is a **source-derived planning sequence**, not a verified execution order or a complete implementation procedure:

1. **For automated OIS test assignment:** establish OCD analyte/media/method mappings and the relevant master method and analysis variation/method relationships. See [§2](#2-ois-integration-project-and-sample-setup) and [§3](#3-analysis-configuration). (S02 pp. 6–7, 10–11.)
2. **For a batched analysis:** establish its analysis/batch link relationship, method-oriented batch template and test membership configuration. See [§4](#4-batch-configuration). (S01 pp. 1–2; S02 pp. 13–15.)
3. **For an instrument-file batch:** additionally establish the matching instrument identity/group, file-path constant, batch file result and parsing routines. See [§5](#5-instrument-configuration) and [§6](#6-data-import-and-parsing). (S01 pp. 1–3; S02 pp. 16–18.)
4. **For Empower:** use the instrument and analysis relationships plus the project-specific interface configuration described in [§8](#8-empower-integration). Do not assume the file-parser requirements apply to Empower. (S02 pp. 12, 19–21.)
5. **For manual data entry:** the S01 introduction says file-parser subroutines may be omitted when there is no file to parse; S02 separately describes manual batch-result checks. See [§4](#4-batch-configuration), [§9](#9-operational-workflows) and [Q09](#q09-manual-batch-checks). (S01 p. 1; S02 p. 16.)

### 1.6 Limits of current evidence

- **Unverified schema details:** Exact physical names and meanings of similarly named method tables, link keys, and instrument-reference fields require configuration/schema evidence; see [Q01](#q01-instrument-reference-fields) and [Q03](#q03-methodvariation-relationships).
- **Unverified dispatch behavior:** The exact selection of instrument-specific parsing routines and which variables are mandatory at runtime require source-code evidence; see [Q04](#q04-parser-dispatch) and [Q05](#q05-parser-contract).
- **No universal correctness inference:** The `ICP_OES` and GC examples illustrate configurations described by S01/S02, not validated templates for all analyses; see [§11](#11-established-configuration-examples).
- **No vendor validation:** The separate LabWare 8 technical manual has not been provided to this project. Nothing in this section is classified `VENDOR_DOCUMENTED` or `VERIFIED`.

## 2. OIS Integration, Project, and Sample Setup

**Scope:** This section owns the documented OIS request-intake boundary, Occupational Chemical Database (OCD) reference data, and automated sample test-assignment prerequisites. It also records what the sources do **not** yet explain about project/sample creation and receiving. Analysis method and variation setup is owned by [§3](#3-analysis-configuration); operational receiving and release workflows are owned by [§9](#9-operational-workflows). **Evidence status:** S02 source-documented only; no XML samples, schema, scripts, configuration exports, or executed tests have been provided.

### 2.1 End-to-end OIS context and intake boundary

**[OIS-CONTEXT-01] — `DOCUMENTED_DESCRIPTION` — Applies to the documented OTC/OIS workflow.** S02 describes receiving testing requests from OIS, creating and managing samples and associated analyses in LabWare, reporting results, and generating a response file to OIS. This is the stated scope of the workflow, **not** evidence that each stage is configured or validated. **Evidence:** S02 p. 5, *Purpose*. **See also:** [§9](#9-operational-workflows), [G01](#g01-xml-and-projectsample-setup), [G03](#g03-release-and-ois-response).

**[OIS-TRANSFER-01] — `DOCUMENTED_DESCRIPTION` — Applies to inbound OIS requests and instrument files as described in S02.** S02 says both file types arrive by SFTP, outside native LabWare functionality; Windows shell scripts executed by a LabWare background instance handle the transfers. The scripts and server directory layouts vary by application. S02 states that each script consists of a `.bat` and a `.txt` file, but does not provide script contents, schedules, retry rules, path details, or authentication configuration. **Evidence:** S02 p. 6, *File Transfer Process* and *Shell Script Maintenance*. **See also:** [§6](#6-data-import-and-parsing) for instrument-result file processing, which is a separate concern.

**Not established:** The XML schema location, parsing and validation rules, staging-table structures, investigation handling, and the exact mechanism that creates LabWare projects and samples are headings without sufficient procedure in S02 p. 6. Do not infer these from the SFTP description. **Tracking:** [G01](#g01-xml-and-projectsample-setup).

### 2.2 Occupational Chemical Database reference tables

**[OIS-OCD-01] — `DOCUMENTED_DESCRIPTION` — Applies to the OCD representation described for this implementation.** S02 says the OCD originated as a document on the DOL intranet and was parsed into the following four LabWare tables. Their roles are **source descriptions**, not independently checked schema definitions. **Evidence:** S02 p. 7, *Occupational Chemical Database (OCD)*.

| Table | Role described in S02 |
|---|---|
| `X_OCD_STANDARD` | Holds standard/material-group names and associated hazard information; parent of the other OCD tables. |
| `X_OCD_MONITORING` | Monitoring-method records; links `X_ANALYTE_CODE` to `X_OCD_STANDARD` and holds testing methods for analyte codes according to sample media. |
| `X_OCD_PHYS_PROP` | Physical properties of a standard. |
| `X_OCD_LITERATURE` | Literature references for a standard, organized by source. |

**[OIS-CODE-01] — `DOCUMENTED_DESCRIPTION` — Applies to analyte-code identity in the documented OCD.** S02 calls analyte codes *substance codes* or *IMIS codes* and describes them as unique identifiers pairing standards with test methods. It states that one standard may have multiple analyte codes, while each analyte code belongs to one standard. The code is a **four-character alphanumeric primary key** in `X_ANALYTE_CODE`. Preserve codes as strings, including leading zeroes; this handling is a data-integrity recommendation inferred from the documented identifier format, not a separately documented LabWare setting. **Evidence:** S02 p. 7, *Analyte Code*. **See also:** [§7](#7-calculations-limits-and-reporting) for outbound reportable-result aliases.

**[OIS-MEDIA-01] — `DOCUMENTED_DESCRIPTION` — Applies to the documented sample categories.** S02 identifies three sample categories: **Air**, **Wipe**, and **Bulk**, with distinct entries in the monitoring-method data. The document does not establish the exact database values, case sensitivity, or the full list of media records for each category. **Evidence:** S02 p. 7, *Sample Types*.

### 2.3 Automated test-assignment prerequisites

**[OIS-ASSIGN-01] — `DOCUMENTED_REQUIREMENT` — Applies when an inbound OIS sample is assigned analyses automatically.** S02 says analyses are assigned using **both sample media and requested analyte codes**. For assignment, the inbound sample must match a record in `X_OCD_MONITORING` on **Analyte Code and Media**, and the analysis must have an **Analysis Variation with the corresponding method assigned**. Both conditions are documented as prerequisites; neither alone is sufficient according to S02. **Evidence:** S02 pp. 6–7, *Test Assignment*. **Related architecture:** [§1.3](#13-method-variation-and-test-assignment-dependencies); **configuration owner:** [§3](#3-analysis-configuration).

**[OIS-ASSIGN-02] — `DOCUMENTED_REQUIREMENT` — Applies to the analysis-side method/variation mapping used at sample login.** S02 states that every method–variation pair must be populated in the analysis's Analysis Methods configuration, which is used during sample-login assignment. S02 also requires a corresponding master method record in `T_ANALYSIS_METHOD` before method/variation linking. These are cross-referenced here rather than restated as a full configuration procedure. **Evidence:** S02 pp. 10–11, *T_Analysis_Method* and *Methods*. **Owning detail:** [§3](#3-analysis-configuration); **naming ambiguity:** [Q03](#q03-methodvariation-relationships).

**[OIS-ASSIGN-03] — `DOCUMENTED_DESCRIPTION` — Applies when field-sample variations are selected by media.** S02 documents media-specific analysis variations, including an example `OSHA_5000_AIR`, and says applicable Sample Type values must be configured on field-sample variations; the QC variation Sample Type is left empty. These are analysis-side settings and do **not** establish the full algorithm for choosing among multiple matching variations. **Evidence:** S02 pp. 10–11, *Variations*. **Owning detail:** [§3](#3-analysis-configuration).

**Documented dependency summary (not a verified algorithm):** Requested analyte code + sample medium → matching `X_OCD_MONITORING` monitoring method → corresponding analysis method/variation association → eligible test assignment. This summarizes the prerequisites above; it does not specify precedence, handling of multiple matches, missing matches, or transaction/error behavior. **Evidence:** S02 pp. 6–7, 10–11.

### 2.4 Project and sample creation: documented limits

S02 p. 6 includes headings for *XML Parsing and Validation*, *Schema location*, *Staging Tables*, *Investigations*, *Project Template*, and *Sample Template* but supplies no actionable field configuration or end-to-end creation sequence under them. **Classification: `UNRESOLVED` / documentation gap.** No project-template name, sample-template name, XML field mapping, staging-table column, or validation rule should be invented. **Tracking:** [G01](#g01-xml-and-projectsample-setup).

S02 p. 7 provides the OCD/test-assignment dependencies described in [§2.3](#23-automated-test-assignment-prerequisites), but those dependencies do not independently demonstrate how a project or sample is instantiated.

### 2.5 Sample receiving and downstream boundaries

S02 p. 8 lists *Sample Receiving Workflow*, *Seal Status*, and *Label Printing* without procedural details. **Classification: `UNRESOLVED`.** Do not infer required seal values, label formats, barcode rules, or receiving-state transitions. These belong to [§9](#9-operational-workflows) once evidence is supplied.

The end-to-end purpose includes outbound OIS reporting, but S02 p. 9 lists *Project Closure and OIS Reporting* without the necessary closure or response-generation procedure. Reportable result flags and analyte-code aliases are discussed elsewhere in the sources; their technical ownership is [§7](#7-calculations-limits-and-reporting). **Tracking:** [G03](#g03-release-and-ois-response).

### 2.6 Evidence limitations and next verification inputs

- **Unverified mapping:** Actual `X_OCD_MONITORING` records, `X_ANALYTE_CODE` records, and method/variation records are needed to check the documented joins and assignment outcomes. **No sample data or schema export has been supplied.**
- **Unverified intake:** A sanitized OIS request XML, its schema, and relevant transfer/processing routines are needed to establish parsing, validation, project/sample creation, and failure handling. See [G01](#g01-xml-and-projectsample-setup).
- **Unverified workflow:** Receiving, seal, label, investigation, and closure procedures require workflow configuration or observed test evidence. See [§9](#9-operational-workflows) and [G03](#g03-release-and-ois-response).
- **No vendor claims:** Nothing in this section is classified `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`; the source describes the laboratory implementation, not a validated general LabWare product contract.

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
| 2026-10-14 | Populated §1 architecture and dependencies, with conditional scope, source references, and unresolved questions retained. | S01 pp. 1–3; S02 pp. 6–7, 9–11, 13–21. Source-documented, not independently verified. |
| 2026-10-14 | Populated §2 OIS integration, OCD, test assignment, and explicitly undocumented project/sample and receiving procedures. | S02 pp. 5–9, 10–11. Source-documented, not independently verified. |

### 14.3 Maintenance procedure

For each material addition or correction: identify the source/evidence → classify the claim and applicability → update its one owning section → add links to examples/questions as appropriate → update source manifest or crosswalk when needed → record the change here → commit to Git. No separate approval process is required for this single-maintainer project.
