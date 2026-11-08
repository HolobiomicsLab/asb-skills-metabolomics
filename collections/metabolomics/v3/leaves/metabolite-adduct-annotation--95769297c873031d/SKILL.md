---
name: metabolite-adduct-annotation
description: Use when you have a preprocessed MSI dataset (measured m/z in the first
  column) and an organ/mode-matched metabolite database with theoretical adduct m/z
  values, and need metabolite/adduct annotations.
when_to_use_negative:
- The MSI dataset has not yet been preprocessed (peak picking/background removal not
  done), since MSIannotator expects processed m/z features rather than raw imaging
  spectra.
- The supplied Database organ or detection mode does not match the MSIdata being annotated,
  e.g. using the liver positive-mode database against a kidney negative-mode acquisition.
- The goal is structural confirmation or metabolite identification beyond nominal-mass
  matching — this tool only assigns names and adducts by m/z error, not by MS/MS spectra.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MSIannotator
  role: Automated annotation tool that matches measured m/z features in the MSI dataset
    against a metabolite database and outputs annotated metabolites with adduct forms
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MSIannotator.m
  role: 'MATLAB script whose core tolerance logic (line 25: abs(theor-mea)/theor <
    10*10^(-6)) performs the m/z-to-metabolite matching'
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which MSIdata and Database string arrays are loaded
    and MSIannotator.m is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:bffccd42fa6e3a907e8706f2e2799d509fb1a22a96bdf4c4aaab840fca65d543
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# metabolite-adduct-annotation

## Summary

A MATLAB-based annotation skill (MSIannotator) that matches measured m/z features from preprocessed mass spectrometry imaging data against an organ- and mode-specific metabolite database within a configurable ppm tolerance, yielding metabolite names and their adduct forms.

## When to use

Use when you have a preprocessed MSI dataset (measured m/z in the first column) and an organ/mode-matched metabolite database with theoretical adduct m/z values, and need metabolite/adduct annotations.

## When NOT to use

- The MSI dataset has not yet been preprocessed (peak picking/background removal not done), since MSIannotator expects processed m/z features rather than raw imaging spectra.
- The supplied Database organ or detection mode does not match the MSIdata being annotated, e.g. using the liver positive-mode database against a kidney negative-mode acquisition.
- The goal is structural confirmation or metabolite identification beyond nominal-mass matching — this tool only assigns names and adducts by m/z error, not by MS/MS spectra.

## Inputs

- MSIdata: preprocessed MSI dataset as a MATLAB string array, with measured m/z in the first column (e.g. MSIdata.xlsx, liver tissue in positive mode)
- Database: metabolite database as a MATLAB string array, with metabolite names in the first column and theoretical m/z in specified adduct-form columns (e.g. Database.xlsx, liver in positive mode)
- Organ specific metabolite database.xlsx: 1298 metabolites in positive mode (Sheet1) and 515 metabolites in negative mode (Sheet2), subdivided per organ/serum in Sheets 3-11

## Outputs

- Annotator: annotated metabolite list with measured m/z in the first column and intermediate and ultimate annotated metabolite names plus their adduct forms in subsequent columns

## How to apply

Import the preprocessed MSI sheet and the metabolite database sheet into MATLAB as string arrays named MSIdata and Database; MSIdata must have measured m/z in the first column, and Database must list metabolite names in the first column with theoretical adduct m/z in subsequent columns. Verify that the selected organ (e.g. brain, liver, kidney) and detection mode (positive or negative) in the Database are consistent with the MSIdata being annotated. Run MSIannotator.m, which computes the relative m/z error as abs(theor-mea)/theor and accepts matches below the default tolerance of 10*10^(-6) (10 ppm); if a tighter window is needed, modify line 25 of the script (e.g. to 5*10^(-6) for 5 ppm). Collect the matched intermediate and ultimate metabolite names together with their adduct forms and save them to the output file Annotator, with measured m/z retained in the first column.

## Related tools

- **MSIannotator** (Automated annotation tool that matches measured m/z features in the MSI dataset against a metabolite database and outputs annotated metabolites with adduct forms) — https://github.com/Yingzhu96/MSIannotator
- **MSIannotator.m** (MATLAB script whose core tolerance logic (line 25: abs(theor-mea)/theor < 10*10^(-6)) performs the m/z-to-metabolite matching) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which MSIdata and Database string arrays are loaded and MSIannotator.m is executed)

## Examples

```
MSIdata = readtable('MSIdata.xlsx'); Database = readtable('Database.xlsx'); MSIannotator
```

## Evaluation signals

- Every reported annotation satisfies the configured ppm criterion: abs(theoretical m/z - measured m/z)/theoretical m/z < 10*10^(-6) (default 10 ppm) or the user-set value.
- The Annotator output preserves the measured m/z values from the first column of MSIdata in its own first column.
- The Database used shares the same organ and detection mode (positive or negative) as the MSIdata being annotated.
- Each annotated entry includes both the metabolite name and its specific adduct form, not just a metabolite name.
- A 10 ppm default tolerance yields matches that remain valid when the tolerance is tightened to 5 ppm only for the subset of features still accepted (no new annotations appear at the stricter threshold).

## Limitations

- The default 10 ppm m/z error window may be too loose or too tight for the instrument's actual mass accuracy; users must edit line 25 of MSIannotator.m to change it.
- Annotation relies solely on m/z matching against a supplied database — it does not use MS/MS spectral confirmation and inherits database completeness limits (e.g. 1298 positive-mode / 515 negative-mode metabolites in the provided organ database).
- Correct output depends on the user verifying that the Database organ and detection mode are consistent with MSIdata; mismatched inputs produce misleading annotations.

## Evidence
