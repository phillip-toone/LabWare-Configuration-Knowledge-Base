# LabWare Configuration Knowledge Base

## Purpose

Maintain one canonical, living Markdown reference for configuring and implementing laboratory analyses in LabWare LIMS. The reference is intended to support reliable LLM assistance without repeatedly re-explaining laboratory-specific conventions.

**Status:** initiation / source assessment. No canonical configuration requirements have yet been approved or consolidated. The Word documents in `source/` are preserved evidence, not automatically authoritative for all analyses.

## Repository layout

- `source/`: original, version-identifiable Word source snapshots (do not edit in place).
- `source/manifest.md`: source identities, SHA-256 checksums, provenance and handling rules.
- `structure-review.md`: assessment of the proposed knowledge-base organization, source coverage, overlaps, gaps and questions.
- `labware-configuration-kb.md`: **planned** single canonical knowledge base; intentionally not created in this initiation phase.

## Evidence and writing rules

1. Distinguish laboratory-specific **documented requirements** from **vendor-documented behavior**, **observed configuration**, **verified behavior**, **recommendations**, and **unresolved questions**.
2. A documented requirement is evidence of what a source instructs, **not** proof that LabWare enforces it or that an existing configuration is correct.
3. Record source ID and a precise location (page and heading, or paragraph/table reference) for each future substantive claim. Keep original terminology, including exact identifiers.
4. Preserve competing statements with their own citations. Never silently resolve a conflict or expand an example into a universal rule.
5. Specify applicability (e.g. file import, manual entry, Empower, QC, media, method). If unknown, say so.
6. Keep general rules in one canonical topic and link to them from implementation checklists and analysis-specific examples.
7. Do not use the separate LabWare 8 Technical Manual project as evidence unless its actual reference is supplied and registered.

## Suggested next steps

1. Extract a full, statement-level requirements crosswalk from both sources, with source IDs, exact locations, applicability, classification and question status.
2. Investigate overlaps and apparent conflicts identified in `structure-review.md`; retain unresolved items explicitly.
3. Create the canonical Markdown skeleton after the crosswalk has been reviewed for coverage.
4. Populate canonical topics in small commits; attach configuration exports and reproducible tests as evidence when available.

## Lightweight maintenance

Use descriptive Git commits; no approval workflow is required for a single maintainer. When a Word document changes, preserve the previous snapshot, register the new filename/version/checksum, and review only the affected crosswalk entries and canonical sections. Record substantive changes in the canonical document's change history. Use a private repository or sanitize evidence if exports contain sensitive laboratory, infrastructure, or integration details.

## First local Git commit

Review files before committing. From the cloned repository, after copying these files into place:

```sh
git status
git add README.md structure-review.md source/manifest.md source/Batch-and-Analysis-Configuration-Requirements.docx source/Sample-Testing-Configuration.docx
git commit -m "Initialize knowledge-base documentation and register source snapshots"
```
