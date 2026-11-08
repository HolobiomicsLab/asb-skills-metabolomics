---
name: duplicate-metabolite-deduplication
description: Use when verifying metabolite coverage from DDA LC-MS/MS organ/serum
  database sheets where per-sheet lists overlap and a deduplicated union count must
  be reconciled with a reported unique-metabolite total.
when_to_use_negative:
- Input is already a deduplicated feature table or annotated MSI output (e.g. the
  Annotator file from MSIannotator.m) rather than per-organ raw database sheets
- The task is metabolite annotation of MSI data via m/z matching, which is handled
  by MSIannotator.m rather than sheet-level count reconciliation
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MATLAB
  role: Tabular reader used to load the organ-specific database sheets and count/deduplicate
    metabolite names across sheets
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:900cf38d111ba66f452ad781cc6d0bdbc3e39bf54725dea038a131990a3d257f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# duplicate-metabolite-deduplication

## Summary

Pool metabolite names across the per-organ and serum sheets of 'Organ specific metabolite database.xlsx' and deduplicate to obtain the unique metabolite total, reconciling per-mode and union counts against reported figures (1298 positive, 515 negative, 1620 unique).

## When to use

Use when verifying metabolite coverage from DDA LC-MS/MS organ/serum database sheets where per-sheet lists overlap and a deduplicated union count must be reconciled with a reported unique-metabolite total.

## When NOT to use

- Input is already a deduplicated feature table or annotated MSI output (e.g. the Annotator file from MSIannotator.m) rather than per-organ raw database sheets
- The task is metabolite annotation of MSI data via m/z matching, which is handled by MSIannotator.m rather than sheet-level count reconciliation

## Inputs

- 'Organ specific metabolite database.xlsx' with Sheet1 (positive mode), Sheet2 (negative mode), and Sheet3–Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum), metabolite names in the first column of each sheet

## Outputs

- metabolite_count_verification.csv with one row per sheet plus the deduplicated union, recording per-organ, per-mode, and union counts and agreement/discrepancy flags

## How to apply

Load 'Organ specific metabolite database.xlsx' in MATLAB, reading Sheet1 (positive mode, 1298 metabolites), Sheet2 (negative mode, 515 metabolites), and the organ/serum sheets Sheet3–Sheet11 (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum) with metabolite names in the first column. Count metabolites per sheet and verify the ion-mode totals against 1298 (positive) and 515 (negative). Pool metabolite names across the nine organ/serum sheets and deduplicate to obtain the unique metabolite total. Reconcile the deduplicated union with the reported figure of 1620 unique metabolites identified from eight mouse organs and serum, and tabulate per-organ, per-mode, and union counts. Write the reconciliation to metabolite_count_verification.csv, marking agreement or discrepancy per row.

## Related tools

- **MATLAB** (Tabular reader used to load the organ-specific database sheets and count/deduplicate metabolite names across sheets) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
db = readcell('Organ specific metabolite database.xlsx'); counts = cellfun(@(s) height(s), sheets); union = numel(unique([sheets{:},1])); writetable(counts,'metabolite_count_verification.csv')
```

## Evaluation signals

- Recomputed positive-mode count (Sheet1) equals 1298
- Recomputed negative-mode count (Sheet2) equals 515
- Deduplicated union across the nine organ/serum sheets is consistent with the reported 1620 unique metabolites
- metabolite_count_verification.csv contains one row per sheet (Sheet1–Sheet11) plus a union row, each with an agreement/discrepancy flag
- Any mismatch between recomputed and reported totals is explicitly flagged in the output table rather than silently accepted

## Limitations

- The deduplicated union must be consistent with (not necessarily exactly equal to) the reported 1620, since pooling logic may differ from the authors'; discrepancies are flagged rather than auto-resolved
- The database is specific to eight mouse organs plus serum and two ion modes; other tissues or acquisition modes are out of scope

## Evidence
