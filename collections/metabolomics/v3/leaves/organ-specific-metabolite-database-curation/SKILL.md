---
name: organ-specific-metabolite-database-curation
description: Reference for building or validating an organ-partitioned metabolite
  database for ambient mass spectrometry imaging when adduct forms in MSI differ from
  LC-MS and annotation coverage must be expanded.
when_to_use_negative:
- The analysis is LC-MS/MS feature annotation in solution metabolomics, where LC-MS
  adduct forms are already correct and the MSI-specific adduct-form re-evaluation
  is unnecessary.
- The metabolite list has not yet been partitioned into organ-specific sheets (brain,
  liver, kidney, heart, spleen, lung, muscle, pancreas, serum); adduct expansion requires
  the organ partition as a starting point.
- The task is annotating an already processed MSI dataset against an existing database;
  use the MSIannotator.m workflow instead of rebuilding the database.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MATLAB
  role: Used to load 'Organ specific metabolite database.xlsx' via readcell/readtable,
    enumerate adduct forms, compute theoretical m/z, and aggregate per-organ adduct
    ion counts.
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator
  role: 'Downstream consumer: the curated organ database is imported as the ''Database''
    string-array sheet so MSIannotator.m can annotate MSI features with metabolite
    names and adduct forms.'
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: The executable script whose expected Database schema (metabolite names in
    first column, theoretical m/z in adduct-form columns) constrains the output format
    of the curated database.
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:8a09af21efb8bf82cfe018a6ceb04c8ae6259919fc4650acf176ccf56e03333e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# organ-specific-metabolite-database-curation

## Summary

Curation of an organ-specific metabolite database for AFADESI-MSI by evaluating adduct-form differences between LC-MS/MS and MSI measurements, expanding 1298 positive-mode and 515 negative-mode organ metabolites into 27,407 adduct ions with theoretical m/z.

## When to use

Reference for building or validating an organ-partitioned metabolite database for ambient mass spectrometry imaging when adduct forms in MSI differ from LC-MS and annotation coverage must be expanded.

## When NOT to use

- The analysis is LC-MS/MS feature annotation in solution metabolomics, where LC-MS adduct forms are already correct and the MSI-specific adduct-form re-evaluation is unnecessary.
- The metabolite list has not yet been partitioned into organ-specific sheets (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum); adduct expansion requires the organ partition as a starting point.
- The task is annotating an already processed MSI dataset against an existing database; use the MSIannotator.m workflow instead of rebuilding the database.

## Inputs

- Organ specific metabolite database.xlsx (1298 positive-mode metabolites in Sheet1, 515 negative-mode metabolites in Sheet2, organ/serum-split Sheets 3-11)
- Evaluation of adduct form differences between LC-MS/MS (DDA) and AFADESI-MSI metabolite ions
- Database.xlsx example metabolite database sheet (liver, positive mode) as template for theoretical m/z columns

## Outputs

- Organ-specific, adduct-ion-expanded metabolite database totaling 27,407 adduct ions with metabolite name, adduct form, theoretical m/z, organ, and ionization mode
- Per-sheet organ databases ready to be imported into MATLAB as the 'Database' sheet for MSIannotator

## How to apply

Start from the organ-specific metabolite lists in 'Organ specific metabolite database.xlsx' (1298 metabolites in positive mode Sheet1, 515 in negative mode Sheet2, split across brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum in Sheets 3-11). For each organ sheet and ionization mode, evaluate how adduct forms of metabolite ions differ between data-dependent LC-MS/MS acquisition and AFADESI-MSI acquisition, then enumerate the adduct forms actually observed in MSI. Compute theoretical m/z for every metabolite-adduct combination with standard adduct mass calculations, paralleling the theoretical m/z columns in Database.xlsx. Aggregate adduct ions per organ sheet and per mode with pandas or MATLAB and sum across all sheets to reach the target total of 27,407 adduct ions. Write the expanded database with columns for metabolite name, adduct form, theoretical m/z, organ, and ionization mode so it can be loaded as the 'Database' string-array sheet for MSIannotator.

## Related tools

- **MATLAB** (Used to load 'Organ specific metabolite database.xlsx' via readcell/readtable, enumerate adduct forms, compute theoretical m/z, and aggregate per-organ adduct ion counts.) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator** (Downstream consumer: the curated organ database is imported as the 'Database' string-array sheet so MSIannotator.m can annotate MSI features with metabolite names and adduct forms.) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator.m** (The executable script whose expected Database schema (metabolite names in first column, theoretical m/z in adduct-form columns) constrains the output format of the curated database.) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
readcell('Organ specific metabolite database.xlsx','Sheet','brain')
```

## Evaluation signals

- Sheet1 of 'Organ specific metabolite database.xlsx' contains exactly 1298 metabolites and Sheet2 contains exactly 515.
- Sheets 3-11 correspond, in order, to brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum metabolite databases.
- The expanded database's total adduct-ion row count equals the reported 27,407.
- Each row carries metabolite name, adduct form, theoretical m/z, organ, and ionization mode, matching the schema MSIannotator expects ('Database' with theoretical m/z in adduct-form columns).
- Theoretical m/z values reproduce the values recorded in the example Database.xlsx sheet (liver, positive mode).

## Limitations

- The curated database is organ-specific: a mismatch between the selected organ/mode and the MSI dataset being annotated invalidates annotation, so the database sheet must be checked for consistency before use.
- Adduct enumeration is grounded in the observed LC-MS vs AFADESI-MSI adduct difference for the eight mouse organs plus serum; applying it outside these conditions or to other tissues is not validated.
- The m/z error range in MSIannotator defaults to 10 ppm and must be adjusted (line 25) if a different tolerance is required.

## Evidence
