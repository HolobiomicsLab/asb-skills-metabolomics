---
name: metabolite-database-structure-validation
description: Use when downloading the Yingzhu96/MSIannotator metabolite database or
  preparing a custom Database sheet, to confirm sheet count, per-sheet metabolite
  counts, and organ/mode assignments before running MSIannotator.m.
when_to_use_negative:
- The Database sheet has already been loaded and validated, and MSIannotator.m is
  ready to run — proceed directly to annotation rather than re-validating.
- You are building a user-supplied database from scratch rather than checking the
  authors' deposited 'Organ specific metabolite database.xlsx'; the expected-count
  invariants (1298/515) do not apply.
- The task is annotating features in MSIdata (measured m/z matching) rather than verifying
  the database file's structure.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: MSIannotator
  role: Supplies the 'Organ specific metabolite database.xlsx' whose structure is
    validated; the validated Database sheet is then used as input for metabolite/adduct
    annotation of MSIdata.
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Environment for loading the workbook sheets (readcell/readtable), counting
    metabolite entries per sheet, and generating the validation report.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:89da49be19e5cbb86befc5f56588695329c0c481c09af0e381b283beb0d85984
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# metabolite-database-structure-validation

## Summary

Validates that the 'Organ specific metabolite database.xlsx' shipped with MSIannotator matches its declared layout: 1298 metabolites in positive mode (Sheet1), 515 in negative mode (Sheet2), and nine organ/serum-split sheets (Sheets 3-11: brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum). This guards downstream metabolite/adduct annotation against loading a malformed or mis-mapped database.

## When to use

Use when downloading the Yingzhu96/MSIannotator metabolite database or preparing a custom Database sheet, to confirm sheet count, per-sheet metabolite counts, and organ/mode assignments before running MSIannotator.m.

## When NOT to use

- The Database sheet has already been loaded and validated, and MSIannotator.m is ready to run — proceed directly to annotation rather than re-validating.
- You are building a user-supplied database from scratch rather than checking the authors' deposited 'Organ specific metabolite database.xlsx'; the expected-count invariants (1298/515) do not apply.
- The task is annotating features in MSIdata (measured m/z matching) rather than verifying the database file's structure.

## Inputs

- 'Organ specific metabolite database.xlsx' from the Yingzhu96/MSIannotator repository (11-sheet workbook: Sheet1 positive, Sheet2 negative, Sheets 3-11 organ/serum-split databases)
- Expected structure specification: 1298 positive-mode metabolites, 515 negative-mode metabolites, organ order brain/liver/kidney/heart/spleen/lung/muscle/pancreas/serum

## Outputs

- database_validation_report.csv with per-sheet names, ion modes, and observed versus expected metabolite counts
- Pass/fail verdict per sheet (Sheet1=1298, Sheet2=515, Sheets 3-11 present with expected organ assignment)

## How to apply

Load 'Organ specific metabolite database.xlsx' in MATLAB with readcell/readtable so all 11 sheets are accessible as tabular data. Count entries in Sheet1 and Sheet2 and confirm they equal 1298 (positive mode) and 515 (negative mode); verify each sheet carries metabolite names in the first column and theoretical m/z values in adduct-form columns, as the README specifies for a Database input. Check Sheets 3-11 map to brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum in that order, recording each sheet's row count. Before annotating, also confirm the selected organ and detection mode are consistent with the MSIdata to be annotated, per the README's input-consistency check. Write per-sheet names, ion modes, and observed-versus-expected counts into database_validation_report.csv; accept the output only when Sheet1 = 1298, Sheet2 = 515, and all nine organ/serum sheets are present with the expected organ assignment.

## Related tools

- **MSIannotator** (Supplies the 'Organ specific metabolite database.xlsx' whose structure is validated; the validated Database sheet is then used as input for metabolite/adduct annotation of MSIdata.) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Environment for loading the workbook sheets (readcell/readtable), counting metabolite entries per sheet, and generating the validation report.)

## Examples

```
sheets = readcell('Organ specific metabolite database.xlsx','Sheet',1:11); assert(height(sheets{1})==1298 && height(sheets{2})==515)
```

## Evaluation signals

- Sheet1 contains exactly 1298 metabolites and Sheet2 exactly 515; any deviation fails the check.
- Sheets 3-11 exist and correspond, in order, to brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum databases.
- Every sheet has metabolite names in the first column and theoretical m/z in adduct-form columns, matching the declared Database input format.
- database_validation_report.csv records observed vs expected counts for all 11 sheets with no missing sheets.
- The organ and detection mode of the chosen Database sheet are consistent with the MSIdata to be annotated before annotation proceeds.

## Limitations

- The validation only checks the deposited database's structure and counts; it does not verify metabolite identities or theoretical m/z values against external references.
- It assumes the specific workbook layout of the Yingzhu96/MSIannotator repository; a user-supplied Database sheet with a different sheet order or naming will not match the 1298/515 and Sheets 3-11 invariants.
- The m/z error threshold used downstream (default 10 ppm, modifiable at line 25 of MSIannotator.m) is a separate setting and is not validated by this skill.

## Evidence
