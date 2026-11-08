---
name: dna-adduct-reference-standard-annotation
description: Use when untargeted HRMS DNA adductomics data must be annotated against
  experimental MS/MS fragments of purchased reference standards (e.g., N6-Me-dA, 8-oxo-dG,
  M1-dG), or when reproducing/verifying an experimental-fragments datatable.
when_to_use_negative:
- The adduct is not covered by any of the 15 reference standards — in that case only
  in silico CFM-ID fragments are available and tentative identification must rely
  on prediction rather than measured spectra
- The task is constructing the main 582-entry database of DNA adducts (literature
  curation, harmonization, implementation) rather than building or using the experimental
  spectral library
- Only a single collision energy was acquired — the method requires spectra at 20,
  40, and 60 eV for the fragment table
tools:
- name: liquid chromatography (LC)
  role: Chromatographic separation of reference standards prior to HRMS/MS fragmentation
- name: tandem HRMS/MS
  role: Acquires experimental MS/MS spectra of the 15 reference standards at 20, 40,
    and 60 eV collision energies
- name: CFM-ID
  role: Generates complementary in silico MS/MS fragments for adducts without reference
    standards
- name: rstudio/DT
  role: R DataTables interface used to render the searchable web datatables of experimental
    and predicted fragments
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:8689f4189c688122d4bf9ed50e6fe7ac905f020b78a081a1113a499fda67cf4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - liquid chromatography (LC)
  - rstudio/DT
  - tandem HRMS/MS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# dna-adduct-reference-standard-annotation

## Summary

Builds a preliminary DNA adduct spectral library by acquiring experimental LC-HRMS/MS spectra of 15 DNA adduct reference standards at three collision energies (20, 40, 60 eV), extracting significant fragment ions, and publishing them in a searchable web datatable alongside in silico predicted fragments (CFM-ID).

## When to use

Use when untargeted HRMS DNA adductomics data must be annotated against experimental MS/MS fragments of purchased reference standards (e.g., N6-Me-dA, 8-oxo-dG, M1-dG), or when reproducing/verifying an experimental-fragments datatable.

## When NOT to use

- The adduct is not covered by any of the 15 reference standards — in that case only in silico CFM-ID fragments are available and tentative identification must rely on prediction rather than measured spectra
- The task is constructing the main 582-entry database of DNA adducts (literature curation, harmonization, implementation) rather than building or using the experimental spectral library
- Only a single collision energy was acquired — the method requires spectra at 20, 40, and 60 eV for the fragment table

## Inputs

- UNIFI-exported .xlsx MS/MS spectra files for the 15 purchased DNA adduct reference standards
- Collision-energy settings: 20, 40, and 60 eV per standard
- Purchased reference standards: N6-Me-dA, 5-Me-dC, O6-Me-dG, 3-Me-dT, N4,5-DiMe-dC, N2-ethyl-dG, N6-(2-OH-ethyl)-dA, 8-oxo-dG, 1,N6-ε-dA, 3,N4-ε-dC, M1-dG, 6-Oxo-M1-dG, Acr-I-dG, 8-ABP-dG, 8-MeIQx-dG

## Outputs

- Searchable web datatable (database of experimental fragments) with one row per fragment ion, containing standard name, precursor ion, fragment m/z, and collision energy
- Preliminary spectral library covering all 15 standards × 3 collision energies
- Companion datatable of CFM-ID in silico predicted MS/MS fragments for all 582 database entries

## How to apply

Acquire experimental MS/MS spectra of the 15 DNA adduct reference standards (e.g., N6-Me-dA, 5-Me-dC, O6-Me-dG, 3-Me-dT, N4,5-DiMe-dC, N2-ethyl-dG, N6-(2-OH-ethyl)-dA, 8-oxo-dG, 1,N6-ε-dA, 3,N4-ε-dC, M1-dG, 6-Oxo-M1-dG, Acr-I-dG, 8-ABP-dG, 8-MeIQx-dG) by LC coupled to tandem high-resolution MS at three collision energies — 20, 40, and 60 eV — so that fragments diagnostic at different energies are captured. Export the spectra files (e.g., UNIFI-exported .xlsx), retrieve the most significant fragment ions per standard per collision energy, and keep one row per fragment ion with standard name, precursor ion, fragment m/z, and collision energy. Deposit these entries as the searchable 'database of experimental fragments' web datatable and supplement them with in silico MS/MS fragments generated with CFM-ID, reporting both experimental and predicted fragments together so tentative identifications lacking reference standards can still be proposed. The decision point is matching unknown adduct annotations: use reference-standard spectra when available, otherwise fall back to NMR evidence or predicted fragments.

## Related tools

- **liquid chromatography (LC)** (Chromatographic separation of reference standards prior to HRMS/MS fragmentation)
- **tandem HRMS/MS** (Acquires experimental MS/MS spectra of the 15 reference standards at 20, 40, and 60 eV collision energies)
- **CFM-ID** (Generates complementary in silico MS/MS fragments for adducts without reference standards)
- **rstudio/DT** (R DataTables interface used to render the searchable web datatables of experimental and predicted fragments) — https://github.com/rstudio/DT

## Examples

```
DT::datatable(exp_fragments)
```

## Evaluation signals

- The reproduced table contains fragment entries for all 15 reference standards across all three collision energies (20, 40, 60 eV)
- Every fragment row carries standard name, precursor ion, fragment m/z, and collision energy fields
- The output matches the deposited experimental-fragments datatable at https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database row-for-row
- The experimental datatable is published alongside (not replacing) the CFM-ID in silico fragment datatable, mirroring the paper's dual experimental/predicted reporting

## Limitations

- Only 15 reference standards were measured, so most of the 582 database entries lack experimental fragments and rely solely on in silico prediction or NMR comparison
- The spectral library is explicitly a 'preliminary' spectral library, not an exhaustive validated library
- The rstudio/DT package is in maintenance-only mode; pull requests for new features cannot be guaranteed to be reviewed or merged

## Evidence

- [abstract] Preliminary spectral library from 15 standards: "In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] Dual experimental and predicted fragment datatables: "reporting both experimental and predicted fragments into interactive web datatables"
- [abstract] Public repository of the database: "The database, including 582 entries, is publicly available ( https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database )."
- [readme] DT datatable function for web tables: [evidence span withheld — anchor status: external_doc]
- [readme] DT maintenance-mode caveat: [evidence span withheld — anchor status: external_doc]
