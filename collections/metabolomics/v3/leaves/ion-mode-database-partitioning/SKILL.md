---
name: ion-mode-database-partitioning
description: Use when preparing or validating an organ-specific metabolite database
  for MSI annotation, where measured m/z values must be matched against theoretical
  adduct m/z per ionization mode and per organ.
when_to_use_negative:
- The MSI dataset is not organ-specific or its detection mode is unknown — mixing
  modes or organs would violate the consistency check MSIannotator requires.
- The task is annotating non-MSI LC-MS data, where adduct-form differences between
  LC-MS and AFADESI-MSI have not been evaluated and this database's partitioning does
  not apply.
- Only a single pooled (non-partitioned) metabolite list is available and per-organ
  splitting has not been performed; the skill requires the 11-sheet structure.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MSIannotator
  role: Consumes the partitioned database sheets together with MSIdata to annotate
    metabolites and adducts; the organ/mode consistency check is performed before
    running MSIannotator.m
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Environment in which the database sheets are imported as string arrays (renamed
    Database) and paired with MSIdata for annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:843024e1865d99ded1f977a469cd94590a08f217a166f790862255022d84f7c1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ion-mode-database-partitioning

## Summary

Organize an MSI metabolite reference database into positive-mode and negative-mode sheets and further split them per organ (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum) so annotation runs only against the adduct forms and metabolites relevant to the tissue and ionization mode of the MSI data.

## When to use

Use when preparing or validating an organ-specific metabolite database for MSI annotation, where measured m/z values must be matched against theoretical adduct m/z per ionization mode and per organ.

## When NOT to use

- The MSI dataset is not organ-specific or its detection mode is unknown — mixing modes or organs would violate the consistency check MSIannotator requires.
- The task is annotating non-MSI LC-MS data, where adduct-form differences between LC-MS and AFADESI-MSI have not been evaluated and this database's partitioning does not apply.
- Only a single pooled (non-partitioned) metabolite list is available and per-organ splitting has not been performed; the skill requires the 11-sheet structure.

## Inputs

- Organ specific metabolite database.xlsx (11-sheet workbook: Sheet1 positive mode, Sheet2 negative mode, Sheets 3–11 per organ/serum)
- Metabolite names in the first column and theoretical m/z in specified adduct-form columns per sheet
- Preprocessed MSI dataset (MSIdata.xlsx) whose organ and ion mode determine which sheet to use

## Outputs

- Partitioned database sheets (Database string array) ready for import into MATLAB alongside MSIdata
- Database validation record confirming 1298 positive-mode and 515 negative-mode metabolite counts and the presence of all nine organ/serum sheets

## How to apply

Build the workbook as 11 sheets: Sheet1 holding 1298 metabolites for positive mode and Sheet2 holding 515 for negative mode, with each organ's metabolite database further divided into brain (Sheet3), liver (Sheet4), kidney (Sheet5), heart (Sheet6), spleen (Sheet7), lung (Sheet8), muscle (Sheet9), pancreas (Sheet10), and serum (Sheet11). Each sheet must place metabolite names in the first column and theoretical m/z in the specified adduct-form columns, matching the layout expected by MSIannotator.m. Before running annotation, verify that the selected organ sheet and detection mode (positive or negative) are consistent with the MSIdata being annotated, since the database originated from evaluating adduct-form differences between LC-MS and AFADESI-MSI. If the annotation tolerance needs adjustment, note that the matched m/z error is governed in the script (default 10 ppm) rather than by re-partitioning the database. The overall adduct-ion space covered by this database construction comprises 27,407 adduct ions spanning the eight organs plus serum.

## Related tools

- **MSIannotator** (Consumes the partitioned database sheets together with MSIdata to annotate metabolites and adducts; the organ/mode consistency check is performed before running MSIannotator.m) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Environment in which the database sheets are imported as string arrays (renamed Database) and paired with MSIdata for annotation)

## Examples

```
readcell('Organ specific metabolite database.xlsx','Sheet','Sheet1') % import positive-mode sheet into MATLAB as Database
```

## Evaluation signals

- Sheet1 contains exactly 1298 metabolites and Sheet2 exactly 515, matching the described positive- and negative-mode counts.
- Sheets 3–11 correspond in order to brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum databases.
- Every sheet places metabolite names in the first column and theoretical m/z in the specified adduct-form columns.
- The selected organ sheet and detection mode match the organ and mode of the MSIdata being annotated before MSIannotator.m is executed.
- The full database spans 27,407 adduct ions across the eight organs and serum.

## Limitations

- The database partitioning is specific to AFADESI-MSI adduct forms; LC-MS adduct behavior differs and was the basis of the evaluation step, so direct reuse for LC-MS data may mis-assign adducts.
- The m/z error tolerance (default 10 ppm) lives in MSIannotator.m (line 25) rather than in the database partitioning itself; changing it requires code modification, not database restructuring.
- The partitioning covers only the eight mouse organs plus serum; other tissues are not represented in the 11-sheet structure.

## Evidence
