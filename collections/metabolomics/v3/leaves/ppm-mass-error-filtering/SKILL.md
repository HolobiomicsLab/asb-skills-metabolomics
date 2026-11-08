---
name: ppm-mass-error-filtering
description: Use when annotating preprocessed MSI features against a metabolite database
  and you need a confidence criterion for m/z-based metabolite/adduct matches, e.g.
  deciding which theoretical adduct ions match a measured peak.
when_to_use_negative:
- The input is raw, unprocessed MSI data rather than a preprocessed peak/m/z table
  with measured m/z in the first column.
- The task requires LC-MS/MS-level identification or MS/MS spectral matching — this
  filter relies solely on precursor m/z agreement against a database of 27,407 adduct
  ions and does not use product-ion evidence.
- The database organ or ionization mode does not match the MSI data (e.g. a negative-mode
  database sheet against positive-mode liver data); the ppm filter cannot compensate
  for a mismatched Database.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MSIannotator.m
  role: MATLAB script that applies the ppm error filter (line 25) to match measured
    MSI m/z against theoretical adduct m/z from the Database
  repo: https://github.com/Yingzhu96/MSIannotator
- name: MATLAB
  role: Runtime environment in which MSIdata and Database string arrays are loaded
    and MSIannotator.m is executed
- name: MSIannotator
  role: Automated annotation tool whose m/z matching step implements the relative
    ppm error threshold
  repo: https://github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:75981a51ffac7eb7a249576d74cceb9d5b674be3bb254e0b99a3eb3de59c2784
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Yingzhu96/MSIannotator
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ppm-mass-error-filtering

## Summary

Relative m/z error filtering criterion used by MSIannotator.m to accept or reject metabolite/adduct matches between measured m/z from AFADESI-MSI data and theoretical adduct m/z in an organ-specific metabolite database; default threshold is 10 ppm, adjustable at line 25.

## When to use

Use when annotating preprocessed MSI features against a metabolite database and you need a confidence criterion for m/z-based metabolite/adduct matches, e.g. deciding which theoretical adduct ions match a measured peak.

## When NOT to use

- The input is raw, unprocessed MSI data rather than a preprocessed peak/m/z table with measured m/z in the first column.
- The task requires LC-MS/MS-level identification or MS/MS spectral matching — this filter relies solely on precursor m/z agreement against a database of 27,407 adduct ions and does not use product-ion evidence.
- The database organ or ionization mode does not match the MSI data (e.g. a negative-mode database sheet against positive-mode liver data); the ppm filter cannot compensate for a mismatched Database.

## Inputs

- MSIdata — preprocessed MSI dataset as a MATLAB string array, measured m/z in the first column (e.g. MSIdata.xlsx, liver tissue in positive mode)
- Database — organ-specific metabolite database as a MATLAB string array, metabolite names in the first column and theoretical m/z in specified adduct form columns (e.g. Database.xlsx, liver positive mode)

## Outputs

- Annotator — annotated metabolite list with measured m/z in the first column, followed by intermediate and ultimately annotated metabolite names and their adduct forms (saved as Annotator.xlsx or Annotator.csv)

## How to apply

Inside MSIannotator.m, each measured m/z (first column of the MSIdata string array) is compared with theoretical adduct m/z values from the Database string array using the criterion abs(theor-mea)/theor < 10*10^(-6), i.e. a 10 ppm relative error window implemented at line 25. To tighten or loosen matching, rewrite line 25 — e.g. 'if abs(theor-mea)/theor<5·10^(-6)' sets a 5 ppm window. The rationale is that a relative (ppm) rather than absolute error keeps the window proportional across the mass range, and the default 10 ppm balances confidence against the accuracy of AFADESI-MSI m/z measurements. Matches passing the filter populate the Annotator output with measured m/z plus intermediate and ultimate annotated metabolite names and their adduct forms. Before running, verify that the organ (e.g. liver) and detection mode (positive/negative) of the Database sheet are consistent with the MSIdata, since the filter only controls mass accuracy, not database relevance.

## Related tools

- **MSIannotator.m** (MATLAB script that applies the ppm error filter (line 25) to match measured MSI m/z against theoretical adduct m/z from the Database) — https://github.com/Yingzhu96/MSIannotator
- **MATLAB** (Runtime environment in which MSIdata and Database string arrays are loaded and MSIannotator.m is executed)
- **MSIannotator** (Automated annotation tool whose m/z matching step implements the relative ppm error threshold) — https://github.com/Yingzhu96/MSIannotator

## Examples

```
if abs(theor-mea)/theor<10*10^(-6) %m/z error set as 10 ppm  % line 25 of MSIannotator.m; change to 5*10^(-6) for a 5 ppm window
```

## Evaluation signals

- Every annotated row in the Annotator output satisfies abs(theor-mea)/theor < 10*10^(-6) against the matched theoretical adduct m/z in the Database.
- The Annotator file exists and is non-empty, with measured m/z in the first column and intermediate plus ultimate annotated metabolite/adduct columns present.
- Each reported adduct ion's theoretical m/z, recomputed from the Database, falls within 10 ppm of the measured m/z listed in the same row.
- Lowering the threshold at line 25 (e.g. to 5 ppm) monotonically reduces the number of accepted annotations, confirming the filter is active.
- The Database organ sheet (e.g. liver, Sheet4) and detection mode (positive, Sheet1) match the annotated MSIdata before filtering.

## Limitations

- The m/z error range is a single relative-ppm criterion with no adduct-form or isotope plausibility checks, so multiple adducts of the same nominal mass can be accepted within the window.
- The default 10 ppm threshold is hard-coded at line 25 and must be manually edited (e.g. to 5 ppm) if a different tolerance is needed.
- Filtering alone cannot rescue an incorrect Database: the organ and detection mode must be manually checked for consistency with the MSIdata.

## Evidence
