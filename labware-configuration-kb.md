# LabWare Configuration Knowledge Base

> **Status:** Incremental consolidation (Phase 4), 2026-10-14; §§1–4 populated from registered sources, other technical sections remain skeletons. **Not yet a complete implementation specification.** The source documents have not been consolidated into this file. No live LabWare behavior has been independently verified.
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

**Scope:** Laboratory-documented setup of the `ANALYSIS` record, master method and variation associations, base and variation components, result attributes, and aliquoting. Batch templates and QC stock are owned by [§4](#4-batch-configuration); instrument records by [§5](#5-instrument-configuration); parser code by [§6](#6-data-import-and-parsing); calculations, reporting-limit application, and OIS output by [§7](#7-calculations-limits-and-reporting). **Evidence status:** S01/S02 instructions and examples only; no configuration export, routine code, vendor manual, or executed test independently verifies these statements.

### 3.1 Analysis record and conditional settings

**[ANA-RECORD-01] — `DOCUMENTED_REQUIREMENT` — Batched analyses.** S01 instructs that the analysis **Batch Link** match the analysis name exactly and that a Batch Link Table record be created for each analysis to be batched. S02 also instructs that the analysis Batch Link match the batch template. The linked batch-side setup is owned by [§4](#4-batch-configuration); see the architecture summary [§1.2](#12-batch-membership-and-analysis-link-identity). **Evidence:** S01 p. 1, *Analyses*; S02 p. 9, *Required Configuration*.

**[ANA-INST-01] — `DOCUMENTED_REQUIREMENT` — Instrument-file imports or Empower.** The analysis **Instruments → Inst Group** must match the group of the instrument providing the data, including the corresponding instrument component on the batch template. S01 describes the batch result Alias Name as the matching instrument-group value. S02 contains additional `X_STOCK` wording that is not reconciled here. **Evidence:** S01 p. 1, *Analyses*; S02 p. 9, *Required Configuration*. **See:** [§5](#5-instrument-configuration), [Q01](#q01-instrument-reference-fields).

**[ANA-DESTRUCT-01] — `DOCUMENTED_REQUIREMENT` with qualified scope — Destructive analyses.** S02 instructs setting **Other Fields → Destructive** to True for *most analyses that conduct destructive testing* and cautions that this may prevent additional analyses from being assigned at sample creation. This is not a universal setting and the exact assignment impact is not verified. **Evidence:** S02 pp. 9–10, *Required Configuration*. **See:** [§2](#2-ois-integration-project-and-sample-setup).

### 3.2 Master methods, analysis methods, and variations

**[ANA-METHOD-01] — `DOCUMENTED_REQUIREMENT` — Method/variation setup.** S02 says a master method must exist in `T_ANALYSIS_METHOD` before linking methods and variations on `ANALYSIS`. The analysis child **Analysis Methods** configuration pairs each method with a variation, and S02 says every method–variation pair used for sample-login assignment must be populated. S01 instructs variations to have methods matching the configured OSHA method ID. **Do not conflate** `T_ANALYSIS_METHOD` with the analysis child table described as `ANALYSIS_METHOD` / `ANALYSIS_METHODS` in S02. **Evidence:** S01 p. 1, *Analyses*; S02 pp. 10–11, *T_Analysis_Method* and *Methods*, and pp. 15–16, *Multiple Methods*. **See:** [§1.3](#13-method-variation-and-test-assignment-dependencies), [Q03](#q03-methodvariation-relationships).

**[ANA-VAR-01] — `DOCUMENTED_REQUIREMENT` — Variations for the applicable workflow.** S02 says every analytical method has one or more variations and uses the naming pattern `{OSHA method}_{variation type}` for automated analysis assignment. Field-sample variation types are described for **AIR**, **WIPE**, and **BULK**; QC and QCSM variations are described for standard samples; an **ALIQUOT** variation is described when aliquoting is required. S01 specifically instructs creation of `{method}_QC` and `{method}_QCSM` **if the batch uses** those respective samples. These are conditional types, not a checklist requiring all variations for every analysis. **Evidence:** S01 p. 1, *Analyses*; S02 pp. 10–11, *Variations*. **Examples:** `OSHA_5000_AIR`, `OSHA_1023_QCSM`, `OSHA_5001_ALIQUOT` (`IMPLEMENTATION_EXAMPLE`, S02 pp. 10–11).

**[ANA-VAR-02] — `DOCUMENTED_REQUIREMENT` — Variation properties.** S02 instructs assigning applicable **Sample Type** values to field-sample variations, leaving Sample Type empty for QC variations, and linking the analytical method in variation properties. S02 does not specify the full set of actual stored Sample Type values or the behavior when multiple variations qualify. **Evidence:** S02 p. 11, *Variations*. **See:** [§2.3](#23-automated-test-assignment-prerequisites), [Q03](#q03-methodvariation-relationships).

**[ANA-MULTI-01] — `DOCUMENTED_DESCRIPTION` / `IMPLEMENTATION_EXAMPLE` — Compatible methods only.** S02 documents a related-method exception: the analysis method/variation mapping can associate a child method with a variation whose own method is the parent, with `T_ANALYSIS_METHOD.X_RELATED_METHOD` pointing to the parent. The `GC` / `OSHA_111` / `OSHA_5000_PASSIVE` example is indexed in [§1.3](#13-method-variation-and-test-assignment-dependencies); batch compatibility rules belong to [§4](#4-batch-configuration). **Evidence:** S02 pp. 15–16, *Multiple Methods*. Do not assume arbitrary methods are interchangeable.

### 3.3 Base components versus variation components

**[ANA-COMP-01] — `DOCUMENTED_REQUIREMENT` — Analyses with multiple variations; broader applicability stated by S02.** S01 directs that when multiple variations are configured, base components be **Optional = T**, **Reportable = F**, **Displayed = F**; variation components should be **Displayed = T**. S02 instructs that base components be **Optional = T** and **Displayed = F** to avoid showing on other variations, and that variation components be displayed. Preserve the difference in scope and wording; neither source establishes every interaction among base and variation flags. **Evidence:** S01 p. 1, *Analyses*; S02 pp. 11–12, *Component Variations* and *Components*. **See:** [Q02](#q02-component-requiredness), [Q07](#q07-reportability-semantics).

**[ANA-COMP-02] — `DOCUMENTED_REQUIREMENT` — Variation-specific component configuration.** Assign the result components needed by the variation; S02 gives a **BULK** variation's sample-weight component as an example of a component not needed by **AIR** or **QC** variations. S02 says variation components may be **Optional = F** depending on the analysis, while **Displayed = T**. Do not confuse variation component optionality with the batch-result requiredness controls in [§4](#4-batch-configuration). **Evidence:** S02 p. 11, *Component Variations*. **See:** [Q02](#q02-component-requiredness).

**[ANA-COMP-03] — `DOCUMENTED_REQUIREMENT` — Final results intended for reporting.** S01 directs **Reportable = T** on the component variation for results intended for review/release and OIS XML; S02 directs that **only final field-sample results** be reportable for outbound OIS/report delivery and that the variation-level flag be set. S02 says reportability *may also* be set at the original component level, whereas S01 specifies **Reportable = F** for base components in multi-variation analyses. Preserve these separate source instructions and do not resolve precedence without evidence. **Evidence:** S01 pp. 1–2, *Analyses*; S02 pp. 11–13, *Component Variations* and *Reportable Results*. **See:** [§7](#7-calculations-limits-and-reporting), [Q07](#q07-reportability-semantics).

**[ANA-COMP-04] — `DOCUMENTED_REQUIREMENT` — Reportable components sent to OIS.** The reportable component **Alias Name** should match the OIS analyte code. Both sources give Beryllium (Final) for `OSHA_1023` with Alias Name `0365` as an **implementation example**, not a general-purpose alias. **Evidence:** S01 p. 2, *Analyses*; S02 p. 13, *Reportable Results*. **See:** [§7](#7-calculations-limits-and-reporting).

### 3.4 Attributes and instrument-result component flags

**[ANA-ATTR-01] — `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT` — File-parsed results with attributes.** The documented analysis-specific parser supports up to **five** result attributes (`ATTRIBUTE_1`–`ATTRIBUTE_5`), configured/named at the analysis level and populated by the parser where used. S01 says these may contain relevant data associated with result entry; S02 ties them to the analysis-specific parsing routine. The `ICP_OES` example uses **Wavelength**, **Dilution Factor**, **%RSD**, **Timestamp**, and **Interferences**. These names are an `IMPLEMENTATION_EXAMPLE`, not required attributes for other analyses. **Evidence:** S01 p. 1; S02 pp. 11–12, *Data Parsing*. **See:** [§6](#6-data-import-and-parsing).

**[ANA-RAW-01] — `DOCUMENTED_REQUIREMENT` — Raw results populated by instrument file or Empower.** S02 instructs setting **Uses Instrument = T** on raw result components receiving instrument data, and **Has Attributes = T** where attributes are returned from the file or Empower. This is conditional on the data source and result design. **Evidence:** S02 p. 12, *Components*. **See:** [§6](#6-data-import-and-parsing), [§8](#8-empower-integration).

**[ANA-EMPOWER-01] — `DOCUMENTED_REQUIREMENT` with `DOCUMENTED_DESCRIPTION` rationale — Empower peak components.** S02 directs a **Text** data type even for numeric-looking Empower peak results because a missing peak may be returned as `ND` rather than `0`, and describes a type mismatch otherwise. This is an Empower-specific configuration instruction, **not** a general rule for all numeric analysis components. **Evidence:** S02 p. 12, *Empower*. **Owning interface details:** [§8](#8-empower-integration).

### 3.5 Aliquoting and dynamically assigned testing

**[ANA-ALIQUOT-01] — `DOCUMENTED_DESCRIPTION` — Media requiring aliquots.** S02 describes an auto-aliquoting workflow in which `VWF_SLTC_TESTING_BATCH_CREATE` checks `X_OCD_MONITORING.NUM_ALIQUOT` for the method/media when a sample is added to a batch, assigns additional testing, and automatically aliquots to child samples. S02 also requires a separate method aliquot variation when aliquoting is needed. This is a described laboratory workflow, not verified code behavior or a universal rule for all sample media. **Evidence:** S02 p. 11, *Variations* and *Aliquoting*. **See:** [§2](#2-ois-integration-project-and-sample-setup), [§4](#4-batch-configuration), [§9](#9-operational-workflows).

### 3.6 Analysis-side calculation and reporting dependencies (references only)

**[ANA-CALC-REF] — `DOCUMENTED_REQUIREMENT` — File-import final results.** Both sources instruct setting final-result calculations **AutoCalculate = F** for batch file imports to control dependencies between blanks and field unknowns; S02 permits **AutoCalculate = T** for manual entry or Empower. Detailed sequencing and limits belong exclusively to [§7](#7-calculations-limits-and-reporting). **Evidence:** S01 p. 2; S02 p. 12, *Components*. **See:** [Q06](#q06-calculation-mode).

**[ANA-LIMIT-REF] — `DOCUMENTED_DESCRIPTION` — Analysis limits.** Both sources state that Analysis Limits are **not checked inherently** in the described configuration and must be applied during result calculations; S02 discusses reporting-limit propagation. Detailed rules, `CALC_ICP_FINAL`, and formatting belong to [§7](#7-calculations-limits-and-reporting). **Evidence:** S01 p. 1; S02 p. 10, *Analysis Limits* and *Reporting Limits*.

### 3.7 Evidence limits and verification needs

- **Method-table identity:** The sources use `ANALYSIS_METHOD` and `ANALYSIS_METHODS` and distinguish them from `T_ANALYSIS_METHOD`; confirm physical names and relationships before SQL or automation. [Q03](#q03-methodvariation-relationships).
- **Component flag precedence:** Variation-level requiredness, base-level reportability, and their effect on display and external reporting require real configuration and output evidence. [Q02](#q02-component-requiredness), [Q07](#q07-reportability-semantics).
- **Destructive and aliquot effects:** The actual sample-login restrictions and `VWF_SLTC_TESTING_BATCH_CREATE` behavior require code and reproducible workflow evidence.
- **Source-specific conventions:** The examples for `ICP_OES`, `GC`, and named OSHA methods are not evidence of correct configuration for unrelated analyses; see [§11](#11-established-configuration-examples).
- **No vendor/runtime verification:** Nothing in §3 is `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED` at this stage.

## 4. Batch Configuration

**Scope:** Laboratory-documented configuration of Batch Test Templates, links to analyses and tests, batch protocols, QC and stock references, batch result components, and method compatibility. This section distinguishes manual-entry, instrument-file, and Empower workflows. The detailed parser interface and file-processing sequence belong to [§6](#6-data-import-and-parsing); instrument master records to [§5](#5-instrument-configuration); result calculations and limits to [§7](#7-calculations-limits-and-reporting); operational review to [§9](#9-operational-workflows). **Evidence status:** S01/S02 instructions and descriptions only; no configuration export, source code, vendor reference, or executed verification test has been supplied.

### 4.1 Batch template identity and core settings

**[BAT-TEMPLATE-01] — `DOCUMENTED_REQUIREMENT` — Normal method-oriented Test Batch configuration.** S01 requires a **Batch Test Template Name** exactly matching the OSHA method for which it is designed (example `OSHA_1023`), **Assign To Objects = True**, and an **Analysis Link Key** matching the analysis name. S02 describes batch configuration at the `T_ANALYSIS_METHOD` level, a **Name Prefix** using the OSHA method name, **Object Class = TEST**, and **Batch Type = Test Batch**. The distinction between S01's template-name requirement and S02's name-prefix setting is preserved; do not assume they are the same field. **Evidence:** S01 p. 2, *Batch*; S02 pp. 13–14, *Batch Configuration / Required Configuration*. **Related:** [§1.2](#12-batch-membership-and-analysis-link-identity), [§3.1](#31-analysis-record-and-conditional-settings).

**[BAT-TEMPLATE-02] — `RECOMMENDATION` — Creating a new method template.** S02 recommends copying an existing Batch Test Template and renaming it for the new method, because the existing template may contain preconfigured QC-stock selection, a batch-review window, and illustrative batch-result components. This is a laboratory recommendation, **not** proof that any copied settings are appropriate for a different analysis; review all copied values. **Evidence:** S02 p. 13, *Required Configuration*.

**[BAT-LINK-01] — `DOCUMENTED_REQUIREMENT` — Analyses assigned to a test batch.** S01 requires a new Batch Link record for each analysis to be batched and matching analysis-side Batch Link / batch-side Analysis Link Key values. S02 identifies the batch link type as **Analysis Link Key** and says it matches `ANALYSIS.NAME`; tests must belong to an analysis matching the batch link. Exact field and lookup behavior has not been independently checked. **Evidence:** S01 pp. 1–2; S02 pp. 14–15, *Required Configuration* and *Batch Tests*. **Related:** [§1.2](#12-batch-membership-and-analysis-link-identity).

### 4.2 Test membership and compatible methods

**[BAT-MEMBER-01] — `DOCUMENTED_DESCRIPTION` — Test batches described in S02.** S02 states that samples are linked to batches through their **tests**, not directly: `TEST.BATCH` contains the batch name, and `BATCH_OBJECTS.OBJECT_ID = TEST.TEST_NUMBER`. Do not infer this model applies to every LabWare batch class. **Evidence:** S02 p. 15, *Batch Tests*. **Architecture owner:** [§1.2](#12-batch-membership-and-analysis-link-identity).

**[BAT-METHOD-01] — `DOCUMENTED_DESCRIPTION` / `IMPLEMENTATION_EXAMPLE` — Compatible-method exceptions.** S02 says a template ordinarily covers one analytical method but some compatible methods may be tested together. In its GC example, the child `OSHA_111` method maps to the `OSHA_5000_PASSIVE` variation (whose method is `OSHA_5000`), and `T_ANALYSIS_METHOD.X_RELATED_METHOD` on `OSHA_111` points to `OSHA_5000`. The analysis method/variation mapping must also be configured. This example does **not** establish a general compatibility rule. **Evidence:** S02 pp. 15–16, *Multiple Methods*. **See:** [§3.2](#32-master-methods-analysis-methods-and-variations), [Q03](#q03-methodvariation-relationships).

### 4.3 Batch protocols, standards, and QC stock

**[BAT-PROTOCOL-01] — `DOCUMENTED_REQUIREMENT` with conditional applicability — Batches requiring standard/QC samples.** S01 says the **Batch Protocol** can add QC samples automatically when a batch is created. S02 describes a **Batch Standard Protocol** that adds configured standard sets, whose records are held in `QC_SAMPLES`. S02 lists typical examples: calibration blank (`CB`), standards (`STD_A`, `STD_B`, etc.), initial calibration verification (`ICV`), and continuing calibration verification (`CCV`). The examples are not a universal required QC composition. **Evidence:** S01 p. 2, *Batch*; S02 pp. 13, 16, *Required Configuration* and *Standards*.

**[BAT-QC-VAR-01] — `DOCUMENTED_REQUIREMENT` — Templates using QC variations.** S01 directs the **Batch Variation** in the batch configuration submenu to point to the QC method variation (example `OSHA_1023_QC`); S02 similarly specifies that the configuration window's **Variation** points to the method's QC analysis variation. Create the corresponding analysis variation when the batch uses QC samples. **Evidence:** S01 pp. 1–2; S02 p. 14, *Required Configuration*. **Variation setup owner:** [§3.2](#32-master-methods-analysis-methods-and-variations).

**[BAT-QCSM-01] — `DOCUMENTED_REQUIREMENT` — Templates using QCSM stock.** S01 says **X_QC_STOCK** on the **parent batch configuration page**, not the submenu opened by Configure, should point to allowed QCSM stock entries; this controls which QCSM stock may be added. S02 says to configure and attach a **QCSM Stock** record under **QC Stock**. The exact equivalence or interaction of these UI labels has not been verified. **Evidence:** S01 p. 2, *Batch*; S02 p. 14, *Required Configuration*.

**[BAT-RLSM-01] — `DOCUMENTED_REQUIREMENT` — Batch templates requiring reporting-limit stock per S02.** S02 instructs configuring an **RLSM Stock** record and attaching it as **RL Stock** on the batch template. S01 does not discuss this field; do not infer its necessity for every possible batch type without examining the analysis workflow. **Evidence:** S02 p. 14, *Required Configuration*. **Reporting-limit calculation owner:** [§7](#7-calculations-limits-and-reporting).

**[BAT-CCV-01] — `DOCUMENTED_DESCRIPTION` — Repeated CCV runs in the documented OSHA batch workflow.** S02 describes CCVs as potentially recurring (typically once per ten field samples). It directs keeping **one batch sample per QC type** and adding additional **analysis replicates** to that sample when multiple CCV runs are required, then ordering those tests appropriately. This is laboratory-specific guidance, not a universal LabWare restriction or a verified scheduling algorithm. **Evidence:** S02 p. 16, *CCVs*. **Related:** [§6](#6-data-import-and-parsing) for file-driven replication and reordering.

### 4.4 Batch result components and reference fields

**[BAT-COMP-01] — `DOCUMENTED_REQUIREMENT` — Batch result components referring to stock.** S01 directs filling **Other Fields → X_STOCK** with the appropriate stock item for stock-referencing results. S02 specifies `BATCH_COMPONENT.X_STOCK = STOCK.NAME` for **Stock Used** and says **Standards Used** is configured similarly. For standard components, S02 additionally states `BATCH_COMPONENT.ALIAS_NAME = QC_SAMPLES.NAME` to match the QC samples in the batch standard protocol. **Evidence:** S01 p. 2, *Batch Result Components*; S02 pp. 14–15, *Required Configuration*.

**[BAT-COMP-02] — `DOCUMENTED_REQUIREMENT` — Instrument-identifying batch results.** S01 requires the instrument result **Alias Name** to match the instrument group. S02 specifies `BATCH_COMPONENT.ALIAS_NAME = INSTRUMENT.INST_GROUP` and describes **Instrument Used** components, including additional components when multiple instruments are involved. S02 elsewhere mentions the batch result `X_STOCK` field in relation to instruments; this is **unresolved**, not silently replaced by the alias rule. **Evidence:** S01 p. 2; S02 pp. 9, 14–15. **See:** [§5](#5-instrument-configuration), [Q01](#q01-instrument-reference-fields).

**[BAT-COMP-03] — `DOCUMENTED_REQUIREMENT` — Instrument-file imports only.** S01 directs that the batch result identifying the instrument producing the parsed file have **Other Fields → Data File Source = True**, with **only one Data File Source per batch**. It also requires an **Instrument File** batch result with type **DB File Name**. S02 likewise identifies **Instrument File** among the batch results needed for processing. Do not apply these requirements automatically to manual-entry or Empower-only batches. **Evidence:** S01 p. 2, *Batch Result Components*; S02 p. 14, *Required Configuration*. **Parser/file handling owner:** [§6](#6-data-import-and-parsing).

**[BAT-COMP-04] — `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT` — Batch component entry calculations.** S02 gives illustrative **Instrument Used** calculation logic (`SELECT BATCH_COMPONENT.ALIAS_NAME` into `instGroupName`; `GOSUB BC_INSTRUMENT_BROWSE`; `RETURN instrumentName`), **Stock Used** / **Standards Used** logic (`GOSUB CALC_ING_SCAN_DIALOG`; `RETURN`), and directs **Auto-calc false** for these batch component calculations. These are source-provided configurations, not independently validated code snippets or a complete executable routine. **Evidence:** S02 pp. 14–15, *Required Configuration*. **See:** [§5](#5-instrument-configuration) and [§7](#7-calculations-limits-and-reporting).

**[BAT-PRECISION-01] — `DOCUMENTED_REQUIREMENT` with `IMPLEMENTATION_EXAMPLE` — QCSM precision results where configured.** S02 calls for **one QCSM Precision record per reportable recovery (F/T)** on the batched analysis, with **Auto-calc false**, `targetElement` set to the applicable recovery component, and `GOSUB CALC_BATCH_QC_REC_PRECISION`. The example target is `Be F/T`; it must not be copied unchanged to unrelated analyses. The precise meaning of “F/T” and runtime calculation behavior have not been independently established. **Evidence:** S02 p. 15, *QCSM Precision*. **Calculation owner:** [§7](#7-calculations-limits-and-reporting).

### 4.5 Pre-analysis results and entry-mode differences

**[BAT-PREREQ-01] — `DOCUMENTED_REQUIREMENT` — Instrument-file import; also described for Empower in S02.** S01 states that batch results with **Optional = F** must be entered before a data file can be imported. S02 says all **non-optional batch results** must be entered before file parsing or Empower testing proceeds, and its batch-template details refer to instrument/stock results marked **Required = T** as blocking parsing until populated. These descriptions may reflect different UI terms or contexts; **do not assume field equivalence** until verified. For `OSHA_1023`, S01 gives the example that lab solutions and instruments other than the balance must be entered before file processing. **Evidence:** S01 p. 2; S02 pp. 8, 14–15. **See:** [Q02](#q02-component-requiredness).

**[BAT-MANUAL-01] — `DOCUMENTED_DESCRIPTION` — Manual batch result entry.** S02 states that **manual-entry batches do not perform batch-level validation of the Batch Results table** and that the analyst and reviewer must ensure required entries are present. This is a documented laboratory workflow description, not proof of all possible manual-entry validation behavior. **Evidence:** S02 p. 16, *Manual Data Entry*. **See:** [§9](#9-operational-workflows), [Q09](#q09-manual-batch-checks).

**[BAT-FILE-01] — `DOCUMENTED_DESCRIPTION` — File-driven batches.** S02 describes a manually initiated **Batch → Batch Process Results** routine, even though automatic file imports are possible, because the OSHA import workflow requires complex validation and data-dependent logic. The routine checks the **Instrument File** result before importing and calls an analysis-specific parser. This entry establishes the batch-side boundary only; routine dispatch, validation-only and commit loops, file lifecycle, and returned variables are owned by [§6](#6-data-import-and-parsing). **Evidence:** S02 pp. 8, 16–18, *Batch Testing / Data Parsing* and *Data File Batches*.

**[BAT-EMPOWER-01] — `DOCUMENTED_DESCRIPTION` — Empower batches.** S02 describes instrument/project consistency checks and additional configuration in both LabWare and Empower. The batch's instrument selection participates in the project check; no inference is made that an Empower-only batch requires the file-parser's **Data File Source** or **Instrument File** configuration. **Evidence:** S02 pp. 19–21, *Empower Batches*. **Owning details:** [§8](#8-empower-integration).

### 4.6 Evidence limitations and verification priorities

- **Template and link fields:** Confirm template name versus name prefix, **Assign To Objects**, Batch Link/Analysis Link Key records, and physical lookup behavior from configuration exports. See [§1.2](#12-batch-membership-and-analysis-link-identity).
- **Instrument reference ambiguity:** Inspect actual batch components to distinguish **Alias Name**, `X_STOCK`, instrument group, and instrument barcode relationships. See [Q01](#q01-instrument-reference-fields).
- **Requiredness semantics:** Determine whether **Optional = F**, **Required = T**, and S02's “non-optional” wording describe one control or different contexts. See [Q02](#q02-component-requiredness).
- **QC and stock applicability:** Confirm which methods require QCSM/RLSM stocks, the exact relationship between **X_QC_STOCK** and **QC Stock**, and how repeated CCVs and precision calculations behave.
- **Manual and automated validation:** Confirm actual behavior of manual batch checks, file import prerequisites, and Empower gating with code/configuration and reproducible tests. See [Q09](#q09-manual-batch-checks), [§6](#6-data-import-and-parsing), [§8](#8-empower-integration).
- **No vendor/runtime verification:** Nothing in §4 is yet classified `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`. Method-specific examples are not universal configuration templates.

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
| 2026-10-14 | Populated §3 analysis configuration, method/variation setup, components, attributes and aliquoting; retained conditional rules and unresolved questions. | S01 pp. 1–2; S02 pp. 9–13, 15–16. Source-documented, not independently verified. |
| 2026-10-14 | Populated §4 batch templates, test membership, protocols, QC/stock references, batch components and entry-mode distinctions; retained unresolved terminology and field questions. | S01 pp. 1–2; S02 pp. 8, 13–21. Source-documented, not independently verified. |

### 14.3 Maintenance procedure

For each material addition or correction: identify the source/evidence → classify the claim and applicability → update its one owning section → add links to examples/questions as appropriate → update source manifest or crosswalk when needed → record the change here → commit to Git. No separate approval process is required for this single-maintainer project.
