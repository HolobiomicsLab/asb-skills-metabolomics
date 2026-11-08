---
name: matlab-data-import
description: Use when a preprocessed MSI dataset (e.g. organ tissue measured in positive
  or negative mode) and a metabolite database must be loaded into MATLAB as string-array
  variables before running m/z-based annotation.
when_to_use_negative:
- When the MSI data have not been preprocessed and still contain raw, unaligned m/z
  features — the import expects a preprocessed dataset as defined by the MSIannotator
  input requirements
- When the goal is annotation itself rather than data loading; that is handled by
  the separate MSIannotator.m script, not this import step
- When the available database's organ or detection mode does not match the MSIdata
  being annotated — importing a mismatched pair would propagate incorrect annotations
  downstream
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MATLAB
  role: Environment in which the MSI dataset and metabolite database sheets are imported
    as string arrays named MSIdata and Database prior to annotation
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: Downstream MATLAB script that consumes the imported MSIdata and Database string
    arrays to perform metabolite/adduct annotation
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:33db712d3ac45173159028e9399a97897765a732c86e4e5db98fdcd00dc35df8
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
yes_rate: 1.0
n_claims: 12
---

# matlab-data-import

## Summary

Import preprocessed mass spectrometry imaging (MSI) data and an organ-specific metabolite database into MATLAB as string arrays named MSIdata and Database, preparing them for the MSIannotator annotation step.

## When to use

Use when a preprocessed MSI dataset (e.g. organ tissue measured in positive or negative mode) and a metabolite database must be loaded into MATLAB as string-array variables before running m/z-based annotation.

## When NOT to use

- When the MSI data have not been preprocessed and still contain raw, unaligned m/z features — the import expects a preprocessed dataset as defined by the MSIannotator input requirements
- When the goal is annotation itself rather than data loading; that is handled by the separate MSIannotator.m script, not this import step
- When the available database's organ or detection mode does not match the MSIdata being annotated — importing a mismatched pair would propagate incorrect annotations downstream

## Inputs

- Preprocessed MSI dataset sheet with measured m/z in the first column (e.g. MSIdata.xlsx, liver tissue in positive mode)
- Metabolite database sheet with metabolite names in the first column and theoretical m/z in specified adduct-form columns (e.g. Database.xlsx, liver database in positive mode)
- Organ-specific metabolite database workbook Organ specific metabolite database.xlsx (1298 metabolites positive mode Sheet1; 515 negative mode Sheet2; organ split into Sheets 3–11: brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum)

## Outputs

- MATLAB string-array variable named MSIdata containing the preprocessed MSI dataset
- MATLAB string-array variable named Database containing the matched organ/mode metabolite database

## How to apply

Import two sheets — one holding the preprocessed MSI dataset with measured m/z values in the first column, the other holding the metabolite database with metabolite names in the first column and theoretical m/z in adduct-form columns — into MATLAB in string-array format, renaming them MSIdata and Database respectively. Verify the input contents match the required input-file structure, using the example files MSIdata.xlsx and Database.xlsx (liver tissue in positive mode) as templates. Before proceeding, check that the selected organ and the detection mode (positive or negative mode) of the database are consistent with those of the MSIdata to be annotated, since a mismatched organ database (e.g. brain vs. liver, Sheets 3–4 of Organ specific metabolite database.xlsx) would yield incorrect annotations. Once both variables are correctly named and validated, the workspace is ready for MSIannotator.m, which applies a default 10 ppm m/z error tolerance.

## Related tools

- **MATLAB** (Environment in which the MSI dataset and metabolite database sheets are imported as string arrays named MSIdata and Database prior to annotation) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator.m** (Downstream MATLAB script that consumes the imported MSIdata and Database string arrays to perform metabolite/adduct annotation) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
MSIdata = readmatrix('MSIdata.xlsx','Sheet','MSIdata'); Database = readmatrix('Database.xlsx','Sheet','Database');
```

## Evaluation signals

- Both workspace variables are MATLAB string arrays named exactly MSIdata and Database, not cell arrays or tables
- MSIdata's first column contains the measured m/z values from the preprocessed MSI dataset
- Database's first column contains metabolite names and subsequent columns contain theoretical m/z values in specified adduct forms
- The selected organ and detection mode (positive or negative mode) of Database match those of MSIdata to be annotated
- The contents conform to the input-file requirements illustrated by the example files MSIdata.xlsx and Database.xlsx

## Limitations

- The imported sheets must already satisfy the input-file column layout; users must refer to the provided example files to prepare correctly formatted sheets
- Only organ-specific databases covering brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum in positive (1298 metabolites) or negative (515 metabolites) mode are provided; other organs or modes require a user-supplied database
- A mismatch between database organ/detection mode and the MSIdata's mode will not be caught automatically — the user must perform the consistency check manually before running MSIannotator.m

## Evidence
