---
name: spectral-library-matching
description: Use when you need to validate in silico (CFM-ID) fragment predictions
  for DNA adducts or other small molecules against experimental tandem HRMS/MS spectra
  of reference standards before using predicted fragments for annotation.
when_to_use_negative:
- You only have experimental MS/MS spectra with no in silico CFM-ID predictions to
  compare against (the skill's core is predicted-vs-experimental overlap)
- Your data are LC-HRMS survey/precursor masses without tandem MS/MS fragment spectra,
  so no fragment-level matching is possible
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CFM-ID
  role: Generates the in silico MS/MS fragments for all DNA adducts that are matched
    against the experimental spectra
- name: tandem HRMS/MS
  role: Provides the experimental MS/MS spectra of the 15 reference standards used
    as the matching target
- name: MS/MS
  role: Acquisition of experimental fragment spectra of the 15 reference standards
    for the preliminary spectral library
- name: DT (rstudio/DT)
  role: R DataTables interface used to expose both experimental and predicted fragment
    tables as interactive web datatables
  repo: https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:42e690e8c20b1c1585a6fc55e83dacddc0ef25b5630adeb9c48cc427fcb8b48f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - DT (rstudio/DT)
  - MS/MS
  - tandem HRMS/MS
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectral-library-matching

## Summary

Matches CFM-ID in silico MS/MS fragment predictions against experimentally acquired MS/MS spectra (20, 40, 60 eV on a Vion-IMS-qTOF) of 15 DNA adduct reference standards to validate predicted fragments for DNA adduct annotation.

## When to use

Use when you need to validate in silico (CFM-ID) fragment predictions for DNA adducts or other small molecules against experimental tandem HRMS/MS spectra of reference standards before using predicted fragments for annotation.

## When NOT to use

- You only have experimental MS/MS spectra with no in silico CFM-ID predictions to compare against (the skill's core is predicted-vs-experimental overlap)
- Your data are LC-HRMS survey/precursor masses without tandem MS/MS fragment spectra, so no fragment-level matching is possible

## Inputs

- Experimental MS/MS fragment datatable for 15 DNA adduct reference standards acquired at 20, 40, and 60 eV on a Vion-IMS-qTOF
- CFM-ID predicted fragment datatable (in silico MS/MS fragments above 5% relative intensity, three energy levels)
- Adduct structure identifiers (SMILES/InChI/.mol files) from the DNA adduct database

## Outputs

- fragment_overlap_summary.csv with per-adduct matched/total predicted and experimental fragment counts
- 15 overlaid predicted-vs-experimental spectrum panels (one per reference standard)
- Interactive web datatables reporting both experimental and predicted fragments

## How to apply

Load the experimental fragment datatable (15 reference standards, MS/MS at 20, 40, and 60 eV on the Vion-IMS-qTOF) and the CFM-ID predicted fragment datatable (fragments above 5% relative intensity at three energy levels). For each of the 15 DNA adduct standards, match predicted fragment m/z values against experimental fragment m/z values at each collision energy using a high-resolution mass tolerance consistent with qTOF accuracy, and compute per-adduct overlap counts (matched fragments / total predicted and experimental fragments), noting the characteristic [M+H-dR]+ deoxyribose-loss fragment. Generate one overlaid predicted-vs-experimental spectrum panel per standard (15 panels total, mirroring Supplementary Figures S1–S15), write the overlap statistics to fragment_overlap_summary.csv, and confirm that spectra show considerable overlap, which supports using predicted fragments for DNA adduct annotation. Interactive browsing of experimental and predicted fragment tables can be served with the R DT package's datatable() function, as in the paper's web datatables.

## Related tools

- **CFM-ID** (Generates the in silico MS/MS fragments for all DNA adducts that are matched against the experimental spectra)
- **tandem HRMS/MS** (Provides the experimental MS/MS spectra of the 15 reference standards used as the matching target)
- **MS/MS** (Acquisition of experimental fragment spectra of the 15 reference standards for the preliminary spectral library)
- **DT (rstudio/DT)** (R DataTables interface used to expose both experimental and predicted fragment tables as interactive web datatables) — https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database

## Examples

```
Rscript -e "library(DT); datatable(read.csv('fragment_overlap_summary.csv'))"
```

## Evaluation signals

- Per-adduct overlap summary (fragment_overlap_summary.csv) shows considerable overlap between CFM-ID predicted and experimental MS/MS fragments across the 15 standards
- Each of the 15 standards has a corresponding overlaid predicted-vs-experimental spectrum panel (15 panels, mirroring Supplementary Figures S1–S15)
- Matched m/z values fall within a high-resolution qTOF mass tolerance at all three collision energies (20, 40, 60 eV)
- The characteristic [M+H-dR]+ deoxyribose-loss fragment is present/annotated for the deoxyribonucleoside adduct standards
- Predicted fragments used are restricted to those above 5% relative intensity at the three energy levels

## Limitations

- The experimental spectral library is preliminary, covering only 15 reference standards; overlap conclusions rest on this limited set
- Overlap was qualitative ('considerable'); no formal similarity score or numeric threshold is reported in the abstract
- Predicted fragments come from CFM-ID only and may not capture all experimental fragmentation behavior

## Evidence

- [methods] Comparing the CFM-ID predicted spectra with the experimental MS/MS spectra of the 15 standards showed considerable overlap, so the predicted fragments can support DNA adduct annotation.: [evidence span withheld — anchor status: not_found]
- [abstract] a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards: "a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] reporting both experimental and predicted fragments into interactive web datatables: "reporting both experimental and predicted fragments into interactive web datatables"
- [readme] This package provides a function datatable() to display R data via the DataTables library: [evidence span withheld — anchor status: external_doc]
