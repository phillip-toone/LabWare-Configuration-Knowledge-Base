# Source Manifest

Status: initial registration, 2026-10-14. Checksums independently verified against the two supplied attachments. Original Word content has not been modified.

| Source ID | Repository path | SHA-256 | Description | Source status |
|---|---|---|---|---|
| S01 | `source/Batch-and-Analysis-Configuration-Requirements.docx` | `8b276ee30793ed958eaf4b7ccaa0f688892f5185107375ef5bf6bfd2e33b8ec1` | *Batch and Analysis Configuration Requirements*; 3 pages; focused notes for batch setup and instrument-file parsing, using ICP_OES / OSHA_1023 examples. | Original snapshot; source requirements and examples not yet verified against configuration. |
| S02 | `source/Sample-Testing-Configuration.docx` | `b6715028afec34529a6caf7a30084b0ec59db6a53ceb2fe62f913852d2d9d9a2` | Internal title: *End-To-End Testing of OIS samples at the OSHA Technical Center using LabWare LIMS*; 21 pages; version history lists v0, 19 May 2026, initial draft. | Original snapshot; includes detailed guidance and unfinished sections. |

## Verification

Run from the repository root:

```sh
sha256sum source/Batch-and-Analysis-Configuration-Requirements.docx source/Sample-Testing-Configuration.docx
```

Compare output against this manifest. If a source is revised, add a separately identifiable snapshot and manifest entry; do not silently overwrite the version cited by existing knowledge entries.

## Citation convention

Use `S01, p. 2, [heading]` or `S02, pp. 17–18, Data File Batches / Validation`. For tables, include the table title and row where relevant. Source statements are classified as `DOCUMENTED_REQUIREMENT`, `IMPLEMENTATION_EXAMPLE`, `DOCUMENTED_DESCRIPTION`, `RECOMMENDATION`, or `UNRESOLVED`; future configuration exports can additionally be `OBSERVED`, and reproducible testing can establish `VERIFIED` behavior. `VENDOR_DOCUMENTED` requires an actual registered vendor source.

## Exclusions

No LabWare technical manual, live configuration export, parser code, or independently verified runtime behavior was provided for this registration. The documents' claims should not be represented as vendor specifications.
