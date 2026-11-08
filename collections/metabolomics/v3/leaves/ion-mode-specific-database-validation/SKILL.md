---
name: ion-mode-specific-database-validation
description: Use when annotating features in a preprocessed AFADESI-MSI dataset with
  MSIannotator and you must confirm that the imported Database (from Organ specific
  metabolite database.xlsx, e.g. Sheet4 = liver) matches the detection mode of MSIdata
  before running MSIannotator.m.
when_to_use_negative:
- When running annotation on already-validated, organ- and mode-matched Database.xlsx
  and MSIdata.xlsx example inputs — the check is already satisfied and adds no information.
- When the goal is to build or expand the metabolite adduct-ion database itself (enumerating
  adduct forms and computing theoretical m/z) rather than to validate an existing
  Database for annotation.
- When the input is a metabolite identification from LC-MS/MS rather than an AFADESI-MSI
  dataset, since the adduct forms and ionization behavior differ between LC-MS and
  AFADESI-MSI.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MSIannotator
  role: MATLAB script whose Database input must be validated for organ- and mode-consistency
    before the m/z error matching (default 10 ppm) is run.
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: The script (line 25) in which the m/z error threshold is applied after the
    database validation check; the validation ensures this matching step operates
    on a correctly matched Database.
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:7f480712dbdbb6564d5b4023998d52e82f6c9056bee702dcd965c3c761010f45
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ion-mode-specific-database-validation

## Summary

A validation step that ensures the metabolite database selected for MSIannotator annotation matches both the organ being imaged and the ionization detection mode (positive or negative) of the preprocessed MSI dataset, preventing false metabolite assignments from mode-organ mismatched adduct-ion tables.

## When to use

Use when annotating features in a preprocessed AFADESI-MSI dataset with MSIannotator and you must confirm that the imported Database (from Organ specific metabolite database.xlsx, e.g. Sheet4 = liver) matches the detection mode of MSIdata before running MSIannotator.m.

## When NOT to use

- When running annotation on already-validated, organ- and mode-matched Database.xlsx and MSIdata.xlsx example inputs — the check is already satisfied and adds no information.
- When the goal is to build or expand the metabolite adduct-ion database itself (enumerating adduct forms and computing theoretical m/z) rather than to validate an existing Database for annotation.
- When the input is a metabolite identification from LC-MS/MS rather than an AFADESI-MSI dataset, since the adduct forms and ionization behavior differ between LC-MS and AFADESI-MSI.

## Inputs

- Preprocessed MSI dataset imported into MATLAB as string array 'MSIdata' (measured m/z in the first column)
- Candidate metabolite database sheet (e.g. from Organ specific metabolite database.xlsx — Sheet3–Sheet11 for brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum; Sheet1 positive mode, Sheet2 negative mode)
- Knowledge of the detection mode (positive or negative) of the MSIdata to be annotated

## Outputs

- Verified Database string array whose organ and detection mode match MSIdata, ready for annotation with MSIannotator.m
- Explicit rejection/swap decision if the imported Database sheet does not match the MSIdata organ or mode

## How to apply

Before executing MSIannotator.m, verify that the sheet imported as Database comes from the organ-specific metabolite database partition consistent with the imaged organ (e.g. liver database for liver MSI data, brain for brain) and the same detection mode as MSIdata, since the database contains separate positive-mode (Sheet1, 1298 metabolites) and negative-mode (Sheet2, 515 metabolites) expansions totaling 27,407 adduct ions across brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum sheets. If the organ or ionization mode of the Database is inconsistent with MSIdata, swap in the correct sheet from Organ specific metabolite database.xlsx or Database.xlsx before annotation. This check precedes the m/z error matching step (default 10 ppm at line 25: if abs(theor-mea)/theor<10·10^(-6)) and is the precondition that makes the theoretical m/z adduct-ion matching meaningful, because an organ- or mode-mismatched Database would match wrong adduct-ion m/z values.

## Related tools

- **MSIannotator** (MATLAB script whose Database input must be validated for organ- and mode-consistency before the m/z error matching (default 10 ppm) is run.) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator.m** (The script (line 25) in which the m/z error threshold is applied after the database validation check; the validation ensures this matching step operates on a correctly matched Database.) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
In MATLAB: msidata = readtable('MSIdata.xlsx'); database = readtable('Database.xlsx'); % verify organ & mode match before: msiannotator(msidata, database)
```

## Evaluation signals

- Confirm that the sheet imported as Database is from the same organ partition (brain, liver, kidney, heart, spleen, lung, muscle, pancreas, or serum) as the imaged tissue in MSIdata.
- Confirm that the Database detection mode matches the MSIdata detection mode (positive-mode Sheet1/organ sheets vs negative-mode Sheet2/organ sheets).
- After validation, run MSIannotator.m with default 10 ppm threshold (abs(theor-mea)/theor<10·10^(-6)) and confirm the resulting Annotator output contains annotations only for metabolites plausibly present in that organ.
- Cross-check that the annotated metabolite list uses the correct adduct-form columns from Database.xlsx for the validated organ-mode combination.

## Limitations

- The m/z error tolerance default is 10 ppm and must be manually edited in line 25 of MSIannotator.m if a different tolerance is needed, so the validation must anticipate user-modified thresholds.
- If the organ or detection mode of the Database is mismatched to MSIdata, the annotation results become unreliable even if a low ppm match is achieved, because theoretical m/z values for the wrong adduct ions would be compared.
- The validation only applies to the organ-specific database supplied with MSIannotator; if the user supplies a custom Database, the organ-mode consistency check must be applied manually against that Database's contents.

## Evidence
