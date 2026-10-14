# Requirements crosswalk — focused quality review

Review date: 2026-10-14

## Scope

Reviewed the 240 entries in the original `crosswalk/requirements.csv` against S01 and S02 Word sources. This is a focused quality pass, not independent confirmation of LabWare behavior. All 240 original IDs remain. Thirty-two atomic child records were added with suffixes `-A1`, `-A2`, etc. The reviewed crosswalk now has **272 rows**.

## Findings and changes

- **Source references:** Original paragraph references were checked against the corresponding Word paragraph text. The 24 `GAP` entries are synthesized annotations of headings, not verbatim statements. The 5 `ISS` entries are synthesized investigation flags. Fifteen Empower table entries were located at the stated tables; table references are less precise than paragraph references and should be improved to explicit row coordinates in a future pass.
- **Compound statements:** Ten parent entries (`BAT-001`, `ANA-005`, `ANA-031`, `ANA-033`, `ANA-043`, `BAT-047`, `IMP-019`, `IMP-039`, `EMP-003`, `ANA-018`) were decomposed into 32 child assertions. Parents retain source wording; children are paraphrases linked to the parent. They are *not* additional independent source evidence.
- **Classification:** Reclassified selected headings, navigation instructions and mapping table headers as `CONTEXT`; explicit examples as `IMPLEMENTATION_EXAMPLE`; parser context inputs and observed workflow descriptions as `DOCUMENTED_BEHAVIOR`; example validation checks as `RECOMMENDATION`. `DOCUMENTED_BEHAVIOR` means **behavior described in the source**, not tested or vendor verified.
- **Applicability:** Added targeted scope notes for file imports, Empower, OIS intake, QC, variation settings and reporting calculations. Blank scope elsewhere does not imply universal applicability.
- **Conflicts:** Preserved five existing `ISS` records as open investigation flags, with links to relevant entries. No source disagreement was adjudicated.

## Remaining limitations

1. Some source rows are still non-atomic. This pass prioritizes high-impact compound entries; it is not an exhaustive semantic decomposition of every paragraph.
2. Source paragraph indices are 0-based in the original references and remain so. Source table references should be expanded to unambiguous row coordinates.
3. The 24 heading-based gaps need section-level checks before concluding the subject is wholly undocumented. A heading can be empty while related details occur elsewhere.
4. No code, LabWare records, runtime tests, vendor manual, or separate technical manual conversion was used to validate behavior.
5. The crosswalk is an extraction and review aid, not the canonical knowledge base. Parent/child rows should not be counted as independent requirements.

## Priority follow-up evidence

Inspect `MNU_BM_SLTC_DATA_PARSE`, `BM_SLTC_DATA_PARSE_ICP_OES`, `CALC_ICP_FINAL`, and a representative batch template and analysis variation, if available. Resolve `ISS-001` through `ISS-005` only with traceable evidence. Avoid promoting implementation examples to general standards.
