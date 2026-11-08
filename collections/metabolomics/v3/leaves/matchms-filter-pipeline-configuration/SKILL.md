---
name: matchms-filter-pipeline-configuration
description: Use when building or reproducing an MS/MS library-cleaning stage that
  must harmonize metadata fields and drop spectra whose annotations (SMILES, adduct,
  precursor m/z, ionization mode) are missing or mutually inconsistent.
when_to_use_negative:
- Input is a single unannotated query spectrum or an mzML/mzXML run file rather than
  a curated reference MGF library with SMILES annotations
- The stock unpatched matchms installation is the only option — the [M]+/[M]− charge-zero
  handling requires the modified _get_neutral_mass from the matchms.zip replacement
- The goal is spectral similarity scoring (cosine/entropy) rather than library cleaning;
  that stage comes after preprocessing
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Executes the declarative YAML-configured filter stacks (library_cleaning_1.yaml,
    library_cleaning_2.yaml) against the MGF library; a patched version fixes _get_neutral_mass
    for [M]+/[M]− spectra with zero reported charge
  repo: https://github.com/matchms/matchms
- name: Python
  role: Runtime environment (Python 3.10 or higher) in which the two-round matchms
    preprocessing pipeline is executed
- name: spectraverse-analysis
  role: Provides the preprocessing code and the library_cleaning_1.yaml / library_cleaning_2.yaml
    filter configurations used to compile Spectraverse
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:6fb6994ac90cf3387e15a317fc74b788d7144238f7aa99a0d9c63ec7bd4c3887
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# matchms-filter-pipeline-configuration

## Summary

Configure and execute the Spectraverse two-round matchms library-cleaning pipeline, in which declarative YAML filter stacks (library_cleaning_1.yaml and library_cleaning_2.yaml) harmonize MS/MS metadata and remove spectra with incomplete or chemically incoherent annotations.

## When to use

Use when building or reproducing an MS/MS library-cleaning stage that must harmonize metadata fields and drop spectra whose annotations (SMILES, adduct, precursor m/z, ionization mode) are missing or mutually inconsistent.

## When NOT to use

- Input is a single unannotated query spectrum or an mzML/mzXML run file rather than a curated reference MGF library with SMILES annotations
- The stock unpatched matchms installation is the only option — the [M]+/[M]− charge-zero handling requires the modified _get_neutral_mass from the matchms.zip replacement
- The goal is spectral similarity scoring (cosine/entropy) rather than library cleaning; that stage comes after preprocessing

## Inputs

- Merged MS/MS spectral library in MGF format (raw data files placed in the data/ directory)
- matchms/library_cleaning_1.yaml declarative round-1 filter configuration
- matchms/library_cleaning_2.yaml declarative round-2 filter configuration
- Patched matchms library (matchms.zip replacing the original matchms libraries)

## Outputs

- Filtered, metadata-repaired MS/MS library in MGF format written to the data/ directory, feeding config_step2/config_step3 preprocessing
- Per-filter removal-count report tallying spectra removed by each filter during both rounds

## How to apply

Place the raw merged MS/MS library in MGF format in the data/ directory, then run matchms round 1 with the filter stack in matchms/library_cleaning_1.yaml (HARMONIZE_METADATA_FIELD_NAMES, DERIVE_METADATA_IN_WRONG_FIELD, HARMONIZE_METADATA_ENTRIES, DERIVE_MISSING_METADATA, REPAIR_ANNOTATION, NORMALIZE_INTENSITY). Round 2 reruns the round-1 filters and adds REQUIRE_COMPLETE_METADATA (valid precursor m/z and ionization mode) and REQUIRE_COMPLETE_ANNOTATION, which verifies the parent mass against the SMILES within 0.1 Da, the adduct against the precursor m/z within 0.1 Da, and the adduct against the ionization mode. Critically, this must run against a patched matchms in which the function _get_neutral_mass ignores spectra annotated with [M]+ or [M]− adducts reporting a charge of zero, since stock matchms mishandles these net-charge-zero spectra. Tally spectra removed per filter during both rounds into a removal-count report alongside the filtered MGF, which then feeds the subsequent config_step2/config_step3 preprocessing stages.

## Related tools

- **matchms** (Executes the declarative YAML-configured filter stacks (library_cleaning_1.yaml, library_cleaning_2.yaml) against the MGF library; a patched version fixes _get_neutral_mass for [M]+/[M]− spectra with zero reported charge) — https://github.com/matchms/matchms
- **Python** (Runtime environment (Python 3.10 or higher) in which the two-round matchms preprocessing pipeline is executed)
- **spectraverse-analysis** (Provides the preprocessing code and the library_cleaning_1.yaml / library_cleaning_2.yaml filter configurations used to compile Spectraverse) — https://github.com/skinniderlab/spectraverse-analysis

## Evaluation signals

- Round-2 output retains all round-1 filters (HARMONIZE_METADATA_FIELD_NAMES through NORMALIZE_INTENSITY) plus REQUIRE_COMPLETE_METADATA and REQUIRE_COMPLETE_ANNOTATION, verifiable by inspecting library_cleaning_2.yaml
- No retained spectrum has a parent mass deviating from its SMILES-derived neutral mass by more than 0.1 Da, or an adduct inconsistent with its precursor m/z (within 0.1 Da) or ionization mode
- No retained spectrum is annotated with [M]+ or [M]− adducts reporting a charge of zero, confirming the patched _get_neutral_mass was active
- A per-filter removal-count report exists covering both rounds, and the filtered MGF lands in the data/ directory ready for config_step2/config_step3
- All retained spectra carry complete metadata (valid precursor m/z and ionization mode) and valid annotations

## Limitations

- Requires the modified matchms build (matchms/matchms.zip replacing the original matchms libraries); stock matchms will not ignore [M]+/[M]− spectra with reported charge zero
- Requires Python 3.10 or higher and a conda or pip environment created from environment.yml
- The authors note curation relied on manual review of only several thousand MS/MS spectra, so additional pitfalls may remain in the underlying data

## Evidence

- [methods] Run matchms using filters specified in matchms/library_cleaning_1.yaml.: "Run matchms using filters specified in matchms/library_cleaning_1.yaml."
- [methods] Run matchms again using filters specified in matchms/library_cleaning_2.yaml.: "Run matchms again using filters specified in matchms/library_cleaning_2.yaml."
- [methods] this version fixes the function _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that had a reported charge of zero.: "this version fixes the function _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that had a reported charge of zero."
- [methods] Place raw data files (MGF format) in the `data/` directory.: "Place raw data files (MGF format) in the `data/` directory."
- [readme] This repository contains code used to preprocess and harmonize MS/MS spectra and their associated metadata in the compilation of Spectraverse.: "This repository contains code used to preprocess and harmonize MS/MS spectra and their associated metadata in the compilation of Spectraverse."
