---
name: ion-mode-partition-analysis
description: Use when a metabolomics paper reports ion-mode-specific or organ-specific
  metabolite counts (e.g. positive/negative mode totals, deduplicated unique-metabolite
  totals) and a companion database file must be counted and reconciled against those
  claims.
when_to_use_negative:
- The goal is metabolite annotation of MSI peaks, not verifying database coverage
  counts — use MSIannotator.m on MSIdata and Database instead.
- The database file's sheets do not follow the documented layout (metabolite names
  in first column, organ/mode sheet structure), so sheet-level counts would not map
  to the reported partitions.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MATLAB
  role: Used to load the xlsx database sheets as string arrays, count metabolites
    per sheet, deduplicate the pooled metabolite names, and write the reconciliation
    table
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:65301d8ceb00999a2ff212354102d478921832eec45673a1c3bb7d8f13835bdf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ion-mode-partition-analysis

## Summary

This skill verifies and reconciles metabolite counts partitioned by ion mode and organ against a paper's reported totals, using the MSIannotator 'Organ specific metabolite database.xlsx' (1298 metabolites in positive mode Sheet1, 515 in negative mode Sheet2, split across organ sheets Sheet3-Sheet11) and the DDA LC-MS/MS figure of 1620 unique metabolites from eight mouse organs and serum.

## When to use

Use when a metabolomics paper reports ion-mode-specific or organ-specific metabolite counts (e.g. positive/negative mode totals, deduplicated unique-metabolite totals) and a companion database file must be counted and reconciled against those claims.

## When NOT to use

- The goal is metabolite annotation of MSI peaks, not verifying database coverage counts — use MSIannotator.m on MSIdata and Database instead.
- The database file's sheets do not follow the documented layout (metabolite names in first column, organ/mode sheet structure), so sheet-level counts would not map to the reported partitions.

## Inputs

- Organ specific metabolite database.xlsx (Sheet1: positive mode, Sheet2: negative mode, Sheet3-Sheet11: brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum; metabolite names in first column)
- Reported figures to reconcile: 1298 positive-mode metabolites, 515 negative-mode metabolites, 1620 unique metabolites

## Outputs

- metabolite_count_verification.csv with one row per sheet plus the union, recording per-organ/per-mode counts and agreement or discrepancy versus reported totals

## How to apply

Load 'Organ specific metabolite database.xlsx' in MATLAB (or an equivalent tabular reader), reading Sheet1 (positive mode) and Sheet2 (negative mode) plus the organ/serum sheets Sheet3-Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum), with metabolite names in the first column. Count rows per sheet and check the ion-mode totals against the reported 1298 (positive) and 515 (negative). Pool the metabolite names across the nine organ/serum sheets and deduplicate to obtain the union; the deduplicated union should be consistent with the 1620 unique metabolites identified via DDA LC-MS/MS from eight mouse organs (brain, liver, kidney, heart, spleen, lung, muscle, pancreas) and serum. Tabulate per-organ, per-mode, and union counts and write the reconciliation to a CSV (e.g. metabolite_count_verification.csv) with one row per sheet plus the union, flagging any agreement or discrepancy versus the reported totals. The rationale is that the database file is the paper's own structured record of its identifications, so per-sheet counts provide a direct, reproducible check on abstract-level claims.

## Related tools

- **MATLAB** (Used to load the xlsx database sheets as string arrays, count metabolites per sheet, deduplicate the pooled metabolite names, and write the reconciliation table) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
counts = arrayfun(@(s) height(readtable('Organ specific metabolite database.xlsx','Sheet',s)), 1:11); union = numel(unique(pool_names_sheets3_11))
```

## Evaluation signals

- Positive-mode sheet (Sheet1) row count equals the reported 1298 metabolites
- Negative-mode sheet (Sheet2) row count equals the reported 515 metabolites
- Deduplicated union of metabolite names across Sheet3-Sheet11 (brain through serum) is consistent with the reported 1620 unique metabolites
- Reconciliation CSV contains one row per sheet (11 sheets) plus a union row, with explicit agreement/discrepancy flags and no missing sheets
- Any mismatch between recomputed and reported counts is flagged rather than silently accepted

## Limitations

- The mode-level counts are recorded only in the companion database file, not directly derived from raw LC-MS/MS data; verification depends on the file's integrity.
- The union check is consistency-based ('consistent with 1620'), so small overlaps between organ sheets can make exact reconciliation ambiguous if deduplication rules differ from the authors'.
- The repository has no changelog, so changes to the database file over time cannot be tracked against previously reported counts.

## Evidence

- [discussion] _No changelog found._: [evidence span withheld — anchor status: not_found]
