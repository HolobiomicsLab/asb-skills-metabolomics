---
name: spreadsheet-sheet-schema-verification
description: Use when a multi-sheet .xlsx database (e.g. an organ-specific metabolite
  database for MSIannotator) must be checked against a declared schema of sheet names,
  ion modes, and entry counts prior to annotation runs.
when_to_use_negative:
- The file is not a multi-sheet metabolite database but raw MSI data or an already-annotated
  output (Annotator) file
- Only a single preselected database sheet (e.g. 'Database.xlsx' liver-positive example)
  is needed and the full workbook schema is not being audited
- The workbook's intended sheet count, ion modes, or entry counts are not known in
  advance, so no expected schema exists to verify against
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: MSIannotator
  role: The annotation tool whose required Database input schema (metabolite names
    in first column, theoretical m/z in adduct-form columns; organ and detection mode
    consistent with MSIdata) drives the verification checks
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Environment in which the workbook sheets are loaded (readcell/readtable) and
    counts/organ-mode consistency are checked before MSIannotator.m is run
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:71bfa50ebec5edca51dd068816f5ce7f7518f457fafaf2da19e79fa2f480e2d7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spreadsheet-sheet-schema-verification

## Summary

Verifies that a multi-sheet metabolite database spreadsheet (here 'Organ specific metabolite database.xlsx') contains the expected number of entries per sheet and the expected sheet-to-organ/ion-mode assignment before it is used for MSI metabolite annotation.

## When to use

Use when a multi-sheet .xlsx database (e.g. an organ-specific metabolite database for MSIannotator) must be checked against a declared schema of sheet names, ion modes, and entry counts prior to annotation runs.

## When NOT to use

- The file is not a multi-sheet metabolite database but raw MSI data or an already-annotated output (Annotator) file
- Only a single preselected database sheet (e.g. 'Database.xlsx' liver-positive example) is needed and the full workbook schema is not being audited
- The workbook's intended sheet count, ion modes, or entry counts are not known in advance, so no expected schema exists to verify against

## Inputs

- Multi-sheet Excel workbook, e.g. 'Organ specific metabolite database.xlsx' with 11 sheets
- Expected schema description: sheet-to-organ mapping, ion modes, and expected entry counts (1298 positive, 515 negative)
- MSIdata to be annotated (optional reference for organ/mode consistency check)

## Outputs

- Per-sheet validation report (e.g. database_validation_report.csv) listing sheet names, ion modes, and observed vs expected metabolite counts
- Confirmation flag that Sheet1 = 1298 positive-mode and Sheet2 = 515 negative-mode metabolites and Sheets 3–11 match the organ/serum order

## How to apply

Load every sheet of the workbook (Sheet1–Sheet11 of 'Organ specific metabolite database.xlsx') into MATLAB via readcell/readtable as tabular data. Check that Sheet1 (positive mode) holds metabolite names in the first column with theoretical m/z in adduct-form columns and contains exactly 1298 metabolites, and that Sheet2 (negative mode) contains exactly 515. Confirm Sheets 3–11 map, in order, to brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum organ/serum-split databases, recording each sheet's row count. Also verify the organ and detection mode of the chosen sheet are consistent with the MSIdata being annotated, since MSIannotator requires this consistency before running. Compile per-sheet names, ion modes, and observed versus expected counts into a validation report (e.g. database_validation_report.csv).

## Related tools

- **MSIannotator** (The annotation tool whose required Database input schema (metabolite names in first column, theoretical m/z in adduct-form columns; organ and detection mode consistent with MSIdata) drives the verification checks) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Environment in which the workbook sheets are loaded (readcell/readtable) and counts/organ-mode consistency are checked before MSIannotator.m is run) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
sheets = readcell('Organ specific metabolite database.xlsx'); assert(height(sheets{1})==1298); assert(height(sheets{2})==515);
```

## Evaluation signals

- Sheet1 contains exactly 1298 metabolite entries (positive mode) and Sheet2 exactly 515 (negative mode)
- All nine organ/serum sheets (Sheets 3–11) are present and in the order brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum
- First column of each sheet holds metabolite names and remaining columns hold theoretical m/z per adduct form, matching MSIannotator's Database input requirements
- The organ and detection mode of the selected sheet match those of the MSIdata to be annotated
- Validation report rows and columns are complete for all 11 sheets with observed vs expected counts

## Limitations

- The README does not provide a formal schema declaration or changelog, so expected counts and sheet order are taken from the article/README prose rather than a machine-readable schema
- If the m/z error range or adduct assumptions differ (users may reset the default 10 ppm in line 25), schema verification alone does not guarantee correct annotation results
- Verification assumes the workbook structure described in the README; user-supplied custom databases may have a different sheet organization that must be checked separately

## Evidence

- [methods] Count the metabolite entries in Sheet1 and Sheet2 and check they equal 1298 (positive mode) and 515 (negative mode) respectively: [evidence span withheld — anchor status: not_found]
