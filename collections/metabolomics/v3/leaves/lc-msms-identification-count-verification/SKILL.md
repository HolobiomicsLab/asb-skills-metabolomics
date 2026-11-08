---
name: lc-msms-identification-count-verification
description: Use when a paper reports a total unique-metabolite count from DDA LC-MS/MS
  and an accompanying organ/mode-specific database file (e.g. an .xlsx workbook with
  per-organ sheets) that must be reconciled against that count.
when_to_use_negative:
- The goal is annotating MSI features against the database rather than verifying database
  counts — use MSIannotator.m (10 ppm m/z error, adjustable at line 25) instead
- The identification list is a single flat feature table with no organ/mode sheet
  structure, so the per-sheet and per-mode verification scheme cannot be applied
tools:
- name: MATLAB
  role: Tabular reader used to load each sheet of 'Organ specific metabolite database.xlsx',
    count metabolites per sheet/mode, deduplicate the pooled union, and write the
    reconciliation CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:331d7925cd498b66ccc714cae7f7d4697812918024aeab498d130edb9e893e3f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MATLAB
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# lc-msms-identification-count-verification

## Summary

Verifies metabolite identification counts from DDA LC-MS/MS by re-counting the organ-specific metabolite database workbook (1298 positive-mode, 515 negative-mode, 1620 unique metabolites across eight mouse organs and serum) in MATLAB and reconciling recomputed totals against reported figures.

## When to use

Use when a paper reports a total unique-metabolite count from DDA LC-MS/MS and an accompanying organ/mode-specific database file (e.g. an .xlsx workbook with per-organ sheets) that must be reconciled against that count.

## When NOT to use

- The goal is annotating MSI features against the database rather than verifying database counts — use MSIannotator.m (10 ppm m/z error, adjustable at line 25) instead
- The identification list is a single flat feature table with no organ/mode sheet structure, so the per-sheet and per-mode verification scheme cannot be applied

## Inputs

- Organ specific metabolite database.xlsx — established database with 1298 metabolites in positive mode (Sheet1), 515 in negative mode (Sheet2), and per-organ/serum sheets Sheet3–Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum) with metabolite names in the first column
- Reported totals to verify: 1298 (positive mode), 515 (negative mode), 1620 unique metabolites across eight organs and serum

## Outputs

- Per-sheet and per-mode metabolite counts (positive, negative, and each organ/serum sheet)
- Deduplicated union count of unique metabolites across the nine organ/serum sheets
- Reconciliation table (e.g. metabolite_count_verification.csv) with one row per sheet plus the union, marking agreement or discrepancy versus reported totals

## How to apply

Load 'Organ specific metabolite database.xlsx' into MATLAB via a tabular reader, reading Sheet1 (positive mode), Sheet2 (negative mode), and the per-organ sheets Sheet3–Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum) with metabolite names in the first column. Count entries per sheet and check ion-mode totals against the reported 1298 (positive) and 515 (negative). Pool metabolite names across the nine organ/serum sheets and deduplicate to obtain the unique-metabolite union. Reconcile the deduplicated union against the reported 1620 unique metabolites identified from the eight mouse organs and serum, and write a reconciliation table (one row per sheet plus the union) marking agreement or discrepancy for each reported total.

## Related tools

- **MATLAB** (Tabular reader used to load each sheet of 'Organ specific metabolite database.xlsx', count metabolites per sheet/mode, deduplicate the pooled union, and write the reconciliation CSV)

## Examples

```
S1 = readtable('Organ specific metabolite database.xlsx','Sheet','Sheet1'); S2 = readtable('Organ specific metabolite database.xlsx','Sheet','Sheet2'); [height(S1), height(S2)]
```

## Evaluation signals

- Recomputed positive-mode sheet count equals 1298
- Recomputed negative-mode sheet count equals 515
- Deduplicated union across Sheet3–Sheet11 is consistent with the reported 1620 unique metabolites
- Reconciliation table contains exactly one row per sheet (Sheet1–Sheet11) plus a union row, with agreement/discrepancy flags populated for every reported total
- Any mismatch between recomputed and reported totals is explicitly flagged rather than silently dropped

## Limitations

- Verification depends on the workbook's sheet layout (Sheet1 = positive, Sheet2 = negative, Sheet3–Sheet11 = brain through serum); a differently structured file breaks the per-sheet mapping
- The union reconciliation checks consistency with the reported 1620, but the article context does not define a tolerance rule if the deduplicated union differs from the reported total
- No changelog was found for the source repository (Yingzhu96/MSIannotator), so repo-side updates to the database workbook cannot be tracked

## Evidence
