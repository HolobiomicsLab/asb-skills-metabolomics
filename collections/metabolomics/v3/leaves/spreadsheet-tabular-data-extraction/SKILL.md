---
name: spreadsheet-tabular-data-extraction
description: Use when a multi-sheet .xlsx workbook organizes entities per condition
  (e.g. organ/serum sheets and ion-mode sheets) and per-sheet counts must be verified
  against reported totals and pooled/deduplicated.
when_to_use_negative:
- The workbook contains mass-spectrometry feature intensities to be annotated (e.g.
  MSIdata.xlsx) rather than a metabolite name database — use MSIannotator.m annotation
  instead
- The data are raw spectra or imaging data rather than tabular spreadsheet records;
  spreadsheet extraction does not apply to m/z feature lists or IMS imaging files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MATLAB
  role: 'Tabular reader and counting environment: import the .xlsx sheets, count metabolites
    per sheet, deduplicate pooled names, and write the reconciliation CSV'
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:3c09b91a93aa99b0d5d707d079db88456006b5ef3770ddd4e3105338f85d0198
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spreadsheet-tabular-data-extraction

## Summary

Extract, count, and reconcile tabular metabolite records stored across multiple sheets of an .xlsx workbook (here 'Organ specific metabolite database.xlsx' with 11 sheets) using MATLAB, verifying sheet-level counts against reported totals (1298 positive, 515 negative) and deduplicating pooled names to reach the reported union (1620 unique metabolites).

## When to use

Use when a multi-sheet .xlsx workbook organizes entities per condition (e.g. organ/serum sheets and ion-mode sheets) and per-sheet counts must be verified against reported totals and pooled/deduplicated.

## When NOT to use

- The workbook contains mass-spectrometry feature intensities to be annotated (e.g. MSIdata.xlsx) rather than a metabolite name database — use MSIannotator.m annotation instead
- The data are raw spectra or imaging data rather than tabular spreadsheet records; spreadsheet extraction does not apply to m/z feature lists or IMS imaging files

## Inputs

- Organ specific metabolite database.xlsx — multi-sheet workbook with Sheet1 (positive mode, 1298 metabolites), Sheet2 (negative mode, 515 metabolites), and Sheet3–Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum), metabolite names in the first column
- Reported totals to reconcile against: 1298 positive-mode, 515 negative-mode, 1620 unique metabolites

## Outputs

- Per-sheet metabolite counts for Sheet1–Sheet11 (per-organ and per-mode)
- Deduplicated union count across the nine organ/serum sheets
- metabolite_count_verification.csv — one row per sheet plus the union, with agreement/discrepancy flags versus reported totals

## How to apply

Load 'Organ specific metabolite database.xlsx' in MATLAB with a tabular reader: read Sheet1 (positive mode), Sheet2 (negative mode), and the organ/serum sheets Sheet3–Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum), with metabolite names in the first column. Count the metabolites in each sheet and check the ion-mode totals against the reported 1298 (positive) and 515 (negative). Pool metabolite names across the nine organ/serum sheets and deduplicate to obtain the unique metabolite total, then reconcile the deduplicated union with the reported figure of 1620 unique metabolites. Tabulate per-organ, per-mode, and union counts, marking agreement or discrepancy versus the reported values.

## Related tools

- **MATLAB** (Tabular reader and counting environment: import the .xlsx sheets, count metabolites per sheet, deduplicate pooled names, and write the reconciliation CSV) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
T = readtable('Organ specific metabolite database.xlsx','Sheet','Sheet1','Range','A:A'); n_pos = height(T)
```

## Evaluation signals

- Recomputed positive-mode (Sheet1) count equals the reported 1298 metabolites
- Recomputed negative-mode (Sheet2) count equals the reported 515 metabolites
- Deduplicated union across the nine organ/serum sheets is consistent with the reported 1620 unique metabolites
- metabolite_count_verification.csv contains exactly one row per sheet (11 sheets) plus the union row, each with an agreement/discrepancy flag
- Any mismatch versus reported totals is explicitly flagged in the output table rather than silently accepted

## Limitations

- Counts depend on the workbook's own organization (Sheet1–Sheet11 layout); a differently structured or user-supplied database will not match the documented sheet ordering
- The union of 1620 unique metabolites relies on exact metabolite-name matching for deduplication; naming inconsistencies across organ sheets could inflate or deflate the union count
- Verification is a consistency check against reported totals — it cannot detect errors already present in the published database

## Evidence
