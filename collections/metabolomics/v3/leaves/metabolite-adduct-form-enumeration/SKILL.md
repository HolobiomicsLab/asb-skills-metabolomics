---
name: metabolite-adduct-form-enumeration
description: Use when constructing a metabolite database for mass spectrometry imaging
  that must contain the adduct ions actually formed under AFADESI-MSI rather than
  the LC-MS adduct forms, given a metabolite list per organ and ionization mode.
when_to_use_negative:
- Input is already a preprocessed MSI dataset with measured m/z values ready for annotation
  (MSIdata.xlsx format) - use MSIannotator.m annotation instead of database enumeration
- The goal is to match measured MSI m/z features against an existing database within
  a ppm tolerance rather than to construct or expand the database itself
- Adduct forms have not yet been evaluated for the LC-MS vs AFADESI-MSI ionization
  regime, so the adduct set to enumerate is unknown
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MATLAB
  role: Used to enumerate adduct ions per organ sheet and per mode and to run downstream
    annotation of MSIdata against the enumerated Database within the m/z error range
- name: MSIannotator.m
  role: MATLAB script that consumes the enumerated Database (metabolite names plus
    theoretical m/z in specified adduct forms) to annotate measured m/z features in
    MSIdata, with the default 10 ppm m/z error range set on line 25
  repo: github.com/Yingzhu96/MSIannotator
- name: MSIannotator
  role: Automated annotation tool created to conduct metabolite annotation in the
    MSI dataset with high efficiency and confidence using the adduct-ion-expanded
    database
  repo: github.com/Yingzhu96/MSIannotator
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c00557
    title: An Organ-Specific Metabolite Annotation Approach for Ambient Mass Spectrometry
      Imaging Reveals Spatial Metabolic Alterations of a Whole Mouse Body
  source_content_hash: sha256:5245b7ac660abed6172dd48abb313eba32558837ad94ac0032cf893d0d8ad3d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MATLAB
  - MSIannotator
  - MSIannotator.m
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# metabolite-adduct-form-enumeration

## Summary

Enumerates the theoretical m/z of every metabolite-adduct combination observed for AFADESI-MSI (in contrast to LC-MS adduct forms) across organ-specific metabolite lists, expanding 1298 positive-mode and 515 negative-mode metabolites into a 27,407-ion adduct database.

## When to use

Use when constructing a metabolite database for mass spectrometry imaging that must contain the adduct ions actually formed under AFADESI-MSI rather than the LC-MS adduct forms, given a metabolite list per organ and ionization mode.

## When NOT to use

- Input is already a preprocessed MSI dataset with measured m/z values ready for annotation (MSIdata.xlsx format) - use MSIannotator.m annotation instead of database enumeration
- The goal is to match measured MSI m/z features against an existing database within a ppm tolerance rather than to construct or expand the database itself
- Adduct forms have not yet been evaluated for the LC-MS vs AFADESI-MSI ionization regime, so the adduct set to enumerate is unknown

## Inputs

- Organ specific metabolite database.xlsx - organ-specific metabolite lists (1298 metabolites in positive mode Sheet1; 515 metabolites in negative mode Sheet2), divided by brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum sheets
- Adduct form assignments from the evaluation of the adduct form difference of metabolite ions between LC-MS and AFADESI-MSI

## Outputs

- adduct_ion_database.csv - metabolite name, adduct form, theoretical m/z, organ, and ionization mode columns; total row count must equal 27,407 adduct ions
- Database.xlsx-style metabolite database with metabolite names in the first column and theoretical m/z in the specified adduct forms columns

## How to apply

Load the organ-specific metabolite lists from 'Organ specific metabolite database.xlsx', which contains 1298 metabolites in positive mode (Sheet1) and 515 metabolites in negative mode (Sheet2), divided into brain, liver, kidney, heart, spleen, lung, muscle, pancreas, and serum sheets. For each organ sheet and ionization mode, enumerate the metabolite adduct forms observed for AFADESI-MSI, following the evaluation of the adduct form difference of metabolite ions between LC-MS and airflow-assisted desorption electrospray ionization (AFADESI)-MSI. Compute the theoretical m/z for every metabolite-adduct combination, paralleling the theoretical m/z columns used in Database.xlsx, and aggregate the enumerated adduct ions per organ sheet and per mode with pandas/MATLAB. Sum across all sheets to obtain the total adduct-ion count, which must equal the reported 27,407 adduct ions. Write the expanded database to adduct_ion_database.csv with columns for metabolite name, adduct form, theoretical m/z, organ, and ionization mode; downstream annotation (MSIannotator.m) then matches measured m/z against these theoretical values within a 10 ppm default error range (abs(theor-mea)/theor < 10*10^(-6)).

## Related tools

- **MATLAB** (Used to enumerate adduct ions per organ sheet and per mode and to run downstream annotation of MSIdata against the enumerated Database within the m/z error range)
- **MSIannotator.m** (MATLAB script that consumes the enumerated Database (metabolite names plus theoretical m/z in specified adduct forms) to annotate measured m/z features in MSIdata, with the default 10 ppm m/z error range set on line 25) — github.com/Yingzhu96/MSIannotator
- **MSIannotator** (Automated annotation tool created to conduct metabolite annotation in the MSI dataset with high efficiency and confidence using the adduct-ion-expanded database) — github.com/Yingzhu96/MSIannotator

## Examples

```
MSIannotator.m % annotate MSIdata features against the enumerated Database; line 25: if abs(theor-mea)/theor<10*10^(-6) % m/z error set as 10 ppm
```

## Evaluation signals

- Total row count of adduct_ion_database.csv equals 27,407 adduct ions, matching the reported metabolite database for MSI
- Positive-mode metabolite count totals 1298 (Sheet1) and negative-mode totals 515 (Sheet2) before adduct expansion
- Every enumerated adduct row has a theoretical m/z value consistent with the metabolite-adduct combination calculation, as in the theoretical m/z columns of Database.xlsx
- Downstream MSIannotator.m annotation succeeds when the selected organ and detection mode of the Database are consistent with the MSIdata to be annotated
- m/z matching against the enumerated database uses the default 10 ppm error range (abs(theor-mea)/theor < 10*10^(-6)) unless the threshold is deliberately modified on line 25

## Limitations

- The database is organ- and mode-specific: the selected organ and detection mode (positive or negative) of the Database must be consistent with the MSIdata to be annotated, otherwise annotation is invalid
- The default m/z error range is 10 ppm; users must modify line 25 of MSIannotator.m if a different tolerance (e.g. 5 ppm) is needed
- Adduct forms enumerated here reflect the AFADESI-MSI ionization regime; adduct distributions differ from LC-MS, so LC-MS adduct lists cannot be reused directly

## Evidence
