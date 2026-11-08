---
name: msi-metabolite-annotation
description: Use when you have preprocessed MSI data with a measured m/z column and
  need to assign metabolite names and adduct forms by matching against an organ- and
  ion-mode-specific metabolite database.
when_to_use_negative:
- The input is raw, unprocessed MSI data — the tool annotates processed MSI data only
- The metabolite database's organ or ion mode does not match the MSI dataset to be
  annotated (e.g., a liver positive-mode database against kidney negative-mode data)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MSIannotator.m
  role: MATLAB script that performs the m/z matching against the Database using the
    line-25 ppm threshold and generates the Annotator output
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Execution environment in which MSIdata and Database are loaded as string arrays
    and MSIannotator.m is run
- name: MSIannotator
  role: Automated annotation tool that conducts metabolite annotation in the MSI dataset
    with high efficiency and confidence
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:4558703da7ee9db5877752a14711338383f85c49d6a28825f4d545dcbe880e3b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msi-metabolite-annotation

## Summary

Annotate metabolites and adduct forms in preprocessed ambient mass spectrometry imaging (MSI) data using the MATLAB script MSIannotator.m, which matches measured m/z values against an organ-specific, mode-specific metabolite database within a default 10 ppm m/z error range.

## When to use

Use when you have preprocessed MSI data with a measured m/z column and need to assign metabolite names and adduct forms by matching against an organ- and ion-mode-specific metabolite database.

## When NOT to use

- The input is raw, unprocessed MSI data — the tool annotates processed MSI data only
- The metabolite database's organ or ion mode does not match the MSI dataset to be annotated (e.g., a liver positive-mode database against kidney negative-mode data)

## Inputs

- MSIdata: preprocessed MSI dataset as a MATLAB string array with measured m/z in the first column (e.g., MSIdata.xlsx, liver tissue in positive mode)
- Database: metabolite database as a MATLAB string array with metabolite names in the first column and theoretical m/z in specified adduct-form columns (e.g., Database.xlsx, liver positive mode)
- Organ specific metabolite database.xlsx: 1298 metabolites in positive mode (Sheet1), 515 in negative mode (Sheet2), organ-specific sheets for brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum

## Outputs

- Annotator: annotated metabolite list with measured m/z in the first column, intermediate and ultimate annotated metabolite names, and their adduct forms (saved as Annotator.xlsx or Annotator.csv)

## How to apply

Import two sheets into MATLAB as string arrays named MSIdata (measured m/z in the first column) and Database (metabolite names in the first column, theoretical adduct m/z in the specified adduct columns). Verify consistency: the Database's selected organ (e.g., liver) and detection mode (positive or negative) must match the MSIdata to be annotated. Run MSIannotator.m, which applies the default 10 ppm threshold at line 25 via 'abs(theor-mea)/theor<10*10^(-6)'; modify line 25 if a different error range (e.g., 5 ppm) is required. Save the generated Annotator output, which contains measured m/z in the first column followed by intermediate and ultimately annotated metabolite names and their adduct forms. For organ-specific work, use the matching sheet of Organ specific metabolite database.xlsx (1298 metabolites in positive mode Sheet1, 515 in negative mode Sheet2; brain, liver, kidney, heart, spleen, lung, muscle, pancreas, serum in Sheets 3–11).

## Related tools

- **MSIannotator.m** (MATLAB script that performs the m/z matching against the Database using the line-25 ppm threshold and generates the Annotator output) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Execution environment in which MSIdata and Database are loaded as string arrays and MSIannotator.m is run)
- **MSIannotator** (Automated annotation tool that conducts metabolite annotation in the MSI dataset with high efficiency and confidence) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
matlab -batch "run('MSIannotator.m')"  % after loading MSIdata and Database as string arrays; default 10 ppm threshold at line 25
```

## Evaluation signals

- The Annotator output file exists and is non-empty, with measured m/z in the first column followed by intermediate and ultimate annotated metabolite/adduct columns
- Every annotation satisfies the m/z error criterion abs(theor-mea)/theor < 10*10^(-6) (or the user-modified threshold set at line 25)
- Each annotated row's metabolite adduct ion m/z matches its measured m/z within the configured ppm tolerance
- The Database organ and detection mode (e.g., liver, positive) are confirmed consistent with the MSIdata before running
- MSIdata and Database are imported as MATLAB string arrays with the required first-column contents (measured m/z; metabolite names)

## Limitations

- The default m/z error range of 10 ppm must be manually modified in line 25 of MSIannotator.m if a different tolerance is needed
- Annotations are only as good as the supplied database — organ and detection mode must be consistent with the MSIdata, and the tool relies on the established organ-specific database (1298 positive-mode / 515 negative-mode metabolites) or a user-supplied equivalent
- No changelog is available for the source repository

## Evidence
