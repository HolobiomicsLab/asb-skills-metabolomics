---
name: spectrum-metadata-validation
description: Use when a public or private MS/MS spectral library (e.g. GNPS .msp/MGF
  exports) contains unstandardized metadata and annotated structures, and spectra
  with inconsistent annotations must be removed before library searching or machine-learning
  training.
when_to_use_negative:
- 'You need plausibility checks that consider measured fragments against the annotation:
  the article notes such checks are not implemented — wrong chemical annotations consistent
  with the measured mass go unnoticed, and fragment-matching filters are only a possible
  future expansion.'
- 'Your goal is to standardize instrument type or collision-energy metadata: the article
  states these ''could be important fields that might require additional standardization''
  and are not yet covered by current matchms filters.'
- Input is already a cleaned library with complete, harmonized annotations and no
  derived metadata is missing — running the pipeline adds overhead without benefit.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: matchms
  role: Python package that imports, processes, cleans and compares MS/MS data; the
    library-cleaning pipeline and the require_* validation filters run within the
    matchms framework (version 0.26.4 was used for the published pipelines)
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Used inside require_valid_annotation to load SMILES, InChI and InChIKey and
    cross-compare them, removing spectra where one of these is incomplete or does
    not match
- name: PubChem
  role: External lookup used by the Derive Annotation From Compound Name filter to
    obtain canonical SMILES, InChI and InChIKey when only a compound name is annotated
- name: bioconda
  role: Conda channel that distributes the matchms package for installation in a virtual
    environment
  repo: https://github.com/bioconda/bioconda-recipes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:2db5e242d11dd29f2e60eefe6bec238e1585d09dc6843f9e85715743a703dc27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - PubChem
  - Python
  - RDKit
  - bioconda
  - matchms
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9259259259259259
n_claims: 27
---

# spectrum-metadata-validation

## Summary

A matchms-based workflow that validates and harmonizes spectral metadata (adduct, ionmode, precursor m/z, parent mass, SMILES/InChI/InChIKey) before keeping a spectrum in a cleaned MS/MS library, discarding entries with incorrect or incomplete annotations.

## When to use

Use when a public or private MS/MS spectral library (e.g. GNPS .msp/MGF exports) contains unstandardized metadata and annotated structures, and spectra with inconsistent annotations must be removed before library searching or machine-learning training.

## When NOT to use

- You need plausibility checks that consider measured fragments against the annotation: the article notes such checks are not implemented — wrong chemical annotations consistent with the measured mass go unnoticed, and fragment-matching filters are only a possible future expansion.
- Your goal is to standardize instrument type or collision-energy metadata: the article states these 'could be important fields that might require additional standardization' and are not yet covered by current matchms filters.
- Input is already a cleaned library with complete, harmonized annotations and no derived metadata is missing — running the pipeline adds overhead without benefit.

## Inputs

- MS/MS spectral library in a matchms-supported format (mzML, mzXML, msp, MGF, JSON, or metabolomics-USI), e.g. the public GNPS library .msp/MGF export
- Per-spectrum metadata fields: adduct, ionmode, precursor m/z, parent mass, compound name, SMILES, InChI, InChIKey

## Outputs

- Cleaned spectral library containing only spectra passing all validation filters (448,485 spectra from 500,569 for GNPS, published on Zenodo)
- Processing report (YAML settings file plus per-filter change counts) summarizing how many spectra each filter removed or modified

## How to apply

Load spectra as matchms Spectrum objects from the target library (e.g. the GNPS public mass spectral library downloaded 21-08-2023, 500,569 spectra) so metadata fields such as adduct, ionmode, precursor m/z, parent mass, SMILES, InChI and InChIKey are accessible. Run the matchms library-cleaning pipeline (matchms 0.26.4), whose key validation filters are require_minimum_number_of_high_peaks, require_matching_adduct_and_ionmode, require_matching_adduct_precursor_mz_and_parent_mass, require_parent_mass_match_smiles and require_valid_annotation; the latter uses RDKit to load and cross-compare SMILES, InChI and InChIKey and removes a spectrum if one of these is incomplete or does not match. Before the requirement filters, metadata is harmonized and repaired: canonical SMILES/InChI/InChIKey are derived from compound names via PubChem (Derive Annotation From Compound Name), salt SMILES are repaired by checking the parent mass against separate or combined salt ions, parent masses calculated from molar mass are corrected to monoisotopic mass, adducts are corrected based on SMILES and precursor m/z, and non-matching annotation fields are reconciled. A spectrum is kept only when every required filter passes; each filter's keep/discard decision is recorded so per-filter counts can be reported. Verify by running the matchms processing report — the pipeline took 6 h 45 min on the full GNPS library — and confirm that reported removal counts are consistent with the 500,569 → 448,485 spectra scale of the cleaned library.

## Related tools

- **matchms** (Python package that imports, processes, cleans and compares MS/MS data; the library-cleaning pipeline and the require_* validation filters run within the matchms framework (version 0.26.4 was used for the published pipelines)) — https://github.com/matchms/matchms
- **RDKit** (Used inside require_valid_annotation to load SMILES, InChI and InChIKey and cross-compare them, removing spectra where one of these is incomplete or does not match)
- **PubChem** (External lookup used by the Derive Annotation From Compound Name filter to obtain canonical SMILES, InChI and InChIKey when only a compound name is annotated)
- **bioconda** (Conda channel that distributes the matchms package for installation in a virtual environment) — https://github.com/bioconda/bioconda-recipes

## Examples

```
from matchms.importing import load_from_msp; from matchms.filtering import require_valid_annotation, require_matching_adduct_and_ionmode, require_parent_mass_match_smiles; spectra = [s for s in load_from_msp('gnps_library.msp')]; kept = [s for s in spectra if all([require_valid_annotation(s), require_matching_adduct_and_ionmode(s), require_parent_mass_match_smiles(s)])]
```

## Evaluation signals

- The pipeline must run to completion without error on the loaded library (matchms 0.26.4; GNPS library of 500,569 spectra took 6 h 45 min).
- The processing report lists a change count for every filter applied; each row of the report should correspond to a matchms filter actually run.
- Per-filter keep/discard decisions can be tabulated per spectrum (filter_minimum_high_peaks, filter_matching_adduct_ionmode, filter_matching_adduct_precursor_parent, filter_parent_mass_match_smiles, filter_valid_annotation, overall_keep); overall_keep is true only if all five pass.
- The output library size should match the scale reported for the cleaned GNPS library: 448,485 spectra from an input of 500,569.
- Every kept spectrum must have a complete, mutually consistent SMILES/InChI/InChIKey set as validated by RDKit inside require_valid_annotation.

## Limitations

- Filters 'currently still lack plausibility checks that consider both metadata and measured fragments. Wrong chemical annotations that are consistent with the measured mass, for instance, will go unnoticed'.
- Instrument type and collision energies are not yet standardized by current matchms filters and 'could be important fields that might require additional standardization'.
- The SMILES derived from compound names via PubChem is only accepted 'if it matches the given parent mass', so names that resolve to a wrong-mass structure are not recovered.
- No automatic curation/validation tool for uploads to public libraries is implemented; such a validator is proposed as future work.
- Processing-report examples are only available as supplementary tables (Table S1, Table S2), which limits direct inspection.

## Evidence

- [intro] structure annotation validation through adduct, precursor m/z, and annotation comparison and harmonization: "structure annotation validation through adduct, precursor m/z, and annotation comparison and harmonization"
- [methods] SMILES, InChI and InChIKey are loaded by RDKit and compared to each other. If one of these is not complete or does not match the spectrum will be removed.: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other. If one of these is not complete or does not match the spectrum will be removed."
- [discussion] The pipeline combines various filter functions to harmonize, derive and repair metadata or discard spectra with incorrect annotations.: "The pipeline combines various filter functions to harmonize, derive and repair metadataor discard spectra with incorrect annotations."
- [results] Before cleaning, the GNPS library contained 500,569 spectra.: "Before cleaning, the GNPS library contained 500,569 spectra."
- [readme] matchms now offers a `Pipeline` class to handle complex workflows. This also allows to create, import, export, or modify workflows using yaml files.: [evidence span withheld — anchor status: external_doc]
