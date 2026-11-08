---
name: ppm-error-tolerance-matching
description: Use when a preprocessed mass spectrometry imaging dataset (e.g. MSIdata.xlsx
  with measured m/z in the first column) must be annotated against a metabolite database
  by an m/z error tolerance expressed in ppm.
when_to_use_negative:
- The dataset is raw, unprocessed MSI data rather than a preprocessed m/z feature
  list with a measured m/z column.
- The database organ or detection mode does not match the MSIdata being annotated
  — the README explicitly requires consistency before running the tolerance match.
- The input already contains metabolite annotations; the skill is only for m/z-to-metabolite/adduct
  matching, not for validating or re-annotating existing labels.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MSIannotator.m
  role: MATLAB script that performs the ppm-error-tolerance matching between measured
    m/z in MSIdata and theoretical m/z in Database; line 25 sets the tolerance (default
    10 ppm).
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which MSIdata and Database string arrays are imported
    and MSIannotator.m is executed to generate the Annotator output file.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:6c87cd8d84dd27963ca459c04d2abc828e6627619051e012388fe8c4c97876bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ppm-error-tolerance-matching

## Summary

Matches measured m/z features in a preprocessed MSI dataset against theoretical m/z values from an organ-specific metabolite database using a relative-error criterion (default 10 ppm, configurable to e.g. 5 ppm by editing the MATLAB tolerance line), producing an annotated metabolite list with adduct forms.

## When to use

Use when a preprocessed mass spectrometry imaging dataset (e.g. MSIdata.xlsx with measured m/z in the first column) must be annotated against a metabolite database by an m/z error tolerance expressed in ppm.

## When NOT to use

- The dataset is raw, unprocessed MSI data rather than a preprocessed m/z feature list with a measured m/z column.
- The database organ or detection mode does not match the MSIdata being annotated — the README explicitly requires consistency before running the tolerance match.
- The input already contains metabolite annotations; the skill is only for m/z-to-metabolite/adduct matching, not for validating or re-annotating existing labels.

## Inputs

- MSIdata: preprocessed MSI dataset as a string array, with measured m/z values in the first column (e.g. MSIdata.xlsx, liver tissue in positive mode)
- Database: metabolite database as a string array, with metabolite names in the first column and theoretical m/z values in the specified adduct-form columns (e.g. Database.xlsx, liver in positive mode; organ-specific database sheets Sheet1–Sheet11)

## Outputs

- Annotator: an annotated metabolite list with measured m/z in the first column, and intermediate and ultimate annotated metabolite names with their adduct forms in the following columns

## How to apply

Import the preprocessed MSI dataset and the selected organ/mode database sheet into MATLAB as string arrays named MSIdata and Database, verifying that the database's organ and detection mode (positive or negative) match the data. For each measured m/z in MSIdata's first column, compute the relative error abs(theor-mea)/theor against each theoretical m/z in the Database adduct columns and accept the match when it falls below the threshold 10*10^(-6) (10 ppm); if tighter confidence is needed, edit line 25 of MSIannotator.m — e.g. changing the literal to 5·10^(-6) for a 5 ppm window. Record for every accepted match the annotated metabolite name and its adduct form, then save the results to the output file Annotator with measured m/z in the first column and annotations in the following columns. The ppm-based relative error is used instead of an absolute Da window because it scales the tolerance with the m/z magnitude, which is the standard behavior for m/z annotation of adduct ions.

## Related tools

- **MSIannotator.m** (MATLAB script that performs the ppm-error-tolerance matching between measured m/z in MSIdata and theoretical m/z in Database; line 25 sets the tolerance (default 10 ppm).) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which MSIdata and Database string arrays are imported and MSIannotator.m is executed to generate the Annotator output file.)

## Examples

```
Carry out the code (MSIannotator.m) in MATLAB after importing MSIdata and Database string arrays
```

## Evaluation signals

- Every annotation in the Annotator file satisfies the configured relative-error criterion: abs(theor-mea)/theor < 10·10^(-6) (or the user-set threshold, e.g. 5·10^(-6)).
- The first column of the Annotator file contains only m/z values present in the first column of the input MSIdata.
- Each reported annotation includes both an intermediate/ultimate metabolite name and its adduct form.
- The organ and detection mode of the Database used match those of MSIdata (e.g. liver, positive mode) before matching begins.
- Changing the tolerance literal in line 25 (e.g. to 5 ppm) changes the number of accepted matches in Annotator accordingly — a tighter threshold yields fewer or equal annotations.

## Limitations

- The default m/z error range of 10 ppm is a single hardcoded literal in line 25 of MSIannotator.m; changing it requires editing the script rather than passing a parameter.
- Tolerance matching alone does not confirm identification — the 10 ppm window accepts any database entry within the window, so organ/mode-correct database selection is essential to avoid false annotations.
- The skill assumes the database sheet follows the required layout (metabolite names in the first column, theoretical m/z in the specified adduct-form columns); deviations break the matching.

## Evidence
