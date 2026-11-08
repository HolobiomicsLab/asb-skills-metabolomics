---
name: collision-energy-spectra-organization
description: Use when experimental MS/MS spectra of reference standards (e.g., the
  15 DNA adduct standards) have been acquired at multiple collision energies and fragments
  must be organized per standard and energy into a searchable fragment datatable.
when_to_use_negative:
- Only in silico predicted fragments are needed (use the CFM-ID-based generation for
  the 303 potential DNA adducts instead of reference-standard spectra)
- The adducts of interest are tentatively identified without available reference standards,
  so no experimental spectra can be acquired
- The goal is querying existing adduct metadata (monoisotopic masses, SMILES, InChIKey)
  rather than building or extending the experimental fragment datatable
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: tandem HRMS/MS
  role: Acquires experimental MS/MS spectra of the 15 reference standards at three
    collision energies for fragment extraction
- name: MS/MS
  role: 'Fourth database step: building the preliminary spectral library from reference-standard
    spectra'
- name: rstudio/DT
  role: R interface used to render the interactive (searchable) web datatables of
    experimental fragments
  repo: https://github.com/rstudio/DT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:8b07a03c7b0737a5358178e3ee67aa145f9ff068e0b1d577be2ac09fcc544985
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - MS/MS
  - rstudio/DT
  - tandem HRMS/MS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# collision-energy-spectra-organization

## Summary

Organizes experimental MS/MS fragments of DNA adduct reference standards acquired at three collision energies (20, 40, 60 eV) into a searchable web datatable, forming the experimental half of the preliminary spectral library in the DNA adductomics database.

## When to use

Use when experimental MS/MS spectra of reference standards (e.g., the 15 DNA adduct standards) have been acquired at multiple collision energies and fragments must be organized per standard and energy into a searchable fragment datatable.

## When NOT to use

- Only in silico predicted fragments are needed (use the CFM-ID-based generation for the 303 potential DNA adducts instead of reference-standard spectra)
- The adducts of interest are tentatively identified without available reference standards, so no experimental spectra can be acquired
- The goal is querying existing adduct metadata (monoisotopic masses, SMILES, InChIKey) rather than building or extending the experimental fragment datatable

## Inputs

- UNIFI-exported .xlsx MS/MS spectra files of the 15 DNA adduct reference standards
- Fragment-ion lists acquired at 20, 40, and 60 eV collision energies

## Outputs

- Searchable web datatable of experimental fragment ions (standard name, precursor ion, fragment m/z, collision energy)
- Experimental portion of the preliminary spectral library deposited at https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database

## How to apply

Acquire tandem HRMS/MS spectra of each reference standard (e.g., N6-Me-dA, 5-Me-dC, O6-Me-dG, 8-oxo-dG, M1-dG, 8-ABP-dG) at three collision energies: 20, 40, and 60 eV, exporting spectra from UNIFI as .xlsx files. Extract the most significant fragment ions per standard and per collision energy, keeping one row per fragment ion with fields for standard name, precursor ion, fragment m/z, and collision energy. Assemble these entries into a searchable web datatable (the experimental-fragments datatable), which complements the in silico fragments generated with CFM-ID for the 582-entry database. Deposit the datatable in the public GitLab repository so it can be queried for annotation of DNA adducts in untargeted LC-HRMS adductomics.

## Related tools

- **tandem HRMS/MS** (Acquires experimental MS/MS spectra of the 15 reference standards at three collision energies for fragment extraction)
- **MS/MS** (Fourth database step: building the preliminary spectral library from reference-standard spectra)
- **rstudio/DT** (R interface used to render the interactive (searchable) web datatables of experimental fragments) — https://github.com/rstudio/DT

## Examples

```
install.packages('DT'); DT::datatable(iris)
```

## Evaluation signals

- Fragment entries are present for all 15 reference standards (N6-Me-dA through 8-MeIQx-dG) in the output table
- Each standard has fragment entries at all three collision energies: 20, 40, and 60 eV
- The table schema retains one row per fragment ion with standard name, precursor ion, fragment m/z, and collision energy fields
- The reproduced datatable matches the deposited experimental-fragments datatable in the public GitLab repository

## Limitations

- The experimental library is preliminary: it covers only 15 reference standards, whereas the database contains 582 entries; most adducts rely on in silico CFM-ID fragments
- Spectra were acquired at only three fixed collision energies (20, 40, 60 eV), so fragments outside these regimes are not represented
- Fragment entries were curated from UNIFI .xlsx exports; no retention-time matching details are provided in the abstract-level description

## Evidence

- [abstract] In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards: "In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] interactive web datatables: "reporting both experimental and predicted fragments into interactive web datatables"
- [other] three different collision energies: [evidence span withheld — anchor status: not_found]
- [readme] DT datatable function: [evidence span withheld — anchor status: external_doc]
