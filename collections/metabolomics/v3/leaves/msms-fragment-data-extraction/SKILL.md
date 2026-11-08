---
name: msms-fragment-data-extraction
description: Use when UNIFI-exported .xlsx MS/MS spectra of reference standards need
  to be converted into a fragment datatable stratified by standard and collision energy
  (20/40/60 eV).
when_to_use_negative:
- Input is in silico predicted fragment data (CFM-ID predictions) rather than experimentally
  acquired MS/MS spectra — those belong in the predicted-fragments datatable
- Spectra were not acquired at the three specified collision energies (20, 40, 60
  eV), so entries cannot align with the deposited datatable's energy stratification
- The goal is de novo spectrum interpretation rather than reproducing/curating the
  reference-standard fragment library
tools:
- name: MS/MS (tandem HRMS/MS)
  role: Source of the experimental fragment spectra acquired at 20, 40, and 60 eV
    for the 15 reference standards
- name: rstudio/DT
  role: R DataTables package used to render the fragment entries as interactive searchable
    web datatables
  repo: https://github.com/rstudio/DT
- name: liquid chromatography (LC)
  role: Coupled with HRMS to acquire the MS/MS spectra of the reference standards
    that the fragments are extracted from
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:4cc99fb9ca8ef7a631e29d76873cb2f7f011dd32c965beeb63ed9df9f19c5109
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - MS/MS (tandem HRMS/MS)
  - liquid chromatography (LC)
  - rstudio/DT
  - tandem HRMS/MS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# msms-fragment-data-extraction

## Summary

Extracts fragment-ion entries from UNIFI-exported .xlsx MS/MS spectra files of 15 DNA adduct reference standards, acquired at three collision energies (20, 40, 60 eV), and reorganizes them into one searchable datatable row per fragment ion for a DNA adductomics spectral library.

## When to use

Use when UNIFI-exported .xlsx MS/MS spectra of reference standards need to be converted into a fragment datatable stratified by standard and collision energy (20/40/60 eV).

## When NOT to use

- Input is in silico predicted fragment data (CFM-ID predictions) rather than experimentally acquired MS/MS spectra — those belong in the predicted-fragments datatable
- Spectra were not acquired at the three specified collision energies (20, 40, 60 eV), so entries cannot align with the deposited datatable's energy stratification
- The goal is de novo spectrum interpretation rather than reproducing/curating the reference-standard fragment library

## Inputs

- UNIFI-exported .xlsx MS/MS spectra files for 15 DNA adduct reference standards
- Deposited experimental-fragments datatable from https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database

## Outputs

- Searchable datatable with one row per fragment ion, containing fields: standard name, precursor ion, fragment m/z, and collision energy (20, 40, or 60 eV)

## How to apply

Locate the deposited experimental-fragments datatable in the dna_adductomics_database repository, which was built from UNIFI-exported .xlsx spectra files of 15 purchased DNA adduct reference standards (e.g., N6-Me-dA, 8-oxo-dG, 1,N6-ε-dA, M1-dG, Acr-1I-dG, 8-ABP-dG, 8-MeIQx-dG). Extract the fragment-ion entries for each standard from the LC-HRMS/MS acquisitions performed at three collision energies (20, 40, and 60 eV), retaining the most significant fragments only. Organize one row per fragment ion, mirroring the structure of the original interactive web datatable (rendered with the R DataTables/DT package). Populate fields for standard name, precursor ion, fragment m/z, and collision energy. Validate that all 15 standards are represented across all three collision energies before treating the table as a faithful reproduction of the deposited datatable.

## Related tools

- **MS/MS (tandem HRMS/MS)** (Source of the experimental fragment spectra acquired at 20, 40, and 60 eV for the 15 reference standards)
- **rstudio/DT** (R DataTables package used to render the fragment entries as interactive searchable web datatables) — https://github.com/rstudio/DT
- **liquid chromatography (LC)** (Coupled with HRMS to acquire the MS/MS spectra of the reference standards that the fragments are extracted from)

## Examples

```
library(DT); DT::datatable(fragments_df)
```

## Evaluation signals

- Reproduced table contains fragment entries for all 15 reference standards (N6-Me-dA, 5-Me-dC, O6-Me-dG, 3-Me-dT, N4,5-DiMe-dC, N2-ethyl-dG, N6-(2-OH-ethyl)-dA, 8-oxo-dG, 1,N6-ε-dA, 3,N4-ε-dC, M1-dG, 6-Oxo-M1-dG, Acr-1I-dG, 8-ABP-dG, 8-MeIQx-dG)
- Every standard's fragments are stratified across all three collision energies (20, 40, and 60 eV)
- Table matches the deposited experimental-fragments datatable in the gitlab repository
- One row per fragment ion, with standard name, precursor ion, fragment m/z, and collision energy fields populated as in the original searchable datatable

## Limitations

- This is a preliminary spectral library built from only 15 reference standards, so it covers a limited subset of the 582 database entries
- The rstudio/DT package is in maintenance-only mode; pull requests for new features may not be reviewed or merged
- Only the most significant fragments were retained — low-intensity fragment ions from the UNIFI exports are not in the datatable

## Evidence

- [abstract] preliminary spectral library built by acquiring experimental MS/MS spectra of 15 reference standards: "In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] experimental and predicted fragments in interactive web datatables: "reporting both experimental and predicted fragments into interactive web datatables"
- [abstract] LC-HRMS as the analytical basis: "Liquid chromatography (LC) coupled with high-resolution mass spectrometry (HRMS) is a powerful tool for DNA adductomics"
- [readme] DT package renders searchable datatables: [evidence span withheld — anchor status: external_doc]
