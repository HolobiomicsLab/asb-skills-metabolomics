---
name: spectral-library-compilation
description: Use when building or extending a curated compound database for LC-HRMS-based
  DNA adductomics where tentative annotation requires a fragment-level spectral library
  built from experimental MS/MS of reference standards.
when_to_use_negative:
- Input is a complete nucleotide digestion / LC-HRMS run of a biological sample meant
  for adduct annotation rather than reference-standard fragmentation for library building
- The target compounds have no available reference standard and no in silico fragmentation
  model has been defined for their chemical class — the library step cannot supply
  spectra for them
- The goal is annotation by NMR or other full-structure confirmation rather than fragment-based
  MS/MS matching
edam_operation: http://edamontology.org/operation_3795
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: tandem HRMS/MS
  role: Instrument used to acquire experimental MS/MS spectra of the 15 reference
    standards at three collision energies (20, 40, 60 eV) for the preliminary spectral
    library
- name: CFM-ID
  role: Generates in silico MS/MS fragments for all the adducts in the database, complementing
    the experimentally acquired spectra
- name: rstudio/DT
  role: R package providing the datatable() function used to render the experimental
    and predicted fragments as searchable, interactive web datatables
  repo: https://github.com/rstudio/DT
- name: ChemDraw
  role: Source of the structural identifiers (.cdxml files, .mol files, SMILES, InChI,
    InChIKey, IUPAC nomenclature) that accompany each adduct entry and feed in silico
    fragmentation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fchem.2022.908572
    title: A Comprehensive Database for DNA Adductomics
  source_content_hash: sha256:0d7bd91da50575e850182c35c74829c9b42596623386e998f17a6e29874bea7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rstudio/DT
  tools:
  - CFM-ID
  - ChemDraw
  - rstudio/DT
  - tandem HRMS/MS
  edam_operation: http://edamontology.org/operation_3795
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-library-compilation

## Summary

Compiles a preliminary DNA-adduct MS/MS spectral library by acquiring experimental tandem HRMS/MS spectra of reference standards at multiple collision energies, extracting significant fragment ions, and uploading them into a searchable web datatable alongside in silico predicted fragments.

## When to use

Use when building or extending a curated compound database for LC-HRMS-based DNA adductomics where tentative annotation requires a fragment-level spectral library built from experimental MS/MS of reference standards.

## When NOT to use

- Input is a complete nucleotide digestion / LC-HRMS run of a biological sample meant for adduct annotation rather than reference-standard fragmentation for library building
- The target compounds have no available reference standard and no in silico fragmentation model has been defined for their chemical class — the library step cannot supply spectra for them
- The goal is annotation by NMR or other full-structure confirmation rather than fragment-based MS/MS matching

## Inputs

- Purchased DNA adduct reference standards (15 compounds, e.g. N6-Me-dA, 5-Me-dC, O6-Me-dG, 3-Me-dT, N4,5-DiMe-dC, N2-ethyl-dG, N6-(2-OH-ethyl)-dA, 8-oxo-dG, 1,N6-ε-dA, 3,N4-ε-dC, M1-dG, 6-Oxo-M1-dG, Acr-1I-dG, 8-ABP-dG, 8-MeIQx-dG)
- UNIFI-exported .xlsx MS/MS spectra files
- In silico fragmentation input: .cdxml / .mol files, SMILES, InChI, InChIKey, IUPAC names, monoisotopic masses and chemical formulas for the 582 database entries

## Outputs

- Searchable web datatable of experimental fragment ions, one row per fragment per standard per collision energy (20, 40, 60 eV), with fields for standard name, precursor ion, fragment m/z, and collision energy
- Preliminary spectral library deposited at https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database
- Interactive web datatables of in silico predicted fragments (CFM-ID) complementing the experimental ones

## How to apply

Acquire experimental MS/MS spectra of the purchased reference standards (15 DNA adducts such as N6-Me-dA, 5-Me-dC, O6-Me-dG, 8-oxo-dG, 1,N6-ε-dA, M1-dG, 8-ABP-dG, 8-MeIQx-dG) with tandem HRMS/MS at three different collision energies (20, 40, and 60 eV). Retrieve the most significant fragment ions from each spectrum and organize them into a searchable web datatable (built with the R rstudio/DT package) keyed by standard name, precursor ion, fragment m/z, and collision energy. Deposit the resulting experimental-fragments datatable in the public repository alongside the UNIFI-exported .xlsx spectra files. Complement the experimental entries with in silico predicted fragments generated with CFM-ID for all 582 adduct entries, reporting both experimental and predicted fragments into the same interactive web datatables so that unknowns in biological samples can be tentatively identified by comparison with reference standards or with the predicted fragment sets.

## Related tools

- **tandem HRMS/MS** (Instrument used to acquire experimental MS/MS spectra of the 15 reference standards at three collision energies (20, 40, 60 eV) for the preliminary spectral library)
- **CFM-ID** (Generates in silico MS/MS fragments for all the adducts in the database, complementing the experimentally acquired spectra)
- **rstudio/DT** (R package providing the datatable() function used to render the experimental and predicted fragments as searchable, interactive web datatables) — https://github.com/rstudio/DT
- **ChemDraw** (Source of the structural identifiers (.cdxml files, .mol files, SMILES, InChI, InChIKey, IUPAC nomenclature) that accompany each adduct entry and feed in silico fragmentation)

## Examples

```
Rscript -e 'DT::datatable(read.csv("experimental_fragments.csv"))'
```

## Evaluation signals

- The reproduced experimental-fragments datatable contains fragment entries for all 15 reference standards across all three collision energies (20, 40, 60 eV) and matches the deposited datatable in the gitlab repository
- Each fragment row carries a standard name, a precursor ion, a fragment m/z, and a collision-energy label consistent with the acquisition parameters
- The published library is publicly retrievable at https://gitlab.com/nexs-metabolomics/projects/dna_adductomics_database and is searchable in a web browser (rstudio/DT interactive datatable)
- In silico predicted fragments from CFM-ID exist for every one of the 582 database entries, not only for the 15 standards with experimental spectra
- Fragment ions retained in the datatable are those judged 'most significant' from the spectra, not the full raw peak list

## Limitations

- The spectral library is preliminary: it covers only 15 reference standards experimentally, and all other adducts rely solely on CFM-ID in silico predictions
- Fragment retrieval depends on which peaks were judged most significant at each collision energy; the deposited datatable may not capture every observed fragment
- Identification is tentative by design — confirmatory annotation still requires reference-standard comparison or NMR for full structural proof
- The rstudio/DT package is in maintenance-only mode; bug fixes are prioritized and new pull requests may not be reviewed or merged

## Evidence

- [abstract] acquiring experimental MS/MS spectra of 15 reference standards, generating in silico MS/MS fragments: "In the fourth step, a preliminary spectral library was built by acquiring experimental MS/MS spectra of 15 reference standards"
- [abstract] reporting both experimental and predicted fragments into interactive web datatables: "reporting both experimental and predicted fragments into interactive web datatables"
- [results] The spectra of 15 DNA adduct reference standards were acquired at three different collision energies: [evidence span withheld — anchor status: not_found]
- [readme] This package provides a function `datatable()` to display R data via the DataTables library: [evidence span withheld — anchor status: external_doc]
- [abstract] generating in silico MS/MS fragments for all the adducts: "generating in silico MS/MS fragments for all the adducts"
