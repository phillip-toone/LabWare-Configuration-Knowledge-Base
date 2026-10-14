# LabWare Configuration Knowledge Base

> **Status:** Standalone operational reference, 2026-10-14; §§1–12 contain documented configuration guidance and recommendations, and §13 tracks unresolved questions. **Not a complete implementation specification.** Coverage is not certified exhaustive, and no live LabWare behavior has been independently verified.
>
> **Audience:** Laboratory implementers and LLM assistants supporting the OSHA Technical Center LabWare LIMS implementation.

## 0. Operational Document Status and Usage

### 0.1 Status and boundaries

This standalone operational reference presents laboratory-documented LabWare configuration guidance, conditional requirements, recommendations, examples, and unresolved questions. It is not a complete implementation specification or a validated standard operating procedure. No live behavior, vendor documentation, configuration exports, or executed tests have been independently verified for this document. Do not treat documented behavior as verified behavior.

### 0.2 Instructions for LLM use

1. Determine the workflow and applicability (file import, manual entry, Empower, method, media, QC, or analysis-specific exception) before applying a rule.
2. Use the owning technical sections and their internal links. Do not treat headings, examples, or unresolved questions as instructions.
3. Distinguish documented laboratory requirements, descriptions, recommendations, implementation examples, observations, verified behavior, and vendor-documented claims.
4. When information is missing, conditional, contradictory, or unverified, state that explicitly and consult the linked open question rather than inventing values.
5. Do not infer that an existing configuration is correct, that an example is universal, or that a recommendation is mandatory.

### 0.3 Evidence classification

| Label | Meaning | Use restriction |
|---|---|---|
| `DOCUMENTED_REQUIREMENT` | Explicit laboratory instruction | Does not establish correctness or runtime verification. |
| `DOCUMENTED_DESCRIPTION` | Laboratory description of implementation or behavior | Do not elevate to a universal requirement. |
| `IMPLEMENTATION_EXAMPLE` | Specific method, analysis, instrument, or workflow illustration | Do not generalize without support. |
| `RECOMMENDATION` | Suggested practice | Not mandatory unless adopted as a requirement. |
| `OBSERVED` | Configuration or behavior supported by a supplied record/export | Observation is not proof of correctness. |
| `VERIFIED` | Behavior supported by a recorded reproducible test | Requires environment, date, test, outcome, and evidence. |
| `VENDOR_DOCUMENTED` | Statement grounded in an identified vendor reference | Requires identifiable vendor evidence. |
| `UNRESOLVED` | Missing, ambiguous, or conflicting information | Do not resolve by inference. |

### 0.4 Rule ownership and verification

Sections 1–9 own reusable technical guidance; §10 is a linked recommended implementation sequence; §11 contains method-specific examples and exceptions; §12 covers troubleshooting and verified-behavior status; §13 tracks unresolved questions. Preserve stable rule and question identifiers when using or updating this reference. For any new verification claim, record the environment, date, test, outcome, and supporting evidence. No independently observed or verified behavior is established by this edition.

## 1. Configuration Architecture and Dependencies

**Scope:** This section describes the laboratory-documented relationships among configuration objects and the dependencies that must be understood before configuring an analysis. Detailed field settings and procedures remain in their owning sections (§§2–8). **Verification status:** Laboratory-documented only; not independently tested.

### 1.1 Object model and relationship boundaries

| Object / table | Documented role or relationship | Classification / verification status | Detail owner |
|---|---|---|---|
| `ANALYSIS` | Analysis configuration is associated with instrument group, batch link, attributes, variations, components, and method/variation assignments. | `DOCUMENTED_DESCRIPTION`; not independently verified | [§3](#3-analysis-configuration) |
| `T_ANALYSIS_METHOD` | Separate master method table. A method record is described as a prerequisite to associating methods and variations with an analysis. It also carries `X_RELATED_METHOD` for the documented compatible-method scenario. | `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT`; not independently verified | [§3](#3-analysis-configuration), [§4](#4-batch-configuration) |
| `ANALYSIS_METHOD` / `ANALYSIS_METHODS` | Child relationship of `ANALYSIS`, used to pair a method with an analysis variation. Both singular and plural spellings appear in the documented terminology; the exact physical table name must be checked before writing queries. It is **not** the same object as `T_ANALYSIS_METHOD`. | `DOCUMENTED_DESCRIPTION`; not independently verified | [§3](#3-analysis-configuration); [Q03](#q03-methodvariation-relationships) |
| Analysis Variation | Identifies the variation used for testing; has an associated method and component-level settings. Media, QC, QCSM and aliquot variation types are documented, conditionally. | `DOCUMENTED_DESCRIPTION`; not independently verified | [§3](#3-analysis-configuration) |
| `TEST` | The object assigned to a batch, rather than the `SAMPLE` directly. | `DOCUMENTED_DESCRIPTION`; not independently verified | [§4](#4-batch-configuration) |
| Batch Test Template / Batch Link | Batch templates are generally organized by OSHA method; an Analysis Link Key / Batch Link record associates eligible analyses with batches. | `DOCUMENTED_REQUIREMENT` with conditional scope; not independently verified | [§4](#4-batch-configuration) |
| `INSTRUMENTS` | Instrument group and parameters support instrument selection, parsing and, for some workflows, Empower project identity. | `DOCUMENTED_DESCRIPTION`; not independently verified | [§5](#5-instrument-configuration) |
| `X_OCD_MONITORING` and `X_ANALYTE_CODE` | OCD records relate requested analyte codes and sample media to monitoring methods used in automated test assignment. | `DOCUMENTED_DESCRIPTION`; not independently verified | [§2](#2-ois-integration-project-and-sample-setup) |
| LIMS Constant and parsing subroutines | Additional objects for instrument-file imports; not generally required for manual batches without file parsing. | `DOCUMENTED_DESCRIPTION`; not independently verified | [§6](#6-data-import-and-parsing) |

**Boundary:** These are relationships described by the laboratory sources, not an independently verified LabWare schema. In particular, do not conflate the analysis child method mapping with the master `T_ANALYSIS_METHOD` table.

### 1.2 Batch membership and analysis link identity

**[ARCH-BATCH-01] — `DOCUMENTED_DESCRIPTION` — Applies to test batches.** The laboratory guidance states that `TEST.BATCH` holds the batch name and that `BATCH_OBJECTS.OBJECT_ID = TEST.TEST_NUMBER`; samples are not directly attached to batches. This is the documented membership model, not a claim that every possible LabWare batch class uses the same relationship. **Details:** [§4](#4-batch-configuration).

**[ARCH-LINK-01] — `DOCUMENTED_DESCRIPTION` — Batched analyses.** The documented architecture connects an analysis to a test-batch template through matching Batch Link / Analysis Link Key values. Exact field-level instructions and unresolved distinctions are covered in [§4.1](#41-batch-template-identity-and-core-settings); the analysis-side setting is indexed in [§3.1](#31-analysis-record-and-conditional-settings).

### 1.3 Method, variation and test-assignment dependencies

**[ARCH-METHOD-01] — `DOCUMENTED_REQUIREMENT` — Applies to method/variation assignment.** The laboratory guidance states that master methods must have a record in `T_ANALYSIS_METHOD` before the method/variation link can be configured, and that the analysis's method-to-variation pairs are used for test assignment at sample login. The documented instructions also require methods on associated variations to match the OSHA method identifier in the method table. **Details:** [§3](#3-analysis-configuration).

**[ARCH-ASSIGN-01] — `DOCUMENTED_DESCRIPTION` — Applies to the documented OIS automated assignment workflow.** Assignment uses the requested analyte code and sample medium: the documented assignment prerequisites specify a matching `X_OCD_MONITORING` record for analyte code and media, together with an analysis variation associated with the corresponding method. This describes the documented assignment prerequisites, not the complete implementation of inbound XML processing. **Details:** [§2](#2-ois-integration-project-and-sample-setup).

**[ARCH-MULTI-01] — `DOCUMENTED_DESCRIPTION` with `IMPLEMENTATION_EXAMPLE` — Applies only when compatible methods share a batch/variation.** The laboratory guidance describes a related-method mechanism using `T_ANALYSIS_METHOD.X_RELATED_METHOD` to point from child to parent method, alongside the analysis method/variation association. Its GC example pairs method `OSHA_111` with variation `OSHA_5000_PASSIVE` whose variation method is `OSHA_5000`; the child method's related method is `OSHA_5000`. **Do not infer** that every method pair may be combined. **Details:** [§4](#4-batch-configuration); [Q03](#q03-methodvariation-relationships).

### 1.4 Naming and matching conventions (architecture index)

These are **laboratory-documented conventions**, not general vendor naming rules. The owning sections specify the fields and workflow conditions.

| Convention or match | Documented scope | Applicability / destination |
|---|---|---|
| Analysis Batch Link / template Analysis Link Key ↔ `ANALYSIS.NAME` | Batched analyses; exact match required in laboratory instructions | [§4](#4-batch-configuration) |
| Batch template name or name prefix ↔ OSHA method name | Normal single-method batch setup; the exact template-name instruction and the separate name-prefix instruction are both documented | [§4](#4-batch-configuration) |
| Variation name `{method}_{variation type}` | Documented automated assignment and variation patterns, e.g. `_AIR`, `_QC`, `_QCSM`, `_ALIQUOT`; actual required variations depend on workflow | [§3](#3-analysis-configuration) |
| Batch instrument result Alias Name ↔ instrument group ↔ analysis instrument group | Instrument-file import or Empower as described; `X_STOCK` wording remains unresolved | [§5](#5-instrument-configuration), [Q01](#q01-instrument-reference-fields) |
| `SLTC_DATA_IMPORT_{analysisName}` | Documented instrument-file import constant | [§6](#6-data-import-and-parsing) |
| `BM_SLTC_DATA_PARSE_{analysisName}` | Documented analysis-specific file parser name | [§6](#6-data-import-and-parsing) |
| `BM_SLTC_DATA_PARSE_{analysisName}_{instrumentName}` | Instrument-specific parser when LabStation parsing is not used; instrument name ties to Parameter 1 value in the laboratory guidance | [§6](#6-data-import-and-parsing), [Q04](#q04-parser-dispatch) |
| Reportable component Alias Name ↔ OIS analyte code | Outbound reporting of reportable field-sample results | [§7](#7-calculations-limits-and-reporting) |

### 1.5 Conditional dependency map

The following is a **guidance-derived planning sequence**, not a verified execution order or a complete implementation procedure:

1. **For automated OIS test assignment:** establish OCD analyte/media/method mappings and the relevant master method and analysis variation/method relationships. See [§2](#2-ois-integration-project-and-sample-setup) and [§3](#3-analysis-configuration).
2. **For a batched analysis:** establish its analysis/batch link relationship, method-oriented batch template and test membership configuration. See [§4](#4-batch-configuration).
3. **For an instrument-file batch:** additionally establish the matching instrument identity/group, file-path constant, batch file result and parsing routines. See [§5](#5-instrument-configuration) and [§6](#6-data-import-and-parsing).
4. **For Empower:** use the instrument and analysis relationships plus the project-specific interface configuration described in [§8](#8-empower-integration). Do not assume the file-parser requirements apply to Empower.
5. **For manual data entry:** File-parser subroutines may be omitted when there is no file to parse; manual batch-result checks are addressed separately. See [§4](#4-batch-configuration), [§9](#9-operational-workflows) and [Q09](#q09-manual-batch-checks).

### 1.6 Limits of current evidence

- **Unverified schema details:** Exact physical names and meanings of similarly named method tables, link keys, and instrument-reference fields require configuration/schema evidence; see [Q01](#q01-instrument-reference-fields) and [Q03](#q03-methodvariation-relationships).
- **Unverified dispatch behavior:** The exact selection of instrument-specific parsing routines and which variables are mandatory at runtime require source-code evidence; see [Q04](#q04-parser-dispatch) and [Q05](#q05-parser-contract).
- **No universal correctness inference:** The `ICP_OES` and GC examples illustrate configurations described by the laboratory guidance, not validated templates for all analyses; see [§11](#11-established-configuration-examples).
- **No vendor validation:** No LabWare 8 vendor technical manual was available for independent verification. Nothing in this section is classified `VENDOR_DOCUMENTED` or `VERIFIED`.

## 2. OIS Integration, Project, and Sample Setup

**Scope:** This section owns the documented OIS request-intake boundary, Occupational Chemical Database (OCD) reference data, and automated sample test-assignment prerequisites. It also records what the laboratory guidance does **not** yet explain about project/sample creation and receiving. Analysis method and variation setup is owned by [§3](#3-analysis-configuration); operational receiving and release workflows are owned by [§9](#9-operational-workflows). **Verification status:** Laboratory-documented only; not independently tested.

### 2.1 End-to-end OIS context and intake boundary

**[OIS-CONTEXT-01] — `DOCUMENTED_DESCRIPTION` — Applies to the documented OTC/OIS workflow.** The laboratory guidance describes receiving testing requests from OIS, creating and managing samples and associated analyses in LabWare, reporting results, and generating a response file to OIS. This is the stated scope of the workflow, **not** evidence that each stage is configured or validated. **See also:** [§9](#9-operational-workflows), [G01](#g01-xml-and-projectsample-setup), [G03](#g03-release-and-ois-response).

**[OIS-TRANSFER-01] — `DOCUMENTED_DESCRIPTION` — Applies to inbound OIS requests and instrument files as described in the laboratory guidance.** Both file types are described as arriving by SFTP, outside native LabWare functionality; Windows shell scripts executed by a LabWare background instance handle the transfers. The scripts and server directory layouts vary by application. The laboratory guidance states that each script consists of a `.bat` and a `.txt` file, but does not provide script contents, schedules, retry rules, path details, or authentication configuration. **See also:** [§6](#6-data-import-and-parsing) for instrument-result file processing, which is a separate concern.

**Not established:** The XML schema location, parsing and validation rules, staging-table structures, investigation handling, and the exact mechanism that creates LabWare projects and samples are headings without sufficient procedure in the laboratory guidance. Do not infer these from the SFTP description. **Tracking:** [G01](#g01-xml-and-projectsample-setup).

### 2.2 Occupational Chemical Database reference tables

**[OIS-OCD-01] — `DOCUMENTED_DESCRIPTION` — Applies to the OCD representation described for this implementation.** The OCD originated as a document on the DOL intranet and was subsequently parsed into the following four LabWare tables. Their roles are **laboratory descriptions**, not independently checked schema definitions.

| Table | Documented role |
|---|---|
| `X_OCD_STANDARD` | Holds standard/material-group names and associated hazard information; parent of the other OCD tables. |
| `X_OCD_MONITORING` | Monitoring-method records; links `X_ANALYTE_CODE` to `X_OCD_STANDARD` and holds testing methods for analyte codes according to sample media. |
| `X_OCD_PHYS_PROP` | Physical properties of a standard. |
| `X_OCD_LITERATURE` | Literature references for a standard, organized by source. |

**[OIS-CODE-01] — `DOCUMENTED_DESCRIPTION` — Applies to analyte-code identity in the documented OCD.** Analyte codes are described as *substance codes* or *IMIS codes* and as unique identifiers pairing standards with test methods. It states that one standard may have multiple analyte codes, while each analyte code belongs to one standard. The code is a **four-character alphanumeric primary key** in `X_ANALYTE_CODE`. Preserve codes as strings, including leading zeroes; this handling is a data-integrity recommendation inferred from the documented identifier format, not a separately documented LabWare setting. **See also:** [§7](#7-calculations-limits-and-reporting) for outbound reportable-result aliases.

**[OIS-MEDIA-01] — `DOCUMENTED_DESCRIPTION` — Applies to the documented sample categories.** The laboratory guidance identifies three sample categories: **Air**, **Wipe**, and **Bulk**, with distinct entries in the monitoring-method data. The available guidance does not establish the exact database values, case sensitivity, or the full list of media records for each category.

### 2.3 Automated test-assignment prerequisites

**[OIS-ASSIGN-01] — `DOCUMENTED_REQUIREMENT` — Applies when an inbound OIS sample is assigned analyses automatically.** The laboratory guidance says analyses are assigned using **both sample media and requested analyte codes**. For assignment, the inbound sample must match a record in `X_OCD_MONITORING` on **Analyte Code and Media**, and the analysis must have an **Analysis Variation with the corresponding method assigned**. Both conditions are documented as prerequisites; neither alone is sufficient according to the laboratory guidance. **Related architecture:** [§1.3](#13-method-variation-and-test-assignment-dependencies); **configuration owner:** [§3](#3-analysis-configuration).

**[OIS-ASSIGN-02] — `DOCUMENTED_REQUIREMENT` — Applies to the analysis-side method/variation mapping used at sample login.** The laboratory guidance states that every method–variation pair must be populated in the analysis's Analysis Methods configuration, which is used during sample-login assignment. Another laboratory instruction also requires a corresponding master method record in `T_ANALYSIS_METHOD` before method/variation linking. These are cross-referenced here rather than restated as a full configuration procedure. **Owning detail:** [§3](#3-analysis-configuration); **naming ambiguity:** [Q03](#q03-methodvariation-relationships).

**[OIS-ASSIGN-03] — `DOCUMENTED_DESCRIPTION` — Applies when field-sample variations are selected by media.** The laboratory guidance documents media-specific analysis variations, including an example `OSHA_5000_AIR`, and says applicable Sample Type values must be configured on field-sample variations; the QC variation Sample Type is left empty. These are analysis-side settings and do **not** establish the full algorithm for choosing among multiple matching variations. **Owning detail:** [§3](#3-analysis-configuration).

**Documented dependency summary (not a verified algorithm):** Requested analyte code + sample medium → matching `X_OCD_MONITORING` monitoring method → corresponding analysis method/variation association → eligible test assignment. This summarizes the prerequisites above; it does not specify precedence, handling of multiple matches, missing matches, or transaction/error behavior.

### 2.4 Project and sample creation: documented limits

The laboratory guidance includes headings for *XML Parsing and Validation*, *Schema location*, *Staging Tables*, *Investigations*, *Project Template*, and *Sample Template* but supplies no actionable field configuration or end-to-end creation sequence under them. **Classification: `UNRESOLVED` / documentation gap.** No project-template name, sample-template name, XML field mapping, staging-table column, or validation rule should be invented. **Tracking:** [G01](#g01-xml-and-projectsample-setup).

the laboratory guidance provides the OCD/test-assignment dependencies described in [§2.3](#23-automated-test-assignment-prerequisites), but those dependencies do not independently demonstrate how a project or sample is instantiated.

### 2.5 Sample receiving and downstream boundaries

The laboratory guidance lists *Sample Receiving Workflow*, *Seal Status*, and *Label Printing* without procedural details. **Classification: `UNRESOLVED`.** Do not infer required seal values, label formats, barcode rules, or receiving-state transitions. These belong to [§9](#9-operational-workflows) once evidence is supplied.

The end-to-end purpose includes outbound OIS reporting, but the laboratory guidance lists *Project Closure and OIS Reporting* without the necessary closure or response-generation procedure. Reportable result flags and analyte-code aliases are discussed elsewhere in the laboratory guidance; their technical ownership is [§7](#7-calculations-limits-and-reporting). **Tracking:** [G03](#g03-release-and-ois-response).

### 2.6 Evidence limitations and next verification inputs

- **Unverified mapping:** Actual `X_OCD_MONITORING` records, `X_ANALYTE_CODE` records, and method/variation records are needed to check the documented joins and assignment outcomes. **No sample data or schema export has been supplied.**
- **Unverified intake:** A sanitized OIS request XML, its schema, and relevant transfer/processing routines are needed to establish parsing, validation, project/sample creation, and failure handling. See [G01](#g01-xml-and-projectsample-setup).
- **Unverified workflow:** Receiving, seal, label, investigation, and closure procedures require workflow configuration or observed test evidence. See [§9](#9-operational-workflows) and [G03](#g03-release-and-ois-response).
- **No vendor claims:** Nothing in this section is classified `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`; the laboratory guidance describes the laboratory implementation, not a validated general LabWare product contract.

## 3. Analysis Configuration

**Scope:** Laboratory-documented setup of the `ANALYSIS` record, master method and variation associations, base and variation components, result attributes, and aliquoting. Batch templates and QC stock are owned by [§4](#4-batch-configuration); instrument records by [§5](#5-instrument-configuration); parser code by [§6](#6-data-import-and-parsing); calculations, reporting-limit application, and OIS output by [§7](#7-calculations-limits-and-reporting). **Verification status:** Laboratory-documented only; not independently tested.

### 3.1 Analysis record and conditional settings

**[ANA-RECORD-01] — `DOCUMENTED_REQUIREMENT` — Batched analyses.** One laboratory instruction specifies that the analysis **Batch Link** match the analysis name exactly and that a Batch Link Table record be created for each analysis to be batched. Another laboratory instruction also instructs that the analysis Batch Link match the batch template. The linked batch-side setup is owned by [§4](#4-batch-configuration); see the architecture summary [§1.2](#12-batch-membership-and-analysis-link-identity).

**[ANA-INST-01] — `DOCUMENTED_REQUIREMENT` — Instrument-file imports or Empower.** Set the analysis **Instruments → Inst Group** for the applicable instrument workflow. The matching convention, batch component aliases, and unresolved `X_STOCK` wording are owned by [§5.1](#51-instrument-group-and-analysisbatch-matching) and [Q01](#q01-instrument-reference-fields).

**[ANA-DESTRUCT-01] — `DOCUMENTED_REQUIREMENT` with qualified scope — Destructive analyses.** Another laboratory instruction specifies setting **Other Fields → Destructive** to True for *most analyses that conduct destructive testing* and cautions that this may prevent additional analyses from being assigned at sample creation. This is not a universal setting and the exact assignment impact is not verified. **See:** [§2](#2-ois-integration-project-and-sample-setup).

### 3.2 Master methods, analysis methods, and variations

**[ANA-METHOD-01] — `DOCUMENTED_REQUIREMENT` — Method/variation setup.** The documented method configuration requires a master method to exist in `T_ANALYSIS_METHOD` before linking methods and variations on `ANALYSIS`. The analysis child **Analysis Methods** configuration pairs each method with a variation, and the laboratory guidance states every method–variation pair used for sample-login assignment must be populated. One laboratory instruction specifies variations to have methods matching the configured OSHA method ID. **Do not conflate** `T_ANALYSIS_METHOD` with the analysis child table described as `ANALYSIS_METHOD` / `ANALYSIS_METHODS` in the laboratory guidance. **See:** [§1.3](#13-method-variation-and-test-assignment-dependencies), [Q03](#q03-methodvariation-relationships).

**[ANA-VAR-01] — `DOCUMENTED_REQUIREMENT` — Variations for the applicable workflow.** The documented variation guidance states that every analytical method has one or more variations and uses the naming pattern `{OSHA method}_{variation type}` for automated analysis assignment. Field-sample variation types are described for **AIR**, **WIPE**, and **BULK**; QC and QCSM variations are described for standard samples; an **ALIQUOT** variation is described when aliquoting is required. The laboratory guidance specifically instructs creation of `{method}_QC` and `{method}_QCSM` **if the batch uses** those respective samples. These are conditional types, not a checklist requiring all variations for every analysis.  **Examples:** `OSHA_5000_AIR`, `OSHA_1023_QCSM`, `OSHA_5001_ALIQUOT` (`IMPLEMENTATION_EXAMPLE`, the laboratory guidance).

**[ANA-VAR-02] — `DOCUMENTED_REQUIREMENT` — Variation properties.** Another laboratory instruction specifies assigning applicable **Sample Type** values to field-sample variations, leaving Sample Type empty for QC variations, and linking the analytical method in variation properties. The laboratory guidance does not specify the full set of actual stored Sample Type values or the behavior when multiple variations qualify. **See:** [§2.3](#23-automated-test-assignment-prerequisites), [Q03](#q03-methodvariation-relationships).

**[ANA-MULTI-01] — `DOCUMENTED_DESCRIPTION` / `IMPLEMENTATION_EXAMPLE` — Compatible methods only.** The laboratory guidance documents a related-method exception: the analysis method/variation mapping can associate a child method with a variation whose own method is the parent, with `T_ANALYSIS_METHOD.X_RELATED_METHOD` pointing to the parent. The `GC` / `OSHA_111` / `OSHA_5000_PASSIVE` example is indexed in [§1.3](#13-method-variation-and-test-assignment-dependencies); batch compatibility rules belong to [§4](#4-batch-configuration).

### 3.3 Base components versus variation components

**[ANA-COMP-01] — `DOCUMENTED_REQUIREMENT` — Analyses with multiple variations; broader applicability also described.** One documented instruction directs that when multiple variations are configured, base components be **Optional = T**, **Reportable = F**, **Displayed = F**; variation components should be **Displayed = T**. Another laboratory instruction specifies that base components be **Optional = T** and **Displayed = F** to avoid showing on other variations, and that variation components be displayed. Preserve the difference in scope and wording; the available guidance does not establish every interaction among base and variation flags. **See:** [Q02](#q02-component-requiredness), [Q07](#q07-reportability-semantics).

**[ANA-COMP-02] — `DOCUMENTED_REQUIREMENT` — Variation-specific component configuration.** Assign the result components needed by the variation; the laboratory guidance gives a **BULK** variation's sample-weight component as an example of a component not needed by **AIR** or **QC** variations. The laboratory guidance states variation components may be **Optional = F** depending on the analysis, while **Displayed = T**. Do not confuse variation component optionality with the batch-result requiredness controls in [§4](#4-batch-configuration). **See:** [Q02](#q02-component-requiredness).

**[ANA-COMP-03] — `DOCUMENTED_REQUIREMENT` — Reportability flags at component level.** For reportable final field-sample results, the laboratory guidance instructs **Reportable = T** at the variation-component level. One documented instruction specifies **Reportable = F** on base components for multi-variation analyses; another permits base-level reportability. These instructions have not been reconciled. Preserve both source scopes; flag precedence is unresolved.

**[ANA-COMP-04] — `DOCUMENTED_REQUIREMENT` — Reportable components sent to OIS.** The reportable component **Alias Name** should match the OIS analyte code. The laboratory guidance give Beryllium (Final) for `OSHA_1023` with Alias Name `0365` as an **implementation example**, not a general-purpose alias. **See:** [§7](#7-calculations-limits-and-reporting).

### 3.4 Attributes and instrument-result component flags

**[ANA-ATTR-01] — `DOCUMENTED_REQUIREMENT` (attribute configuration) with `IMPLEMENTATION_EXAMPLE` (ICP_OES names) — File-parsed results with attributes.** The documented analysis-specific parser supports up to **five** result attributes (`ATTRIBUTE_1`–`ATTRIBUTE_5`), configured/named at the analysis level and populated by the parser where used. The laboratory guidance states these may contain relevant data associated with result entry; the laboratory guidance ties them to the analysis-specific parsing routine. The `ICP_OES` example uses **Wavelength**, **Dilution Factor**, **%RSD**, **Timestamp**, and **Interferences**. These names are an `IMPLEMENTATION_EXAMPLE`, not required attributes for other analyses. **See:** [§6](#6-data-import-and-parsing).

**[ANA-RAW-01] — `DOCUMENTED_REQUIREMENT` — Raw results populated by instrument file or Empower.** Another laboratory instruction specifies setting **Uses Instrument = T** on raw result components receiving instrument data, and **Has Attributes = T** where attributes are returned from the file or Empower. This is conditional on the data source and result design. **See:** [§6](#6-data-import-and-parsing), [§8](#8-empower-integration).

**[ANA-EMPOWER-01] — `DOCUMENTED_REQUIREMENT` with `DOCUMENTED_DESCRIPTION` rationale — Empower peak components.** The laboratory guidance directs a **Text** data type even for numeric-looking Empower peak results because a missing peak may be returned as `ND` rather than `0`, and describes a type mismatch otherwise. This is an Empower-specific configuration instruction, **not** a general rule for all numeric analysis components.

### 3.5 Aliquoting and dynamically assigned testing

**[ANA-ALIQUOT-01] — `DOCUMENTED_DESCRIPTION` — Media requiring aliquots.** The laboratory guidance describes an auto-aliquoting workflow in which `VWF_SLTC_TESTING_BATCH_CREATE` checks `X_OCD_MONITORING.NUM_ALIQUOT` for the method/media when a sample is added to a batch, assigns additional testing, and automatically aliquots to child samples. Another laboratory instruction also requires a separate method aliquot variation when aliquoting is needed. This is a described laboratory workflow, not verified code behavior or a universal rule for all sample media. **See:** [§2](#2-ois-integration-project-and-sample-setup), [§4](#4-batch-configuration), [§9](#9-operational-workflows).

### 3.6 Analysis-side calculation and reporting dependencies (references only)

**[ANA-CALC-REF] — `DOCUMENTED_DESCRIPTION` — Analysis-side calculation dependency.** Calculation settings differ by testing mode and must be evaluated before configuring final-result components. The authoritative documented `AutoCalculate` guidance and unresolved execution-order behavior are in [§7.3](#73-calculation-execution-and-data-source-distinctions) and [Q06](#q06-calculation-mode).

**[ANA-LIMIT-REF] — `DOCUMENTED_DESCRIPTION` — Analysis limits.** The laboratory guidance states that Analysis Limits are **not checked inherently** in the described configuration and must be applied during result calculations; reporting-limit propagation is discussed separately. Detailed rules, `CALC_ICP_FINAL`, and formatting belong to [§7](#7-calculations-limits-and-reporting).

### 3.7 Evidence limits and verification needs

- **Method-table identity:** The laboratory guidance uses `ANALYSIS_METHOD` and `ANALYSIS_METHODS` and distinguishes them from `T_ANALYSIS_METHOD`; confirm physical names and relationships before SQL or automation. [Q03](#q03-methodvariation-relationships).
- **Component flag precedence:** Variation-level requiredness, base-level reportability, and their effect on display and external reporting require real configuration and output evidence. [Q02](#q02-component-requiredness), [Q07](#q07-reportability-semantics).
- **Destructive and aliquot effects:** The actual sample-login restrictions and `VWF_SLTC_TESTING_BATCH_CREATE` behavior require code and reproducible workflow evidence.
- **Laboratory-specific conventions:** The examples for `ICP_OES`, `GC`, and named OSHA methods are not evidence of correct configuration for unrelated analyses; see [§11](#11-established-configuration-examples).
- **No vendor/runtime verification:** Nothing in §3 is `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED` at this stage.

## 4. Batch Configuration

**Scope:** Laboratory-documented configuration of Batch Test Templates, links to analyses and tests, batch protocols, QC and stock references, batch result components, and method compatibility. This section distinguishes manual-entry, instrument-file, and Empower workflows. The detailed parser interface and file-processing sequence belong to [§6](#6-data-import-and-parsing); instrument master records to [§5](#5-instrument-configuration); result calculations and limits to [§7](#7-calculations-limits-and-reporting); operational review to [§9](#9-operational-workflows). **Verification status:** Laboratory-documented only; not independently tested.

### 4.1 Batch template identity and core settings

**[BAT-TEMPLATE-01] — `DOCUMENTED_REQUIREMENT` — Normal method-oriented Test Batch configuration.** One documented instruction requires a **Batch Test Template Name** exactly matching the OSHA method for which it is designed (example `OSHA_1023`), **Assign To Objects = True**, and an **Analysis Link Key** matching the analysis name. a separate documented instruction describes batch configuration at the `T_ANALYSIS_METHOD` level, a **Name Prefix** using the OSHA method name, **Object Class = TEST**, and **Batch Type = Test Batch**. The distinction between the exact template-name requirement and the separate name-prefix setting is preserved; do not assume they are the same field. **Related:** [§1.2](#12-batch-membership-and-analysis-link-identity), [§3.1](#31-analysis-record-and-conditional-settings).

**[BAT-TEMPLATE-02] — `RECOMMENDATION` — Creating a new method template.** The laboratory guidance recommends copying an existing Batch Test Template and renaming it for the new method, because the existing template may contain preconfigured QC-stock selection, a batch-review window, and illustrative batch-result components. This is a laboratory recommendation, **not** proof that any copied settings are appropriate for a different analysis; review all copied values.

**[BAT-LINK-01] — `DOCUMENTED_REQUIREMENT` — Analyses assigned to a test batch.** One documented instruction requires a new Batch Link record for each analysis to be batched and matching analysis-side Batch Link / batch-side Analysis Link Key values. another documented instruction identifies the batch link type as **Analysis Link Key** and says it matches `ANALYSIS.NAME`; tests must belong to an analysis matching the batch link. Exact field and lookup behavior has not been independently checked. **Related:** [§1.2](#12-batch-membership-and-analysis-link-identity).

### 4.2 Test membership and compatible methods

**[BAT-MEMBER-01] — `DOCUMENTED_DESCRIPTION` — Documented test batches.** The laboratory guidance states that samples are linked to batches through their **tests**, not directly: `TEST.BATCH` contains the batch name, and `BATCH_OBJECTS.OBJECT_ID = TEST.TEST_NUMBER`. Do not infer this model applies to every LabWare batch class.

**[BAT-METHOD-01] — `DOCUMENTED_DESCRIPTION` / `IMPLEMENTATION_EXAMPLE` — Compatible-method exceptions.** The laboratory guidance says a template ordinarily covers one analytical method but some compatible methods may be tested together. In its GC example, the child `OSHA_111` method maps to the `OSHA_5000_PASSIVE` variation (whose method is `OSHA_5000`), and `T_ANALYSIS_METHOD.X_RELATED_METHOD` on `OSHA_111` points to `OSHA_5000`. The analysis method/variation mapping must also be configured. This example does **not** establish a general compatibility rule. **See:** [§3.2](#32-master-methods-analysis-methods-and-variations), [Q03](#q03-methodvariation-relationships).

### 4.3 Batch protocols, standards, and QC stock

**[BAT-PROTOCOL-01] — `DOCUMENTED_REQUIREMENT` with conditional applicability — Batches requiring standard/QC samples.** The laboratory guidance says the **Batch Protocol** can add QC samples automatically when a batch is created. The laboratory guidance describes a **Batch Standard Protocol** that adds configured standard sets, whose records are held in `QC_SAMPLES`. The laboratory guidance lists typical examples: calibration blank (`CB`), standards (`STD_A`, `STD_B`, etc.), initial calibration verification (`ICV`), and continuing calibration verification (`CCV`). The examples are not a universal required QC composition.

**[BAT-QC-VAR-01] — `DOCUMENTED_REQUIREMENT` — Templates using QC variations.** The laboratory guidance directs the **Batch Variation** in the batch configuration submenu to point to the QC method variation (example `OSHA_1023_QC`); the laboratory guidance also specifies that the configuration window's **Variation** points to the method's QC analysis variation. Create the corresponding analysis variation when the batch uses QC samples.

**[BAT-QCSM-01] — `DOCUMENTED_REQUIREMENT` — Templates using QCSM stock.** One documented instruction says **X_QC_STOCK** on the **parent batch configuration page**, not the submenu opened by Configure, should point to allowed QCSM stock entries; this controls which QCSM stock may be added. another documented instruction directs configuring and attaching a **QCSM Stock** record under **QC Stock**. The exact equivalence or interaction of these UI labels has not been verified.

**[BAT-RLSM-01] — `DOCUMENTED_REQUIREMENT` — Batch templates requiring reporting-limit stock per the laboratory guidance.** Another laboratory instruction specifies configuring an **RLSM Stock** record and attaching it as **RL Stock** on the batch template. other available laboratory guidance does not address this field; do not infer its necessity for every possible batch type without examining the analysis workflow.

**[BAT-CCV-01] — `DOCUMENTED_DESCRIPTION` — Repeated CCV runs in the documented OSHA batch workflow.** The laboratory guidance describes CCVs as potentially recurring (typically once per ten field samples). It directs keeping **one batch sample per QC type** and adding additional **analysis replicates** to that sample when multiple CCV runs are required, then ordering those tests appropriately. This is laboratory-specific guidance, not a universal LabWare restriction or a verified scheduling algorithm. **Related:** [§6](#6-data-import-and-parsing) for file-driven replication and reordering.

### 4.4 Batch result components and reference fields

**[BAT-COMP-01] — `DOCUMENTED_REQUIREMENT` — Batch result components referring to stock.** The laboratory guidance directs filling **Other Fields → X_STOCK** with the appropriate stock item for stock-referencing results. Another laboratory instruction specifies `BATCH_COMPONENT.X_STOCK = STOCK.NAME` for **Stock Used** and says **Standards Used** is configured similarly. For standard components, another laboratory instruction additionally states `BATCH_COMPONENT.ALIAS_NAME = QC_SAMPLES.NAME` to match the QC samples in the batch standard protocol.

**[BAT-COMP-02] — `DOCUMENTED_REQUIREMENT` — Instrument-identifying batch results.** Configure **Instrument Used** batch components (including additional components for multiple instruments when applicable) using the documented group-matching convention. The precise Alias Name / instrument-group relationship and conflicting `X_STOCK` reference are maintained in [§5.1](#51-instrument-group-and-analysisbatch-matching) and [Q01](#q01-instrument-reference-fields).

**[BAT-COMP-03] — `DOCUMENTED_REQUIREMENT` — Instrument-file imports only.** The laboratory guidance directs that the batch result identifying the instrument producing the parsed file have **Other Fields → Data File Source = True**, with **only one Data File Source per batch**. It also requires an **Instrument File** batch result with type **DB File Name**. Another laboratory instruction likewise identifies **Instrument File** among the batch results needed for processing. Do not apply these requirements automatically to manual-entry or Empower-only batches.

**[BAT-COMP-04] — `DOCUMENTED_REQUIREMENT` (auto-calc setting) with `IMPLEMENTATION_EXAMPLE` (routine snippets) — Batch component entry calculations.** The laboratory guidance gives illustrative **Instrument Used** calculation logic (`SELECT BATCH_COMPONENT.ALIAS_NAME` into `instGroupName`; `GOSUB BC_INSTRUMENT_BROWSE`; `RETURN instrumentName`), **Stock Used** / **Standards Used** logic (`GOSUB CALC_ING_SCAN_DIALOG`; `RETURN`), and directs **Auto-calc false** for these batch component calculations. These are documented configurations, not independently validated code snippets or a complete executable routine. **See:** [§5](#5-instrument-configuration) and [§7](#7-calculations-limits-and-reporting).

**[BAT-PRECISION-01] — `DOCUMENTED_REQUIREMENT` with `IMPLEMENTATION_EXAMPLE` — QCSM precision results where configured.** The laboratory guidance calls for **one QCSM Precision record per reportable recovery (F/T)** on the batched analysis, with **Auto-calc false**, `targetElement` set to the applicable recovery component, and `GOSUB CALC_BATCH_QC_REC_PRECISION`. The example target is `Be F/T`; it must not be copied unchanged to unrelated analyses. The precise meaning of “F/T” and runtime calculation behavior have not been independently established.

### 4.5 Pre-analysis results and entry-mode differences

**[BAT-PREREQ-01] — `DOCUMENTED_REQUIREMENT` — Instrument-file import; also described for Empower in the laboratory guidance.** One documented instruction states that batch results with **Optional = F** must be entered before a data file can be imported. another documented instruction states all **non-optional batch results** must be entered before file parsing or Empower testing proceeds, and its batch-template details refer to instrument/stock results marked **Required = T** as blocking parsing until populated. These descriptions may reflect different UI terms or contexts; **do not assume field equivalence** until verified. For `OSHA_1023`, the laboratory guidance gives the example that lab solutions and instruments other than the balance must be entered before file processing. **See:** [Q02](#q02-component-requiredness).

**[BAT-MANUAL-01] — `DOCUMENTED_DESCRIPTION` — Manual batch result entry.** The laboratory guidance states that **manual-entry batches do not perform batch-level validation of the Batch Results table** and that the analyst and reviewer must ensure required entries are present. This is a documented laboratory workflow description, not proof of all possible manual-entry validation behavior. **See:** [§9](#9-operational-workflows), [Q09](#q09-manual-batch-checks).

**[BAT-FILE-01] — `DOCUMENTED_DESCRIPTION` — File-driven batches.** The laboratory guidance describes a manually initiated **Batch → Batch Process Results** routine, even though automatic file imports are possible, because the OSHA import workflow requires complex validation and data-dependent logic. The routine checks the **Instrument File** result before importing and calls an analysis-specific parser. This entry establishes the batch-side boundary only; routine dispatch, validation-only and commit loops, file lifecycle, and returned variables are owned by [§6](#6-data-import-and-parsing).

**[BAT-EMPOWER-01] — `DOCUMENTED_DESCRIPTION` — Empower batches.** The laboratory guidance describes instrument/project consistency checks and additional configuration in both LabWare and Empower. The batch's instrument selection participates in the project check; no inference is made that an Empower-only batch requires the file-parser's **Data File Source** or **Instrument File** configuration.

### 4.6 Evidence limitations and verification priorities

- **Template and link fields:** Confirm template name versus name prefix, **Assign To Objects**, Batch Link/Analysis Link Key records, and physical lookup behavior from configuration exports. See [§1.2](#12-batch-membership-and-analysis-link-identity).
- **Instrument reference ambiguity:** Inspect actual batch components to distinguish **Alias Name**, `X_STOCK`, instrument group, and instrument barcode relationships. See [Q01](#q01-instrument-reference-fields).
- **Requiredness semantics:** Determine whether **Optional = F**, **Required = T**, and the documented “non-optional” wording describe one control or different contexts. See [Q02](#q02-component-requiredness).
- **QC and stock applicability:** Confirm which methods require QCSM/RLSM stocks, the exact relationship between **X_QC_STOCK** and **QC Stock**, and how repeated CCVs and precision calculations behave.
- **Manual and automated validation:** Confirm actual behavior of manual batch checks, file import prerequisites, and Empower gating with code/configuration and reproducible tests. See [Q09](#q09-manual-batch-checks), [§6](#6-data-import-and-parsing), [§8](#8-empower-integration).
- **No vendor/runtime verification:** Nothing in §4 is yet classified `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`. Method-specific examples are not universal configuration templates.

## 5. Instrument Configuration

**Scope:** Laboratory-documented instrument master-record settings and the relationships connecting instrument groups, batch instrument results, analysis configuration, instrument-file parsing, and Empower projects. This section owns instrument-record fields; batch component settings are owned by [§4](#4-batch-configuration), parser routines and directory constants by [§6](#6-data-import-and-parsing), and Empower project/interface setup by [§8](#8-empower-integration). **Verification status:** Laboratory-documented only; not independently tested.

### 5.1 Instrument group and analysis/batch matching

**[INST-GROUP-01] — `DOCUMENTED_REQUIREMENT` — Instrument-file import or Empower.** The documented matching instruction specifies that the instrument's **Instrument Group** match the analysis **Inst Group** and the batch instrument-result identification. The batch configuration guidance describes the batch instrument-result **Alias Name** as matching the instrument group; the documented detailed batch instructions express this as `BATCH_COMPONENT.ALIAS_NAME = INSTRUMENT.INST_GROUP`. This is the documented group-matching convention for the described workflows, not a claim that all analyses need an instrument group. **See:** [§3.1](#31-analysis-record-and-conditional-settings), [§4.4](#44-batch-result-components-and-reference-fields).

**[INST-REF-01] — `UNRESOLVED` — Instrument batch-result reference.** A separate documented instruction says the **Batch Result X_STOCK** field should match the instrument group when connecting an instrument to a batch. This differs from the Alias Name matching rule described in the batch configuration instructions. Preserve both statements; **do not infer** whether `X_STOCK` is also required, represents a different object, or is an error. **Tracking:** [Q01](#q01-instrument-reference-fields).

### 5.2 Instrument Parameter 1: laboratory instrument identifier

**[INST-PARAM1-01] — `DOCUMENTED_REQUIREMENT` with conditional use — Instrument-specific parsing.** The documented configuration describes setting **Instrument Parameter 1 Name = `NAME`** and **Parameter 1 Value = `{instrumentName}`** when using instrument-file parsing without a LabStation script; its example value is `HOOVER`. A separate instruction also specifies that **Instrument Params → Parameter 1 Name** be `NAME` and **Parameter 1 Value** be the laboratory name for the instrument, noting that it may support instrument-specific configuration. These are laboratory conventions, not vendor-wide parameter semantics.

**[INST-PARAM1-02] — `DOCUMENTED_DESCRIPTION` — Parser naming dependency.** The laboratory guidance specifies the instrument-specific subroutine pattern `BM_SLTC_DATA_PARSE_{analysisName}_{instrumentName}` when **not** using a LabStation parsing script, and states that the instrument-name suffix matches the instrument's Parameter 1 value. Examples `BM_SLTC_DATA_PARSE_ICP_OES_HOOVER` and `BM_SLTC_DATA_PARSE_ICP_OES_MILHOUSE` are `IMPLEMENTATION_EXAMPLE`s, not prescribed routines for other instruments. The exact selection/dispatch logic is not shown in the laboratory guidance.

### 5.3 LabStation script versus instrument-specific parser

**[INST-PARSE-01] — `DOCUMENTED_DESCRIPTION` — Instrument-generated files requiring parsing.** The laboratory guidance describes two alternative instrument-side configurations: a **LabStation parsing script**, or **Instrument Parameter 1** used with an instrument-specific parsing subroutine. It does not document LabStation script setup, precedence when both approaches exist, or a complete runtime selection algorithm. One laboratory instruction also states that subroutine configuration may be skipped when no data file is parsed. Do not impose instrument-file parser requirements on manual-only or Empower-only workflows. **See:** [§6](#6-data-import-and-parsing), [Q04](#q04-parser-dispatch).

### 5.4 Instrument Parameter 2 and Empower project association

**[INST-PARAM2-01] — `DOCUMENTED_DESCRIPTION` — Analysis-specific instrument settings.** The laboratory guidance states that **Parameter 2 Name/Value** are available for analysis-specific configuration, mentioning ICP-OES and GC as examples. No general value, format, or processing rule for Parameter 2 is provided for all analyses.

**[INST-EMPOWER-01] — `DOCUMENTED_REQUIREMENT` — Documented OSHA Empower workflow.** The documented OSHA Empower workflow describes one project per instrument for the OSHA use case. It directs setting `INSTRUMENTS.PARAMETER_2_NAME` to **`EMPOWER PROJECT`** and `INSTRUMENTS.PARAMETER_2_VALUE` to the **full Empower project name**; its example is `GC_Default\GC4 – LabWare1`. During batch processing, the selected batch instrument is compared with the project loaded in the Empower Interface; the laboratory guidance states a mismatch prompts the user to switch projects. This is a documented description of laboratory behavior, not an independently verified interface contract.  **Empower configuration owner:** [§8](#8-empower-integration).

### 5.5 Instrument setup dependency checklist (guidance-derived)

The following is a **planning aid**, not a verified UI sequence or a complete instrument qualification procedure:

1. **When using file import or Empower:** establish the instrument group and ensure the relevant analysis and batch instrument-result references follow the documented group-matching convention. See [§3.1](#31-analysis-record-and-conditional-settings), [§4.4](#44-batch-result-components-and-reference-fields), and [Q01](#q01-instrument-reference-fields).
2. **When using instrument-specific file parsing rather than LabStation:** establish the Parameter 1 `NAME` value and matching instrument-specific parser name. See [§6](#6-data-import-and-parsing) and [Q04](#q04-parser-dispatch).
3. **When using the documented Empower integration:** establish the expected project in Parameter 2 and complete project-specific Empower interface configuration. See [§8](#8-empower-integration).
4. **When no instrument file is parsed:** do not assume file-parser routines, path constants, or a Data File Source component are necessary merely because an instrument is used. See [§4.5](#45-pre-analysis-results-and-entry-mode-differences).

### 5.6 Evidence limitations and verification priorities

- **Instrument-reference ambiguity:** Confirm how `INSTRUMENT.INST_GROUP` / `INSTRUMENTS` group settings, `BATCH_COMPONENT.ALIAS_NAME`, and batch-result `X_STOCK` interact using actual configuration records. [Q01](#q01-instrument-reference-fields).
- **Instrument identity:** Confirm physical field names and whether Parameter 1 values must be unique or satisfy particular case/character rules; the available guidance does not establish these constraints.
- **Parser dispatch:** Inspect the relevant parsing routines and LabStation configuration before treating the documented naming convention as proven runtime behavior. [Q04](#q04-parser-dispatch).
- **Empower project validation:** Verify the exact project comparison and mismatch handling with instrument records, interface configuration, and a reproducible test. See [§8](#8-empower-integration), [G04](#g04-empower-exception-behavior).
- **No vendor/runtime verification:** Nothing in §5 is classified `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`; instrument names and project paths in the documented examples are implementation examples.

## 6. Data Import and Parsing

**Scope:** Laboratory-documented import of instrument-generated result files into batches: file location, batch prerequisites, parser invocation, returned data and context variables, validation, batch ordering, and failure controls. This is **not** the inbound OIS request XML pipeline ([§2](#2-ois-integration-project-and-sample-setup)) or Empower interface processing ([§8](#8-empower-integration)). Instrument record settings belong to [§5](#5-instrument-configuration); batch result-component settings to [§4](#4-batch-configuration); final-result calculation sequencing to [§7](#7-calculations-limits-and-reporting). **Verification status:** Laboratory-documented only; not independently tested.

### 6.1 File location and batch prerequisites

**[IMP-PATH-01] — `DOCUMENTED_REQUIREMENT` — Instrument-file imports in the documented SLTC implementation.** One laboratory instruction specifies configuring a LIMS constant named `SLTC_DATA_IMPORT_{analysisName}` for each analysis, with a full directory path and file extension containing `*` as the filename wildcard (example `D:\OIS Integration\Inbound ICP Results\*.csv`). The documented implementation uses at **analysis-level granularity**, while noting that a different granularity may be possible; do not assume instrument-level or method-level overrides are implemented. **Open:** [Q08](#q08-path-constant-granularity).

**[IMP-PREREQ-01] — `DOCUMENTED_REQUIREMENT` — File-import batches.** The laboratory guidance requires the batch component identifying the file-generating instrument to have **Data File Source = True** and says there should be **only one** such component per batch; it also requires an **Instrument File** result of type **DB File Name**. The documented batch instructions state batch results marked **Optional = F** must be entered before import; another laboratory instruction uses **Required = T** and “non-optional” terminology for related gating. These labels are not assumed equivalent.

**[IMP-TRANSFER-01] — `DOCUMENTED_DESCRIPTION` — Transport versus parsing.** The laboratory guidance describes instrument files arriving through SFTP via Windows shell scripts executed by a LabWare background instance, while the batch result-import process is initiated separately. The scripts, file locations, and transfer configuration are not specified. **See:** [§2.1](#21-end-to-end-ois-context-and-intake-boundary).

### 6.2 Import entry point and parser dispatch

**[IMP-ENTRY-01] — `DOCUMENTED_DESCRIPTION` — Manually initiated file imports.** The laboratory guidance describes **Batch → Batch Process Results** (`MNU_BM_SLTC_DATA_PARSE`): the routine identifies the instrument directory from the assigned batch instrument, presents a file selection, validates the batch and file, and prompts for confirmation before import. The laboratory guidance states manual initiation was preferred for the OSHA workflow even though automatic import is possible. This is documented behavior, not a verified UI or execution trace.

**[IMP-DISPATCH-01] — `DOCUMENTED_REQUIREMENT` / `DOCUMENTED_DESCRIPTION` — Analysis-specific file parser.** The analysis parser naming instruction specifies `BM_SLTC_DATA_PARSE_{analysisName}`. The documented workflow states the menu routine constructs/calls this name dynamically and errors if the subroutine does not exist. The instrument-specific parsing instruction additionally specifies `BM_SLTC_DATA_PARSE_{analysisName}_{instrumentName}` for instrument-specific parsing **when a LabStation parsing script is not used**, with the suffix matching instrument Parameter 1 value. The laboratory guidance does **not** show the full instrument-specific dispatch implementation or precedence. **Instrument configuration:** [§5.2](#52-instrument-parameter-1-laboratory-instrument-identifier), [§5.3](#53-labstation-script-versus-instrument-specific-parser); **open:** [Q04](#q04-parser-dispatch).

**[IMP-DUPLICATE-01] — `DOCUMENTED_DESCRIPTION` — Reimport guard.** The laboratory guidance says the menu routine checks whether the batch **Instrument File** result has already been entered and, if so, assumes that data were imported and does not proceed with the normal import path. This is a documented safeguard, **not proof of idempotency** or a complete recovery/override procedure. **Open:** [G02](#g02-parser-failure-and-recovery).

### 6.3 Analysis parser result contract

**[IMP-ARRAY-01] — `DOCUMENTED_REQUIREMENT` — Analysis-specific file parser.** The documented parser interface describes returning `resultDataArr` as a **headerless two-dimensional array** with **one row per result** and ten columns in this order. The laboratory guidance states unused attribute positions may be blank but must remain present.

| Column | Value |
|---|---|
| 1 | Sample Number |
| 2 | Test Number |
| 3 | Result Number |
| 4 | Result Name |
| 5 | Result Entry |
| 6 | Result Attribute 1 |
| 7 | Result Attribute 2 |
| 8 | Result Attribute 3 |
| 9 | Result Attribute 4 |
| 10 | Result Attribute 5 |

**[IMP-RETURNS-01] — `DOCUMENTED_DESCRIPTION` — Additional parser outputs.** One parser description treats `reorderBatch` as optional and identifies `batchOrderArr` when reordering is requested. Another description lists `stopParse`, `validationStr`, and `wrongBatch` as additional context outputs. These are **not all established as unconditional mandatory returns**. The table records their documented roles, not verified variable initialization or consumption semantics.

| Variable | Documented role / condition |
|---|---|
| `reorderBatch` | Boolean; if true, batch objects are reordered using `batchOrderArr`. It is described as optional. |
| `batchOrderArr` | Two-dimensional array required **if `reorderBatch = TRUE`**; column 1 contains batch object IDs (test numbers) in desired order. |
| `stopParse` | Boolean; if true, the laboratory guidance states parsing stops and the file is returned to the instrument directory. |
| `validationStr` | Text describing why processing cannot proceed; the laboratory guidance states it should be written during the validation-only loop. |
| `wrongBatch` | Boolean used for automatic processing; if true, the laboratory guidance states it stops processing similarly to `stopParse`. |

**[IMP-CONTEXT-01] — `DOCUMENTED_DESCRIPTION` — Values supplied to the called analysis parser.** The following context variables are described as supplied by `MNU_BM_SLTC_DATA_PARSE`. Their types and meanings are documented, but the actual call signature is unverified.

| Variable | Documented description |
|---|---|
| `instrumentBarcode` | Primary key of the batch-file instrument in `INSTRUMENTS`. |
| `Batch` | Batch name (primary key). |
| `batchSampsArr` | One-dimensional array of batch sample numbers. |
| `directory` | Instrument-data directory path. |
| `directoryProcess` | Processing-directory path under the instrument folder. |
| `fileArr` | Two-dimensional array of filenames in the instrument-data directory. |
| `filename` | Full path and name of the file to parse. |
| `Method` | Method associated with the batch. |
| `Analysis` | Analysis associated with the batch. |

**[IMP-CONTRACT-OPEN] — `UNRESOLVED` — Parser interface details.** The shorter and longer documented parser-output descriptions are not assumed contradictory or fully reconciled. Which variables are mandatory, their precise scope, initialization, and return/side-effect handling must be checked against `MNU_BM_SLTC_DATA_PARSE` and representative parsers. **Open:** [Q05](#q05-parser-contract).

### 6.4 Validation-only and commit processing

**[IMP-VALIDATE-01] — `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT` — File-import validation design.** The laboratory guidance describes **two loops**: a first **`validateOnly = TRUE`** pass that checks the file and gives the user actionable feedback, followed by a second pass that performs commits such as result replication and CCV additions. Its suggested checks include matching file sample IDs to the batch, verifying standard/control sample counts and types, and ensuring data are present and have the expected types. Another laboratory instruction specifies guarding commits with `IF` conditions during the validation-only pass and writing useful messages to `validationStr`. These are documented design instructions, **not evidence of transaction isolation or a guarantee of no writes**.

**[IMP-STOP-01] — `DOCUMENTED_DESCRIPTION` — Stop conditions and feedback.** The laboratory guidance says `stopParse = TRUE` stops parsing and returns the file to the instrument directory; `wrongBatch = TRUE` is used in automatic processing to stop similarly; `validationStr` communicates reasons a batch cannot be processed. The laboratory guidance does not specify all branches, filesystem operations, partial-commit rollback, or the effect of a failed second pass. **Open:** [G02](#g02-parser-failure-and-recovery).

### 6.5 Result replication and batch reordering

**[IMP-ORDER-01] — `DOCUMENTED_REQUIREMENT` — Only when parser requests reordering.** If `reorderBatch = TRUE`, the documented instructions require `batchOrderArr`, with its **first column** holding the batch object IDs (test numbers) in desired order. The documented `ICP_OES` example includes timestamp and sample number in additional columns, but says only column 1 is used for reordering. In that example, some tests are added after file processing, so tests are reordered by instrument timestamp.

**[IMP-REPLICATE-01] — `DOCUMENTED_DESCRIPTION` — Data-dependent testing.** The laboratory guidance says the import process may add result replicates and CCV-related entries based on file contents, and describes repeated CCV runs as additional analysis replicates on one QC sample. The exact replication and association logic is not included in the supplied documents. **Batch/QC owner:** [§4.3](#43-batch-protocols-standards-and-qc-stock).

### 6.6 File lifecycle, failure boundaries, and evidence needs

- **Documented paths:** the laboratory guidance identifies an instrument directory and a processing directory; one laboratory instruction specifies a wildcard file-path constant. Their exact relationship, file movement, archive behavior, cleanup, and retry mechanism are **not established**. [Q08](#q08-path-constant-granularity), [G02](#g02-parser-failure-and-recovery).
- **Duplicate and partial imports:** The **Instrument File** check is described, but no source establishes safe repeat import, rollback, or recovery after partial results have been committed. [G02](#g02-parser-failure-and-recovery).
- **Dispatch and output semantics:** Confirm parser naming, LabStation interactions, context variables, and reordering against actual routines. [Q04](#q04-parser-dispatch), [Q05](#q05-parser-contract).
- **Calculation ordering:** File-import calculation-mode requirements and blank/unknown dependencies are owned by [§7.3](#73-calculation-execution-and-data-source-distinctions); the execution sequence remains unverified ([Q06](#q06-calculation-mode)).
- **No vendor/runtime verification:** Nothing in §6 is classified `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`. `ICP_OES`, instrument names, paths, and code identifiers supplied as examples are not universal configuration defaults.

## 7. Calculations, Limits, and Reporting

**Scope:** Laboratory-documented use of analysis limits and reporting limits, calculation execution settings, recovery results, and final-result selection for outbound OIS reporting. This section owns result-level calculation and reporting rules; component and variation setup belongs to [§3](#3-analysis-configuration), batch QC and precision components to [§4](#4-batch-configuration), parser execution to [§6](#6-data-import-and-parsing), and operational release/response generation to [§9](#9-operational-workflows). **Verification status:** Laboratory-documented only; not independently tested.

### 7.1 Analysis limits and enforcement boundary

**[CAL-LIMIT-01] — `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT` — Analyses using configured analysis limits.** The documented implementation states that **Analysis Limits are not checked inherently** in the described implementation. Limits may be configured in the Analysis Limits table, but must be explicitly applied during result calculations. The laboratory guidance points to `CALC_ICP_FINAL` as an **implementation example** of accessing those limits; the available guidance neither provides its code nor establishes the calculation engine's general enforcement behavior. **See:** [§3.6](#36-analysis-side-calculation-and-reporting-dependencies-references-only).

### 7.2 Lower reporting limit and final-result propagation

**[CAL-RL-01] — `DOCUMENTED_REQUIREMENT` — Documented raw-to-final result workflow.** The documented workflow states that each raw instrument result has a corresponding **Lower Reporting Limit**, determined annually from measured **Reporting Limit Spiked Media (RLSM)** samples through a separate process. The limit is stored in `ANALYSIS_LIMITS.MIN_LIMIT`. Another laboratory instruction specifies using that value to calculate a reporting limit for final results and storing the resulting value in `RESULT.X_REPORTING_LIMIT`. The annual RLSM determination procedure, selection criteria for the relevant limit row, and underlying limit records are **not supplied**.

**[CAL-RL-02] — `DOCUMENTED_REQUIREMENT` — Final results transformed from raw instrument values.** The laboratory guidance says that, **generally**, transformations applied to the raw measured result to obtain the final result should also be applied to the Lower Reporting Limit in parallel. The laboratory guidance does not specify a universal formula, unit-conversion sequence, or list of permitted transformations. Do not infer that a raw limit can simply be copied unchanged to every final result.

**[CAL-RL-03] — `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT` — Final-result reporting-limit formatting.** The laboratory guidance describes a format calculation, usually `ICP_LOWER_LIMIT`, that compares final-result `RESULT.ENTRY` with `RESULT.X_REPORTING_LIMIT` and, where the result is below the reportable threshold, produces a lower-bracket presentation described as **`<RL`**. Another laboratory instruction also calls for a format calculation *similar to* `ICP_LOWER_LIMIT` for most final calculated results. The laboratory guidance does not supply the code, exact equality/boundary rule, rounding, formatting expression, or whether `RESULT.ENTRY` itself is overwritten. Treat the routine name as an example, not a requirement to reuse the same calculation for every method. **Open:** [G03](#g03-release-and-ois-response).

### 7.3 Calculation execution and data-source distinctions

**[CAL-AUTO-01] — `DOCUMENTED_REQUIREMENT` — Final-result calculations for instrument-file batch imports.** Both laboratory descriptions instruct **`AutoCalculate = F`** for final-result calculations when batch files are imported. Their stated reason is that blank samples and field unknowns may arrive or be tested out of order while calculations depend on one another. This is a laboratory sequencing rule for the documented file-import workflow, **not** evidence of a general LabWare engine limitation or an instruction to disable every calculation. **See:** [§6](#6-data-import-and-parsing), [Q06](#q06-calculation-mode).

**[CAL-AUTO-02] — `DOCUMENTED_DESCRIPTION` — Manual-entry or Empower workflows.** The laboratory guidance says calculations **may** use **`AutoCalculate = T`** for manual entry or Empower. This is permission conditioned on workflow and dependencies, **not** a universal requirement to enable automatic calculation. The laboratory guidance does not provide a complete execution-order specification for either mode. **See:** [§8](#8-empower-integration), [Q06](#q06-calculation-mode).

**[CAL-SEQUENCE-OPEN] — `UNRESOLVED` — Final calculation orchestration.** The available guidance does not supply the full execution sequence for blank correction, replicates, reporting-limit transformations, and final-result calculation after import. In particular, `AutoCalculate = F` does not itself document what routine triggers final calculations or how failed calculations are recovered. Verify with calculation code, batch processing routines, and reproducible tests before prescribing execution order. **See:** [Q06](#q06-calculation-mode), [§6.4](#64-validation-only-and-commit-processing).

### 7.4 Recovery and batch QC calculation boundaries

**[CAL-RECOVERY-01] — `DOCUMENTED_DESCRIPTION` — Standards, controls, QCSM, and RLSM.** Recovery is defined as the measured analyte amount relative to the theoretical content and described as a **unitless fraction**. Its illustration is 99 mg measured from a nominal 100 mg, giving a recovery of **0.99**. This is a conceptual and numerical example, not a supplied LabWare calculation formula or acceptance criterion for all recovery components.

**[CAL-QC-01] — `DOCUMENTED_REQUIREMENT` with `IMPLEMENTATION_EXAMPLE` — QCSM precision components when configured.** The laboratory guidance directs one **QCSM Precision** batch result per reportable recovery (F/T), with `Auto-calc false`, a `targetElement` naming the applicable recovery component, and `GOSUB CALC_BATCH_QC_REC_PRECISION`. Its example target `Be F/T` must be replaced as appropriate for the analysis. The meaning of “F/T,” calculation formula, and QC acceptance limits are not documented. Batch-component setup is owned by [§4.4](#44-batch-result-components-and-reference-fields); this entry records only the calculation dependency.

### 7.5 Reportable components, analyte codes, and OIS output

**[RPT-FLAG-01] — `DOCUMENTED_DESCRIPTION` / `DOCUMENTED_REQUIREMENT` — Outbound result selection.** The laboratory guidance describes OIS response XML and CSHO reports as including reportable field-sample components, and directs that only **final results** be designated reportable. The component-level flag settings and the differing documented instructions are addressed by [§3.3](#33-base-components-versus-variation-components); their effective precedence and output behavior remain unverified ([Q07](#q07-reportability-semantics)).

**[RPT-ALIAS-01] — `DOCUMENTED_REQUIREMENT` — Reportable components mapped to OIS analytes.** The laboratory guidance instructs that a reportable component's **Alias Name** match the corresponding **OIS Analyte Code**. The documented example is **Beryllium (Final)** for `OSHA_1023` with alias **`0365`**. The code must be treated as an identifier, not a number; this data-handling recommendation does not establish additional LabWare validation behavior.

**[RPT-OUTPUT-OPEN] — `UNRESOLVED` — Release and outbound response mechanics.** The laboratory guidance describes the overall purpose of generating a response file for OIS and reporting to CSHOs, but the *Batch Review* and *Project Closure and OIS Reporting* headings do not provide the full review, release, XML construction, delivery, or failure-handling procedure. The component flags and aliases above do **not** establish those steps. **See:** [§9](#9-operational-workflows), [G03](#g03-release-and-ois-response).

### 7.6 Evidence limitations and verification priorities

- **Limits and formulas:** Obtain representative `ANALYSIS_LIMITS` records, `CALC_ICP_FINAL`, `ICP_LOWER_LIMIT`, and example raw/final results to verify limit selection, propagation, formatting, units, rounding, and comparison boundaries.
- **Execution sequence:** Examine actual final-result calculations and batch-processing code, then test blank/unknown dependencies and repeated calculations for each relevant import mode. [Q06](#q06-calculation-mode).
- **Recovery and QC:** Obtain recovery and QCSM precision calculations, target-component records, and applicable laboratory acceptance criteria before asserting pass/fail logic.
- **Reportability:** Compare base/variation component flags with review screens, exported OIS XML, and emailed reports to establish effective precedence and output behavior. [Q07](#q07-reportability-semantics), [G03](#g03-release-and-ois-response).
- **No vendor/runtime verification:** Nothing in §7 is classified `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`. Named calculations, method IDs, and analyte codes are source examples, not universal defaults.

## 8. Empower Integration

**Scope:** Laboratory-documented setup of Empower projects and their LabWare interface: instrument-to-project association, required Empower custom fields, sample-set and result mappings, project mapping reuse, and Empower-specific result types. This section is based primarily on the laboratory guidance and describes the **OSHA implementation**, not a universal LabWare–Empower contract. Instrument master-record settings belong to [§5](#5-instrument-configuration); analysis component configuration to [§3](#3-analysis-configuration); calculation sequencing to [§7](#7-calculations-limits-and-reporting); batch prerequisites to [§4](#4-batch-configuration). **Verification status:** Laboratory-documented only; not independently tested.

### 8.1 Instrument association and project validation

**[EMP-PROJECT-01] — `DOCUMENTED_REQUIREMENT` (Parameter 2 fields) with `DOCUMENTED_DESCRIPTION` (project-check behavior) — Documented OSHA Empower batches.** The laboratory guidance describes configuring each instrument for **one Empower project** in this use case. Set `INSTRUMENTS.PARAMETER_2_NAME` to `EMPOWER PROJECT` and `INSTRUMENTS.PARAMETER_2_VALUE` to the full project name. Its example `GC_Default\GC4 – LabWare1` is an `IMPLEMENTATION_EXAMPLE`, not a default. The laboratory guidance states that batch processing compares the instrument selected on the Batch Result tab with the project loaded in the Empower Interface and prompts the user to switch projects on a mismatch. The comparison and prompt have not been independently verified. **Instrument field owner:** [§5.4](#54-instrument-parameter-2-and-empower-project-association); **open:** [G04](#g04-empower-exception-behavior).

### 8.2 Empower project custom fields

**[EMP-FIELDS-01] — `DOCUMENTED_REQUIREMENT` — New Empower projects transferred to LabWare.** Another laboratory instruction specifies adding four custom fields to each new Empower project so LabWare can identify the relevant records. In Empower, open **Project Configuration Manager**, select **Configure the System**, open the target project's **Properties**, then the **Custom Fields** tab. The field names, scopes, types, and widths below reproduce the laboratory guidance; do not change capitalization or infer additional constraints.

| Custom field | Empower field scope | Data type | Width |
|---|---|---|---|
| `LimsSampleId` | Sample | Text | 20 |
| `LimsTestId` | Sample | Text | 20 |
| `PostedToLIMS` | Result | Text | 5 |
| `ReviewedInLIMS` | Result | Text | 20 |

**[EMP-FIELDS-OPEN] — `UNRESOLVED` — Custom-field runtime semantics.** The laboratory guidance provides the required field definitions but does not explain which routines write `PostedToLIMS` or `ReviewedInLIMS`, which values they contain, or how missing/duplicate identifiers are handled. The widths are source-prescribed settings, **not** evidence that every identifier or value has been validated to fit. **Open:** [G04](#g04-empower-exception-behavior).

### 8.3 LabWare Empower Interface mapping setup

**[EMP-MAP-SETUP-01] — `DOCUMENTED_REQUIREMENT` with `IMPLEMENTATION_EXAMPLE` — Onboarding a new Empower project.** The laboratory guidance says new project mappings can be made by copying an existing configured project mapping using **Save As**, **after** the custom fields have been added to the target Empower project. Its documented operator sequence is: in LabWare, open **Options → Empower Interface** and log in; select the existing project used for **MeCl2 (OSHA 1025)**; open **Project → Result Mapping Manager** and **Save As** to the new project(s); open **Project → Sample Set Method Mapping Manager** and **Save As** to the same project(s); exit the Empower Interface and log out of LabWare. The MeCl2 project is a documented example of an existing mapping, **not** proof it is suitable for every new method. Review copied mappings for the target project's requirements.

**[EMP-MAP-SETUP-OPEN] — `UNRESOLVED` — Mapping reuse.** The laboratory guidance does not provide criteria for when copying the MeCl2 mapping is appropriate, how to validate field compatibility, or how to test/save/rollback a new mapping. Do not treat successful Save As as evidence of correct result transfer. **Open:** [G04](#g04-empower-exception-behavior).

### 8.4 Sample Set Method mapping

**[EMP-SAMPLE-MAP-01] — `DOCUMENTED_DESCRIPTION` — Documented mappings.** The following is the documented **Sample Set Mapping** table. It is a description of documented mapping configuration, not a verified export from the interface.

| LabWare SAMPLE or TEST field | Empower Sample Set Method field |
|---|---|
| `SAMPLE.SAMPLE_NUMBER` | `LimsSampleId` |
| `TEST.TEST_NUMBER` | `LimsTestId` |
| `SAMPLE.TEXT_ID` | `SampleName` |

### 8.5 Result Set mapping

**[EMP-RESULT-MAP-01] — `DOCUMENTED_DESCRIPTION` — Documented mappings.** The following is the documented **Result Set Mapping** table. In particular, `RESULT.MILL_RES_NUM` is reproduced **as documented**; do not silently change it to a similarly named field without schema evidence.

| LabWare SAMPLE, TEST, or RESULT field | Empower Result or Peak field |
|---|---|
| `RESULT.ATTRIBUTE_3` | `Result.CalibrationId` |
| `RESULT.ENTRY` | `Peak.Amount` |
| `RESULT.ATTRIBUTE_4` | `Peak.CurveId` |
| `RESULT.ATTRIBUTE_2` | `Result.Dilution` |
| `RESULT.NAME` | `Peak.Name` |
| `SAMPLE.SAMPLE_NUMBER` | `Result.LimsSampleId` |
| `TEST.TEST_NUMBER` | `Result.LimsTestId` |
| `RESULT.MILL_RES_NUM` | `Result.ResultNum` |
| `TEST.PARAMETER_VALUE` | `Result.SampleName` |
| `RESULT.ATTRIBUTE_5` | `Peak.Response` |

**[EMP-MAP-SCOPE-01] — `UNRESOLVED` — Mapping scope and attribute meaning.** The available guidance does not establish whether these maps apply unchanged to all Empower methods, whether every analysis uses the same attribute assignments, or what other interface settings are required. Attribute meanings must be checked against the analysis configuration and actual project mappings before copying them. **See:** [§3.4](#34-attributes-and-instrument-result-component-flags), [G04](#g04-empower-exception-behavior).

### 8.6 Empower peak values and calculation considerations

**[EMP-ND-01] — `DOCUMENTED_REQUIREMENT` with documented rationale — Empower peak components.** Another laboratory instruction specifies setting Empower peak component data type to **Text**, even when the reported value normally appears numeric, because a peak not detected by the Empower processing method may be returned as the literal string `ND` rather than `0`; the laboratory guidance states a numeric type would cause a runtime type-mismatch error. This is specific to the described Empower peak results, not a rule for all numerical results.  **Analysis component owner:** [§3.4](#34-attributes-and-instrument-result-component-flags).

**[EMP-CALC-01] — `DOCUMENTED_DESCRIPTION` — Empower calculation mode.** Empower may use a different calculation mode than file-import workflows; consult the conditional rule in [§7.3](#73-calculation-execution-and-data-source-distinctions) rather than assuming a universal setting.

### 8.7 Evidence limitations and verification priorities

- **Project identity and mismatch handling:** Obtain instrument Parameter 2 records and a reproducible batch/interface test to establish the actual project comparison and prompt behavior. [G04](#g04-empower-exception-behavior).
- **Custom fields:** Inspect a configured Empower project to confirm names, field scopes, data types, widths, and actual use of posting/review markers.
- **Mapping accuracy:** Export or inspect the existing Result Mapping and Sample Set Method Mapping records; verify all field names (especially `RESULT.MILL_RES_NUM`), direction of transfer, attribute semantics, and applicability to new projects.
- **Result handling:** Test numeric peak values, `ND`, absent peaks, and downstream calculation behavior using representative records; no error-handling or retry semantics are established by the laboratory guidance.
- **No vendor/runtime verification:** Nothing in §8 is `VENDOR_DOCUMENTED`, `OBSERVED`, or `VERIFIED`. The existing-project example and listed mappings are documented configuration guidance, not independent proof of correct interface operation.

## 9. Operational Workflows

**Scope:** Documented sequence and operator-facing boundaries for OIS sample intake, receiving, batch creation, batch testing, review, and project closure. This section is an **operational orientation**, not a validated standard operating procedure. The detailed technical requirements are maintained in their owning sections: OIS and test assignment [§2](#2-ois-integration-project-and-sample-setup), analysis configuration [§3](#3-analysis-configuration), batch setup [§4](#4-batch-configuration), instrument setup [§5](#5-instrument-configuration), file parsing [§6](#6-data-import-and-parsing), calculations/reporting [§7](#7-calculations-limits-and-reporting), and Empower [§8](#8-empower-integration). **Verification status:** Laboratory-documented only; not independently tested.

### 9.1 End-to-end process boundary and intake

**[WF-SCOPE-01] — `DOCUMENTED_DESCRIPTION` — OTC testing of OIS field samples.** The laboratory guidance describes an intended process beginning with an OIS testing-request file, followed by LabWare project/sample creation, assignment and testing of analyses, result reporting, and an outbound response to OIS. A complete step-by-step process is not established. **Technical owners:** [§2](#2-ois-integration-project-and-sample-setup), [§7](#7-calculations-limits-and-reporting).

**[WF-INTAKE-01] — `DOCUMENTED_DESCRIPTION` — File transfer boundary.** The documented transfer workflow uses SFTP and Windows shell scripts run by a LabWare background instance for incoming OIS requests and instrument files. It states that the scripts have `.bat` and `.txt` parts. This describes transport, **not** the XML validation, project-creation, or sample-login execution sequence. **Open:** [G01](#g01-xml-and-projectsample-setup).

**[WF-LOGIN-OPEN] — `UNRESOLVED` — Project and sample creation.** The laboratory guidance names *XML Parsing and Validation*, *Staging Tables*, *Investigations*, *Project Template*, and *Sample Template* but supplies no actionable sequence for these stages. Its documented analyte-code/media method-assignment prerequisites are maintained in [§2.3](#23-automated-test-assignment-prerequisites); they are **not** a complete login workflow. **Open:** [G01](#g01-xml-and-projectsample-setup).

### 9.2 Sample receiving

**[WF-RECEIVE-OPEN] — `UNRESOLVED` — Receiving operations.** The laboratory guidance lists *Sample Receiving Workflow*, *Seal Status*, and *Label Printing* without documenting operator actions, required fields, label layouts, acceptance criteria, or status transitions. Do not infer an executable receiving SOP from these headings.  **Needed evidence:** receiving workflow configuration, sanitized example records, and an observed test.

### 9.3 Batch creation, preparation, and verification

**[WF-BATCH-01] — `DOCUMENTED_DESCRIPTION` — Preparing a batch for testing.** The laboratory guidance identifies *Batch Creation*, *Standard Inventory*, *Stock Assignment*, and *Batch Verification* as operational stages. It describes batch protocols and QC/standard configuration elsewhere, but does **not** give a complete operator-facing batch creation or verification procedure under these headings. The underlying template, QC, stock, and batch-component requirements belong to [§4](#4-batch-configuration).

**[WF-ALIQUOT-01] — `DOCUMENTED_DESCRIPTION` — Applicable media requiring aliquots.** The laboratory guidance says that, when certain samples are added to a batch through the Batch Creation workflow, `VWF_SLTC_TESTING_BATCH_CREATE` checks `X_OCD_MONITORING.NUM_ALIQUOT` for method/media, assigns additional testing, and aliquots to child samples. This is a documented special case, not proof that all samples are aliquoted or that the workflow has been tested.

### 9.4 Batch testing: mode-specific operator boundaries

**[WF-TEST-01] — `DOCUMENTED_DESCRIPTION` — File import or Empower testing.** The laboratory guidance says that after batch creation and entry of pre-analysis results (instruments, stocks, and standards), results can be entered; **all non-optional batch results must be entered before file parsing or Empower testing proceeds**. The documents also use `Optional = F` and `Required = T` in related contexts; do not assume these are the same field. **Owning rule:** [§4.5](#45-pre-analysis-results-and-entry-mode-differences); **open:** [Q02](#q02-component-requiredness).

**[WF-FILE-01] — `DOCUMENTED_DESCRIPTION` — Instrument-file batch processing.** The laboratory guidance identifies **Batch → Batch Process Results** (`MNU_BM_SLTC_DATA_PARSE`) as the operator entry point. It describes selection of a file from the assigned instrument's directory, batch/file validation, and a confirmation prompt before import. The documented process uses a validation-only pass followed by a commit pass, but does **not** establish complete rollback or repeat-import safety. **Owning procedure and limits:** [§6](#6-data-import-and-parsing); **open:** [G02](#g02-parser-failure-and-recovery).

**[WF-MANUAL-01] — `DOCUMENTED_DESCRIPTION` — Manually entered batch results.** The laboratory guidance states that, for manual batch entry, the Batch Results table is **not validated at batch level** and the analyst and reviewer must ensure that necessary fields are entered. This is a documented description of the laboratory's manual workflow; it does not establish the absence of every possible validation elsewhere. **Owning detail:** [§4.5](#45-pre-analysis-results-and-entry-mode-differences); **open:** [Q09](#q09-manual-batch-checks).

**[WF-EMPOWER-01] — `DOCUMENTED_DESCRIPTION` — Empower batch project check.** The laboratory guidance says the selected batch instrument is compared with the Empower project currently loaded, and the user is prompted to change projects if they differ. It does not document the full operator test sequence, posting, exception resolution, or retry behavior. **Owning detail:** [§8](#8-empower-integration); **open:** [G04](#g04-empower-exception-behavior).

### 9.5 Review, release, closure, and outbound reporting

**[WF-REVIEW-OPEN] — `UNRESOLVED` — Review and release sequence.** The laboratory guidance includes *Batch Review* as a heading but does not specify the steps, roles, review statuses, acceptance criteria, or release transitions. One documented instruction associates variation-level `Reportable = T` with review/release and outbound OIS XML; another laboratory instruction also describes reportable final results, but the operational review procedure remains unestablished. **Owning result-selection rules:** [§7.5](#75-reportable-components-analyte-codes-and-ois-output); **open:** [G03](#g03-release-and-ois-response).

**[WF-CLOSE-OPEN] — `UNRESOLVED` — Project closure and OIS response.** The laboratory guidance states the purpose of producing an OIS response file and reports to CSHOs but leaves *Project Closure and OIS Reporting* without an implementation procedure. Do not infer response XML generation, delivery, acknowledgments, or error handling from result flags alone. **Open:** [G03](#g03-release-and-ois-response).

### 9.6 Documented stage map and verification priorities

| Stage | What is documented | What is not established |
|---|---|---|
| OIS request intake | SFTP and script mechanism; purpose of OIS request handling (the laboratory guidance) | XML validation, staging, project/sample creation, exception handling ([G01](#g01-xml-and-projectsample-setup)) |
| Sample receiving | Receiving, seal, and label stages are named (the laboratory guidance) | Operator sequence, required fields, status transitions |
| Batch creation | Stock, standards, QC, and batch verification topics are named; configuration described elsewhere (the laboratory guidance) | Full operator procedure and verification criteria |
| Batch testing | Pre-analysis results, manual/file/Empower distinctions, file import entry point (the laboratory guidance) | Complete mode-specific SOPs, error recovery ([G02](#g02-parser-failure-and-recovery), [G04](#g04-empower-exception-behavior)) |
| Batch review | Review stage named; reportability settings described (documented guidance) | Review and release criteria, statuses, approval actions ([G03](#g03-release-and-ois-response)) |
| Closure and response | OIS response and CSHO reporting are intended outputs (the laboratory guidance) | Closure sequence, XML construction and transmission ([G03](#g03-release-and-ois-response)) |

**Verification priorities:** Obtain configured workflow routines/screens and sanitized execution evidence for sample receiving, batch creation, batch review/release, and OIS response generation. Confirm manual-entry checks, import/Empower prerequisites, and failure handling separately. **No stage in this section is independently classified `OBSERVED`, `VERIFIED`, or `VENDOR_DOCUMENTED`.**

## 10. End-to-End Analysis Implementation Procedure

**Scope:** A practical, **recommended implementation sequence** for a new or revised analysis, derived from the laboratory-documented dependencies in §§1–9. This is a **planning and verification checklist**, not a vendor-prescribed sequence, validated laboratory SOP, or substitute for the owning configuration sections. Each item must be evaluated for applicability; do not interpret an unchecked item as a universally required LabWare setting. **Verification status:** Laboratory-documented only; not independently tested.

### 10.1 Define the implementation case before configuring

**[PROC-SCOPE-01] — `RECOMMENDATION` — All proposed analysis implementations.** Record the intended analysis name, OSHA method(s), field-sample media, requested analyte codes, expected results, and whether the work is a new analysis, a new method/variation of an existing analysis, or a modification of an existing implementation. Identify the testing mode: **manual result entry**, **instrument-file import**, **Empower**, or a documented combination. Identify whether QC/QCSM, RLSM, aliquoting, multiple instruments, or compatible-method batching are relevant. These are planning questions, **not** proof that every field or feature is required. **Basis:** [§1](#1-configuration-architecture-and-dependencies), [§2](#2-ois-integration-project-and-sample-setup), [§3](#3-analysis-configuration), [§4](#4-batch-configuration).

**Decision boundary:** If required media-to-method mappings, reportable analyte codes, or calculation/reporting expectations are unknown, mark them `UNRESOLVED`; do not select values from an unrelated analysis example. For an OIS workflow, note that inbound XML parsing, project/sample creation, and receiving procedures remain incomplete in the laboratory guidance ([G01](#g01-xml-and-projectsample-setup)).

### 10.2 Common configuration path (guidance-derived prerequisites)

The order below is a **recommended dependency-aware work order**, not a verified UI execution order. Use the linked sections for documented requirements, values, and applicability conditions.

| Step | Configuration activity and applicability | Technical owner | Proposed check / evidence to capture |
|---|---|---|---|
| 1 | **Confirm assignment prerequisites**, if automated OIS test assignment is in scope: analyte code, sample media, corresponding `X_OCD_MONITORING` monitoring method, and applicable method/variation association. | [§2.2](#22-occupational-chemical-database-reference-tables), [§2.3](#23-automated-test-assignment-prerequisites) | Record the relevant reference rows and expected method/variation; do not infer behavior for missing or multiple matches. |
| 2 | **Establish method records and associations:** identify the master `T_ANALYSIS_METHOD` record and configure applicable analysis method-to-variation pairs. Treat related-method exceptions separately. | [§3.2](#32-master-methods-analysis-methods-and-variations), [§1.3](#13-method-variation-and-test-assignment-dependencies) | Record method IDs, variation names, and source/configuration evidence; retain the physical-table naming question [Q03](#q03-methodvariation-relationships). |
| 3 | **Configure the analysis and variations:** analysis identity, conditional Batch Link and instrument group, applicable sample types, and destructive flag only where justified. | [§3.1](#31-analysis-record-and-conditional-settings), [§3.2](#32-master-methods-analysis-methods-and-variations) | Check exact matches and variation applicability; document effects of the destructive flag as unverified until tested. |
| 4 | **Configure result components:** base versus variation flags, optionality/display, reportable final components and OIS analyte-code aliases, plus applicable attributes and instrument-result flags. | [§3.3](#33-base-components-versus-variation-components), [§3.4](#34-attributes-and-instrument-result-component-flags), [§7.5](#75-reportable-components-analyte-codes-and-ois-output) | Inspect representative test components; keep [Q02](#q02-component-requiredness) and [Q07](#q07-reportability-semantics) open where precedence is not established. |
| 5 | **Configure batch testing, if used:** method-oriented Batch Test Template, Analysis Link Key, `TEST` membership, QC protocol, stocks, and batch result components as applicable. | [§4.1](#41-batch-template-identity-and-core-settings)–[§4.5](#45-pre-analysis-results-and-entry-mode-differences) | Capture template and link settings; distinguish stock references from instrument aliases and preserve [Q01](#q01-instrument-reference-fields). |
| 6 | **Configure calculations and reporting:** appropriate analysis limits, reporting-limit propagation, final-result calculations, recovery/QC calculations if used, and reportable output. | [§7](#7-calculations-limits-and-reporting) | Identify actual calculation records, expected result/limit transformations, and unknown execution or formatting behavior; do not invent formulas. |

**Conditional extensions:** If aliquoting is required, configure its variation and investigate `X_OCD_MONITORING.NUM_ALIQUOT` and `VWF_SLTC_TESTING_BATCH_CREATE` under [§3.5](#35-aliquoting-and-dynamically-assigned-testing). If compatible methods share a batch, follow the documented related-method exception in [§4.2](#42-test-membership-and-compatible-methods); do not infer compatibility from naming alone. For QC, QCSM, RLSM, and repeated CCVs, follow [§4.3](#43-batch-protocols-standards-and-qc-stock) rather than assuming every method needs the same standard set. **Basis:** documented guidance.

### 10.3 Select and configure the testing mode

| Mode | Additional documented configuration to consult | Boundaries and open questions |
|---|---|---|
| **Manual entry** | Batch result components and analyst/reviewer checks in [§4.5](#45-pre-analysis-results-and-entry-mode-differences); calculation-mode guidance in [§7.3](#73-calculation-execution-and-data-source-distinctions). | The laboratory guidance states manual Batch Results are not validated at batch level; do not infer that file-parser setup is needed or that no other validation exists. [Q09](#q09-manual-batch-checks). |
| **Instrument-file import** | Instrument-group/Parameter 1 conventions [§5](#5-instrument-configuration); Data File Source and Instrument File result [§4.4](#44-batch-result-components-and-reference-fields); parser contract and validation [§6](#6-data-import-and-parsing); calculation-mode guidance [§7.3](#73-calculation-execution-and-data-source-distinctions). | Do not assume parser dispatch, all returned-variable obligations, retry/rollback safety, or alternate path granularity are verified. [Q04](#q04-parser-dispatch), [Q05](#q05-parser-contract), [Q08](#q08-path-constant-granularity), [G02](#g02-parser-failure-and-recovery). |
| **Empower** | Instrument Parameter 2/project association [§5.4](#54-instrument-parameter-2-and-empower-project-association); four custom fields, sample-set/result mappings, and `ND` handling [§8](#8-empower-integration); conditional calculation settings [§7.3](#73-calculation-execution-and-data-source-distinctions). | Do not assume the instrument-file parser's constant, Data File Source, or Instrument File result is required for an Empower-only workflow. Validate copied mappings and project matching; error/retry behavior remains undocumented. [G04](#g04-empower-exception-behavior). |

**[PROC-MODE-01] — `RECOMMENDATION` — Mixed or atypical workflows.** If a method combines modes or uses an undocumented integration, assess each dependency separately and record the evidence; the laboratory guidance does not provide a universal combined-mode procedure. **Basis:** Documented configuration dependencies.

### 10.4 Verification checkpoints before relying on the configuration

The checks below are **recommended tests to design and record**, not claims that these tests have already passed or that the laboratory guidance prescribe formal acceptance criteria. Use sanitized evidence where appropriate.

| Checkpoint | What to test or inspect | Expected outcome source / unresolved boundary |
|---|---|---|
| **Assignment** | Representative analyte-code/media combinations; correct method and variation; missing and ambiguous mappings. | [§2.3](#23-automated-test-assignment-prerequisites); no documented missing/multiple-match handling. |
| **Analysis and batch linking** | Method/variation records, batch links, test membership, component visibility/requiredness. | [§1.2](#12-batch-membership-and-analysis-link-identity), [§3](#3-analysis-configuration), [§4](#4-batch-configuration); field semantics may remain unresolved. |
| **QC and aliquots (if used)** | Standard protocol/stock selection, repeated CCVs, aliquot creation, and resulting test associations. | [§3.5](#35-aliquoting-and-dynamically-assigned-testing), [§4.3](#43-batch-protocols-standards-and-qc-stock); no general acceptance criteria supplied. |
| **Result entry/import** | Manual result completion; file validation and rejection; parser result mapping and order; Empower project/mapping and `ND` values as applicable. | [§4.5](#45-pre-analysis-results-and-entry-mode-differences), [§6](#6-data-import-and-parsing), [§8](#8-empower-integration); failure/retry behavior is not established. |
| **Calculation and reporting** | Raw/final result transformation, blank dependencies, limits, recovery, final reportable components and analyte aliases. | [§7](#7-calculations-limits-and-reporting); no universal formulas, rounding rules, or release criteria supplied. |
| **Review, release and OIS response** | Only when the relevant configured workflow and acceptance criteria are available: review states, release, XML output, and delivery. | [§9.5](#95-review-release-closure-and-outbound-reporting), [G03](#g03-release-and-ois-response); currently not documented sufficiently for a pass/fail script. |

**[PROC-VERIFY-01] — `RECOMMENDATION` — Evidence capture.** For each executed check, record environment, date, method/analysis/variation, relevant configuration snapshot or routine version, input, expected outcome, actual outcome, and outstanding questions. Classify an inspected configuration as `OBSERVED` and a reproducible successful behavior as `VERIFIED` only when supporting evidence is retained; neither label applies automatically because a source document describes the behavior. See [§0.3](#03-evidence-classification) and [§12](#12-troubleshooting-and-verified-behaviors).

### 10.5 Completion boundary and handoff

**[PROC-CLOSE-01] — `RECOMMENDATION` — Implementation handoff.** Record which applicable configurations are complete, which verification checkpoints have evidence, and which open issues remain. Link method-specific deviations to [§11](#11-established-configuration-examples), verified failure modes to [§12](#12-troubleshooting-and-verified-behaviors), and unresolved items to [§13](#13-open-questions-and-undocumented-areas). An analysis must **not** be described as fully validated merely because this planning checklist is complete. The laboratory guidance does not provide a general go-live approval rule, formal sign-off procedure, or complete end-to-end release test. **Evidence limitations:** This sequence is synthesized from the technical guidance in §§1–9. It is `RECOMMENDATION` where it proposes work order or verification, while the linked sections retain the actual `DOCUMENTED_REQUIREMENT` and `DOCUMENTED_DESCRIPTION` statements. No vendor or runtime verification is asserted here.

## 11. Established Configuration Examples

**Scope:** Index the *specific analyses, methods, instruments, and example values* actually named in the laboratory guidance. This section records their described use and evidence limitations; reusable configuration instructions remain exclusively in §§1–8. **“Established” in this heading does not mean independently verified.** No configuration exports, subroutine source, instrument records, or executed test evidence have been supplied. The entries below are `IMPLEMENTATION_EXAMPLE` or `DOCUMENTED_DESCRIPTION`, **not** `OBSERVED` or `VERIFIED`.

### 11.1 ICP_OES / OSHA_1023 — instrument-file import example

**[EX-ICP-01] — `IMPLEMENTATION_EXAMPLE` — Analysis `ICP_OES`, method `OSHA_1023`.** This example is described as the **first configured batch**, involving unusually complex data parsing, ordered code execution, and linked references. This example motivates much of the documented configuration guidance, **not** evidence that its method-specific values should be copied to other analyses.

| Documented example detail | Evidence status | Reusable rule / limitation |
|---|---|---|
| Five result attributes: **Wavelength**, **Dilution Factor**, **%RSD**, **Timestamp**, **Interferences** | Documented example; not independently verified | [§3.4](#34-attributes-and-instrument-result-component-flags); other analyses may choose different attributes. |
| QC and QCSM variation names illustrated by `OSHA_1023_QC` and `OSHA_1023_QCSM` | Documented example; not independently verified | [§3.2](#32-master-methods-analysis-methods-and-variations); create types only when applicable. |
| Batch template named `OSHA_1023` | Documented example; not independently verified | [§4.1](#41-batch-template-identity-and-core-settings); exact name is an example of the method-based convention. |
| Before file processing, laboratory solutions and instruments **except the balance** must be entered | Documented example; not independently verified | [§4.5](#45-pre-analysis-results-and-entry-mode-differences); example of required pre-analysis entries, not a general exemption for balances. |
| Reportable **Beryllium (Final)** component alias `0365` | Documented example; not independently verified | [§3.3](#33-base-components-versus-variation-components), [§7.5](#75-reportable-components-analyte-codes-and-ois-output); preserve leading zero. |
| File-path constant example `SLTC_DATA_IMPORT_ICP_OES` and wildcard `D:\OIS Integration\Inbound ICP Results\*.csv` | Documented example; not independently verified | [§6.1](#61-file-location-and-batch-prerequisites); path is an example, not a currently verified server location. |
| Analysis parser `BM_SLTC_DATA_PARSE_ICP_OES` | Documented example; not independently verified | [§6.2](#62-import-entry-point-and-parser-dispatch); no parser source supplied. |
| Instrument-specific parser names `BM_SLTC_DATA_PARSE_ICP_OES_HOOVER` and `BM_SLTC_DATA_PARSE_ICP_OES_MILHOUSE`; `HOOVER` also illustrates Parameter 1 value | Documented example; not independently verified | [§5.2](#52-instrument-parameter-1-laboratory-instrument-identifier), [§6.2](#62-import-entry-point-and-parser-dispatch); dispatch not verified. |
| `BM_SLTC_DATA_PARSE_ICP_OES` described as constructing a `batchOrderArr` with test number, timestamp, and sample number; **only column 1** drives ordering | Documented example; not independently verified | [§6.5](#65-result-replication-and-batch-reordering); timestamps and post-import test addition are example-specific. |
| `CALC_ICP_FINAL` cited as a calculation that references Analysis Limits; `ICP_LOWER_LIMIT` cited as a reporting-limit format calculation | Documented example; not independently verified | [§7.1](#71-analysis-limits-and-enforcement-boundary), [§7.2](#72-lower-reporting-limit-and-final-result-propagation); no calculation code or validated output supplied. |

**Evidence boundary:** These examples are dispersed across the laboratory guidance; they do not constitute a complete exported `ICP_OES` configuration or a successful end-to-end test. Verify actual variation records, component flags, parser routines, calculations, batch protocol, and output before using this as a template. See [Q04](#q04-parser-dispatch), [Q05](#q05-parser-contract), [Q06](#q06-calculation-mode), and [G02](#g02-parser-failure-and-recovery).

### 11.2 GC — related-method batching example

**[EX-GC-01] — `IMPLEMENTATION_EXAMPLE` — Analysis `GC`, child method `OSHA_111`, parent method `OSHA_5000`.** The following specific compatible-method arrangement is documented:

| Record / relationship | Documented example value |
|---|---|
| Analysis Variation `OSHA_5000_PASSIVE` → Method | `OSHA_5000` |
| Analysis Method → Method / Variation | `OSHA_111` / `OSHA_5000_PASSIVE` |
| `T_ANALYSIS_METHOD` record `OSHA_111` → `X_RELATED_METHOD` | `OSHA_5000` |

**Open:** [Q03](#q03-methodvariation-relationships).

### 11.3 Empower-related examples (not a complete GC configuration)

**[EX-EMP-01] — `IMPLEMENTATION_EXAMPLE`.** The value `GC_Default\GC4 – LabWare1` illustrates a full Empower project name for instrument Parameter 2 (`EMPOWER PROJECT`). It also names an existing **MeCl2 (OSHA 1025)** project as the starting mapping to copy using **Save As** when onboarding a new project. These are **two distinct examples**; the available guidance does not establish that the MeCl2 mapping is a verified mapping for every GC method or that the example project path remains current.

### 11.4 Other named variations and analyses — evidence inventory

| Documented identifier | What is actually documented | What remains unknown |
|---|---|---|
| `OSHA_5000_AIR` | Example of an AIR field-sample variation. | Its actual component set, batch, instrument, and verified assignment behavior. |
| `OSHA_5001_ALIQUOT` | Example of an aliquot variation. | Which method/media require it and the implemented aliquot workflow. |
| `SILICA_XRD` | Heading under *Specific Analysis Configuration Notes*. | No substantive analysis-specific configuration provided. |
| `GRAV_PREWEIGH` | Heading under *Specific Analysis Configuration Notes*. | No substantive analysis-specific configuration provided. |
| `GRAV_POSTWEIGH` | Heading under *Specific Analysis Configuration Notes*. | No substantive analysis-specific configuration provided. |

**[EX-OTHER-OPEN] — `UNRESOLVED`.** The final three names are **placeholders for future evidence**, not established examples. Do not invent their methods, components, parsing approach, calculations, or relationship to one another.

### 11.5 Promoting an example to an evidence-backed implementation

**[EX-EVIDENCE-01] — `RECOMMENDATION`.** For each analysis eventually documented as implemented, attach or register a dated configuration snapshot (analysis, method/variation, components, batch template, instruments, calculations, and relevant integration mappings), identify the environment and version, and link reproducible test evidence where available. Distinguish a configuration merely **inspected** (`OBSERVED`) from a behavior actually **tested** (`VERIFIED`). Record deviations in this section, generalizable rules in their owning §§1–8, and unresolved questions in [§13](#13-open-questions-and-undocumented-areas). This is a proposed maintenance practice, not a requirement stated by the laboratory guidance.

## 12. Troubleshooting and Verified Behaviors

**Scope:** An evidence-status index of *reported or described* failure conditions, plus a separate register for independently reproduced and verified behavior. Troubleshooting guidance here does not replace the owning configuration requirements in §§2–8. **Verification status:** Laboratory-documented only; not independently tested.

### 12.1 Documented failure conditions and investigation leads

The following entries are **`DOCUMENTED_DESCRIPTION`**, not verified defects or guaranteed diagnostic rules. A source's stated cause should be treated as a hypothesis to check against actual configuration and runtime evidence.

| ID / situation | Documented symptom, condition, or control | Investigation starting point | Verification boundary |
|---|---|---|---|
| `TR-SRC-01` — File parser missing | The laboratory guidance states `MNU_BM_SLTC_DATA_PARSE` constructs `BM_SLTC_DATA_PARSE_{analysisName}` dynamically and exits with an error when the subroutine does not exist. | Confirm analysis name and parser routine name; inspect dispatch code before asserting exact resolution. [§6.2](#62-import-entry-point-and-parser-dispatch). | Error text and dispatch internals not supplied; [Q04](#q04-parser-dispatch). |
| `TR-SRC-02` — Previously imported file | The documented workflow states the menu routine checks whether **Instrument File** is populated and assumes the file has already been imported. | Inspect the batch's Instrument File result and any existing results before considering a repeat import. [§6.2](#62-import-entry-point-and-parser-dispatch). | This does **not** prove duplicate protection or safe retry; [G02](#g02-parser-failure-and-recovery). |
| `TR-SRC-03` — Pre-analysis results missing | The laboratory guidance describes file-import gating when required/non-optional batch results have not been entered; another laboratory instruction also describes gating for Empower testing. | Review batch results, component requiredness, and workflow mode. [§4.5](#45-pre-analysis-results-and-entry-mode-differences). | `Optional = F` and `Required = T` are not presumed identical; [Q02](#q02-component-requiredness). |
| `TR-SRC-04` — File validation fails | The laboratory guidance describes a `validateOnly` pass checking batch/sample IDs, standard/control types and counts, and presence/types of values; `validationStr` provides feedback. | Capture actual validation message and input file; compare with batch membership and parser logic. [§6.4](#64-validation-only-and-commit-processing). | Exact checks, messages, and rollback semantics not supplied; [Q05](#q05-parser-contract), [G02](#g02-parser-failure-and-recovery). |
| `TR-SRC-05` — Parsing deliberately stopped | The laboratory guidance describes `stopParse` as stopping import and returning a file to the instrument directory, and `wrongBatch` as a stop condition for automatic processing. | Inspect parser outputs, source file state, and the actual menu routine; do not assume a transaction rollback. [§6.3](#63-analysis-parser-result-contract), [§6.4](#64-validation-only-and-commit-processing). | Filesystem and partial-write behavior unknown; [G02](#g02-parser-failure-and-recovery). |
| `TR-SRC-06` — Empower project mismatch | The laboratory guidance states batch processing compares the selected instrument's expected Empower project with the project loaded in the interface and prompts a switch if they differ. | Check instrument Parameter 2 and the active Empower project. [§8.1](#81-instrument-association-and-project-validation). | Comparison and exception behavior unverified; [G04](#g04-empower-exception-behavior). |
| `TR-SRC-07` — Empower `ND` type mismatch | The laboratory guidance states Empower can return `ND` for an undetected peak, producing a runtime type mismatch if a numeric component type is used; it directs **Text** for Empower peak components. | Inspect peak component data type and representative Empower output. [§8.6](#86-empower-peak-values-and-calculation-considerations). | Exact error and downstream numeric handling not independently tested. |
| `TR-SRC-08` — Limit not applied | The laboratory guidance states Analysis Limits are not checked inherently in the described implementation and require explicit use in result calculations. | Inspect limit records and the relevant calculation; compare raw/final result and reporting limit. [§7.1](#71-analysis-limits-and-enforcement-boundary), [§7.2](#72-lower-reporting-limit-and-final-result-propagation). | Not proof of vendor-wide behavior or a specific observed failure. |
| `TR-SRC-09` — Out-of-order dependent calculations | The laboratory guidance describes calculation-order dependencies in file-import batches; see the authoritative mode-specific rule in [§7.3](#73-calculation-execution-and-data-source-distinctions). | Inspect calculation flags and actual execution path before changing them. [§7.3](#73-calculation-execution-and-data-source-distinctions). | No failed run or complete sequencing code supplied; [Q06](#q06-calculation-mode). |
| `TR-SRC-10` — Manual batch fields omitted | The laboratory guidance states manual batch entry has no batch-level validation of the Batch Results table and relies on analyst/reviewer checks. | Inspect entered batch results and the applicable manual review workflow. [§4.5](#45-pre-analysis-results-and-entry-mode-differences), [§9.4](#94-batch-testing-mode-specific-operator-boundaries). | Does not exclude validation elsewhere; [Q09](#q09-manual-batch-checks). |

**Not a troubleshooting conclusion:** A condition in this table does not establish that it occurred in the laboratory, that the stated mechanism is the only possible cause, or that the suggested inspection resolves it. Do not promote an entry to `VERIFIED` without a reproducible record.

### 12.2 Verified-behavior register

**No verified-behavior records currently exist.** Written configuration guidance is not test evidence. Do not manufacture a passing test, environment, date, error message, or resolution from the documented examples.

When evidence is supplied, add a compact entry here with a stable identifier such as `TR-VER-001`. A record should specify **analysis/method and workflow**, **environment/version and date**, **configuration or code revision**, **input and preconditions**, **reproduction steps**, **expected versus actual outcome**, **evidence location**, **root cause (if established)**, **resolution or workaround**, **retest result**, and **remaining applicability limits**. If a record demonstrates only a configured setting, classify it `OBSERVED` rather than `VERIFIED`.

### 12.3 Investigation and promotion procedure

1. Start with the symptom and testing mode; use the documented table above only to identify relevant technical topics and possible checks.
2. Preserve sanitized input, actual messages/logs, relevant configuration or code snapshots, environment, and time of occurrence. Do not include sensitive production data unnecessarily.
3. Record what was actually reproduced, including negative or inconclusive outcomes. Distinguish a suspected cause from a demonstrated cause and a proposed fix from a retested fix.
4. Add an evidence-backed `OBSERVED` or `VERIFIED` record only when the evidence supports that classification. If a general rule changes, update its owning section; retain the case-specific record here and cross-reference the relevant [§13](#13-open-questions-and-undocumented-areas) question.

**Verification priorities:** Parser dispatch and recovery ([Q04](#q04-parser-dispatch), [Q05](#q05-parser-contract), [G02](#g02-parser-failure-and-recovery)); requiredness semantics ([Q02](#q02-component-requiredness)); calculation ordering ([Q06](#q06-calculation-mode)); Empower exception handling ([G04](#g04-empower-exception-behavior)); and manual-batch checks ([Q09](#q09-manual-batch-checks)). These remain open until independently supported evidence is registered.

## 13. Open Questions and Undocumented Areas

The Q01–Q09 and G01–G04 identifiers are preserved as **unresolved investigation prompts, not confirmed defects**. Priorities below are *conditional implementation dependencies*, not deadlines or claims of risk severity. Do not close an item solely because written guidance describes expected behavior; obtain verification evidence.

### 13.1 Investigation index

| ID | Topic | Technical owner | Investigation priority | Evidence to obtain |
|---|---|---|---|---|
| [Q01](#q01-instrument-reference-fields) | Instrument reference fields | §5.1; §4.4 | High — instrument/file-import setup | Batch component and instrument records, relevant lookup/selection routine |
| [Q02](#q02-component-requiredness) | Component requiredness | §3.3; §4.5 | High — batch and component setup | Base/variation and batch component records; controlled missing-entry tests |
| [Q03](#q03-methodvariation-relationships) | Method/variation relationships | §3.2; §4.2 | High — automated assignment | Actual schema/table definitions and method/variation records; assignment tests |
| [Q04](#q04-parser-dispatch) | Parser dispatch | §6.2; §5.3 | High — file-import setup | `MNU_BM_SLTC_DATA_PARSE`, representative parsers, LabStation configuration |
| [Q05](#q05-parser-contract) | Parser contract | §6.3 | High — file-import setup | Menu routine and parser code; validation/reorder test cases |
| [Q06](#q06-calculation-mode) | Calculation mode | §7.3 | High — calculation-dependent analyses | Calculation records/code; ordered blank/unknown runs in each testing mode |
| [Q07](#q07-reportability-semantics) | Reportability semantics | §3.3; §7.5 | High — OIS reporting | Base and variation flags; review screens and sanitized OIS/report outputs |
| [Q08](#q08-path-constant-granularity) | Path constant granularity | §6.1 | Conditional — alternate import layouts | Constant definitions and directory-selection code |
| [Q09](#q09-manual-batch-checks) | Manual batch checks | §4.5; §9.4 | Conditional — manual-entry analyses | Manual workflow routines and tests with missing required results |
| [G01](#g01-xml-and-projectsample-setup) | XML and project/sample setup | §2.1–2.4; §9.1 | High — OIS intake | Sanitized XML/XSD, staging tables, intake routines, project/sample templates |
| [G02](#g02-parser-failure-and-recovery) | Parser failure and recovery | §6.4–6.6; §12.1 | High — file-import recovery | Parser/menu code, file state evidence, controlled failure/retry tests |
| [G03](#g03-release-and-ois-response) | Release and OIS response | §7.5; §9.5 | High — release/OIS output | Workflow records, release routines, sanitized response XML and delivery logs |
| [G04](#g04-empower-exception-behavior) | Empower exception behavior | §8.1–8.7 | Conditional — Empower analyses | Empower project/mapping exports, posting logs, mismatch/ND/error tests |

### 13.2 Open items and resolution questions

#### Q01 — Instrument reference fields

**Status:** `UNRESOLVED`. **Question:** Distinguish Alias Name, instrument group, and `X_STOCK`; preserve both documented statements. **Owner:** §5.1; §4.4. **Evidence needed:** Batch component and instrument records, relevant lookup/selection routine. **Priority:** High — instrument/file-import setup.

#### Q02 — Component requiredness

**Status:** `UNRESOLVED`. **Question:** Determine `Optional` versus `Required` behavior by object and workflow. **Owner:** §3.3; §4.5. **Evidence needed:** Base/variation and batch component records; controlled missing-entry tests. **Priority:** High — batch and component setup.

#### Q03 — Method/variation relationships

**Status:** `UNRESOLVED`. **Question:** Check child-table naming, related-method linkage, and selection rules. **Owner:** §3.2; §4.2. **Evidence needed:** Actual schema/table definitions and method/variation records; assignment tests. **Priority:** High — automated assignment.

#### Q04 — Parser dispatch

**Status:** `UNRESOLVED`. **Question:** Determine analysis versus instrument-specific dispatch and Parameter 1 handling. **Owner:** §6.2; §5.3. **Evidence needed:** `MNU_BM_SLTC_DATA_PARSE`, representative parsers, LabStation configuration. **Priority:** High — file-import setup.

#### Q05 — Parser contract

**Status:** `UNRESOLVED`. **Question:** Classify each context/return variable as mandatory, optional, or conditional. **Owner:** §6.3. **Evidence needed:** Menu routine and parser code; validation/reorder test cases. **Priority:** High — file-import setup.

#### Q06 — Calculation mode

**Status:** `UNRESOLVED`. **Question:** Establish triggering, dependency order, and `AutoCalculate` behavior. **Owner:** §7.3. **Evidence needed:** Calculation records/code; ordered blank/unknown runs in each testing mode. **Priority:** High — calculation-dependent analyses.

#### Q07 — Reportability semantics

**Status:** `UNRESOLVED`. **Question:** Determine flag precedence and output selection. **Owner:** §3.3; §7.5. **Evidence needed:** Base and variation flags; review screens and sanitized OIS/report outputs. **Priority:** High — OIS reporting.

#### Q08 — Path constant granularity

**Status:** `UNRESOLVED`. **Question:** Check whether non-analysis-level path selection is implemented. **Owner:** §6.1. **Evidence needed:** Constant definitions and directory-selection code. **Priority:** Conditional — alternate import layouts.

#### Q09 — Manual batch checks

**Status:** `UNRESOLVED`. **Question:** Identify checks actually performed and responsible stage. **Owner:** §4.5; §9.4. **Evidence needed:** Manual workflow routines and tests with missing required results. **Priority:** Conditional — manual-entry analyses.

#### G01 — XML and project/sample setup

**Status:** `UNRESOLVED`. **Question:** Document validation, creation, failure, and investigation handling. **Owner:** §2.1–2.4; §9.1. **Evidence needed:** Sanitized XML/XSD, staging tables, intake routines, project/sample templates. **Priority:** High — OIS intake.

#### G02 — Parser failure and recovery

**Status:** `UNRESOLVED`. **Question:** Establish partial writes, rollback, reimport, and retry behavior. **Owner:** §6.4–6.6; §12.1. **Evidence needed:** Parser/menu code, file state evidence, controlled failure/retry tests. **Priority:** High — file-import recovery.

#### G03 — Release and OIS response

**Status:** `UNRESOLVED`. **Question:** Document review/closure statuses, XML generation, delivery, and exceptions. **Owner:** §7.5; §9.5. **Evidence needed:** Workflow records, release routines, sanitized response XML and delivery logs. **Priority:** High — release/OIS output.

#### G04 — Empower exception behavior

**Status:** `UNRESOLVED`. **Question:** Establish posting/review markers, mapping errors, and recovery. **Owner:** §8.1–8.7. **Evidence needed:** Empower project/mapping exports, posting logs, mismatch/ND/error tests. **Priority:** Conditional — Empower analyses.

### 13.3 Resolution and deferral rules

When evidence arrives, record the evidence identifier, environment/version where applicable, tested conditions, observed result, and remaining limitations. Update the substantive rule only in its canonical owner; retain this question ID with a resolution note and link to evidence. A question can remain open or be deferred if its workflow is not currently being implemented. Do not manufacture answers or assign due dates from the laboratory guidance.
