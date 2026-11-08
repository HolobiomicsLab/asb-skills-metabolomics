---
name: msms-fragment-overlap-analysis
description: Decision support for whether CFM-ID predicted fragments can back annotation
  of DNA adducts in HRMS adductomics when experimental MS/MS spectra exist only for
  a subset (e.g. 15 of 582) of database entries.
when_to_use_negative:
- No experimental MS/MS spectra exist for the adducts of interest — the comparison
  degenerates and overlap cannot be computed.
- The spectra were acquired on a low-resolution instrument whose mass accuracy cannot
  support the high-resolution tolerance assumed for qTOF fragment matching.
- A preliminary spectral library already exists for the adducts; annotation should
  then rely on measured spectra rather than predicted-fragment overlap.
tools:
- name: CFM-ID
  role: generates in-silico MS/MS fragments for all database adducts that are matched
    against experimental spectra
- name: MS/MS
  role: experimental tandem spectra of the 15 reference standards to be compared with
    predictions
- name: tandem HRMS/MS
  role: high-resolution MS/MS acquisition on the Vion-IMS-qTOF providing fragment
    m/z values for overlap matching
- name: DT
  role: R DataTables package used to render the experimental and predicted fragment
    datatables that feed the comparison
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:721994247ce40038377edebd37f05417372eb15b6e71b214a227a63982e145c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - DT
  - MS/MS
  - tandem HRMS/MS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# msms-fragment-overlap-analysis

## Summary

Assess agreement between CFM-ID in-silico MS/MS fragment predictions and experimental tandem HRMS/MS spectra of DNA adduct reference standards, quantifying fragment overlap to validate using predicted fragments for DNA adduct annotation.

## When to use

Decision support for whether CFM-ID predicted fragments can back annotation of DNA adducts in HRMS adductomics when experimental MS/MS spectra exist only for a subset (e.g. 15 of 582) of database entries.

## When NOT to use

- No experimental MS/MS spectra exist for the adducts of interest — the comparison degenerates and overlap cannot be computed.
- The spectra were acquired on a low-resolution instrument whose mass accuracy cannot support the high-resolution tolerance assumed for qTOF fragment matching.
- A preliminary spectral library already exists for the adducts; annotation should then rely on measured spectra rather than predicted-fragment overlap.

## Inputs

- experimental MS/MS fragment datatable for 15 DNA adduct reference standards (three collision energies: 20, 40, 60 eV)
- CFM-ID predicted fragment datatable for the same adducts (fragments above 5% relative intensity, three energy levels)
- deoxyribose-loss [M+H-dR]+ target fragment identities for each adduct

## Outputs

- fragment_overlap_summary.csv with per-adduct matched-fragment counts and overlap ratios
- 15 overlaid predicted-vs-experimental MS/MS spectrum plots (one per standard, analogous to Supplementary Figures S1–S15)

## How to apply

Load the experimental fragment datatable for the reference standards (MS/MS acquired on the Vion-IMS-qTOF at 20, 40, and 60 eV) and the CFM-ID predicted fragment datatable (predicted fragments above 5% relative intensity generated at three energy levels). For each of the 15 DNA adduct standards, match predicted fragment m/z values against experimental fragment m/z values per collision energy using a high-resolution qTOF-compatible mass tolerance, tracking the characteristic [M+H-dR]+ deoxyribose-loss fragment. Compute per-adduct overlap statistics (matched fragments relative to total predicted and total experimental fragments) and write them to fragment_overlap_summary.csv. Generate overlaid predicted-vs-experimental spectrum plots, one panel per standard (15 panels total), mirroring Supplementary Figures S1–S15, and save them. The considerable qualitative overlap between predicted and experimental spectra is the rationale for using in-silico fragments to support annotation of adducts lacking reference standards.

## Related tools

- **CFM-ID** (generates in-silico MS/MS fragments for all database adducts that are matched against experimental spectra)
- **MS/MS** (experimental tandem spectra of the 15 reference standards to be compared with predictions)
- **tandem HRMS/MS** (high-resolution MS/MS acquisition on the Vion-IMS-qTOF providing fragment m/z values for overlap matching)
- **DT** (R DataTables package used to render the experimental and predicted fragment datatables that feed the comparison) — https://github.com/rstudio/DT

## Examples

```
Rscript plot_fragment_overlap.R --predicted cfmid_fragments.csv --experimental msms_15standards.csv --energies 20,40,60 --out fragment_overlap_summary.csv
```

## Evaluation signals

- fragment_overlap_summary.csv is produced and contains one row per adduct with matched-fragment and total-fragment counts
- 15 overlaid spectrum panels are generated, one per reference standard, each showing predicted and experimental fragment m/z values at the three collision energies
- Per-adduct overlap counts qualitatively show considerable agreement between CFM-ID predictions and experimental MS/MS spectra
- The characteristic [M+H-dR]+ deoxyribose-loss fragment appears as a matched fragment for the corresponding deoxynucleoside adducts
- Fragment matches are computed with a high-resolution tolerance consistent with qTOF mass accuracy, not with unit-mass resolution

## Limitations

- Only 15 of the 582 database entries have experimental MS/MS spectra; overlap analysis for the remaining adducts rests entirely on predicted fragments
- The spectral library is described as preliminary — experimental coverage and annotation confidence are limited accordingly
- Overlap is reported qualitatively ('considerable overlap') rather than as a validated numeric annotation-confidence threshold

## Evidence

- [other] Comparing the CFM-ID predicted spectra with the experimental MS/MS spectra of the 15 standards showed considerable overlap: [evidence span withheld — anchor status: not_found]
- [abstract] In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards: "In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] generating in silico MS/MS fragments for all the adducts: "generating in silico MS/MS fragments for all the adducts"
- [abstract] The database, including 582 entries, is publicly available: "The database, including 582 entries, is publicly available"
- [readme] This package provides a function datatable() to display R data via the DataTables library: [evidence span withheld — anchor status: external_doc]
