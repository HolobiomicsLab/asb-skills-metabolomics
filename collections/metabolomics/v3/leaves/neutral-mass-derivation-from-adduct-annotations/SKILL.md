---
name: neutral-mass-derivation-from-adduct-annotations
description: Use when cleaning MS/MS spectral libraries (MGF format) where adduct
  metadata must yield chemically coherent neutral masses, especially when [M]+ or
  [M]− adducts are annotated with a reported charge of zero.
when_to_use_negative:
- Input spectra lack adduct annotations entirely (or lack precursor m/z and ionization
  mode), since round 2 REQUIRE_COMPLETE_METADATA filters would simply discard them
- The target library contains non-small-molecule spectra outside the Spectraverse
  small-molecule curation scope
- You need the unmodified upstream matchms behavior for a non-Spectraverse pipeline
  — the required matchms.zip patch changes _get_neutral_mass semantics
edam_operation: http://edamontology.org/operation_3799
tools:
- name: matchms
  role: Patched version performs the metadata harmonization, annotation coherence
    checks, and neutral-mass derivation (_get_neutral_mass) driven by the library_cleaning
    YAML configs
  repo: https://github.com/matchms/matchms
- name: spectraverse-analysis
  role: Repository supplying the patched matchms zip, library_cleaning_1.yaml/library_cleaning_2.yaml
    configs, and pipeline scripts
  repo: https://github.com/skinniderlab/spectraverse-analysis
- name: spectraverse
  role: Source pipeline repository cloned and installed in development mode to run
    the two-round cleaning
  repo: https://github.com/GuptaVishu2002/spectraverse
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:323ac56e9d0ff862f462a29c17b6e7089f80bc90f828807762c10dd7022134b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  - spectraverse
  - spectraverse-analysis
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Neutral mass derivation from adduct annotations in MS/MS library cleaning

## Summary

Derives and validates compound neutral masses from adduct annotations (e.g., [M]+, [M]−, [M+H]+) during matchms-based MS/MS library curation, using a patched matchms whose _get_neutral_mass function excludes [M]+/[M]− spectra that report a net charge of zero.

## When to use

Use when cleaning MS/MS spectral libraries (MGF format) where adduct metadata must yield chemically coherent neutral masses, especially when [M]+ or [M]− adducts are annotated with a reported charge of zero.

## When NOT to use

- Input spectra lack adduct annotations entirely (or lack precursor m/z and ionization mode), since round 2 REQUIRE_COMPLETE_METADATA filters would simply discard them
- The target library contains non-small-molecule spectra outside the Spectraverse small-molecule curation scope
- You need the unmodified upstream matchms behavior for a non-Spectraverse pipeline — the required matchms.zip patch changes _get_neutral_mass semantics

## Inputs

- Merged MS/MS library in MGF format (e.g., Spectraverse compilation, 488,630 spectra from 44,237 unique small molecules)
- Declarative matchms filter configurations matchms/library_cleaning_1.yaml and matchms/library_cleaning_2.yaml
- Patched matchms (matchms/matchms.zip replacing original libraries, with fixed _get_neutral_mass)

## Outputs

- Filtered, metadata-repaired MGF written to the data/ directory for subsequent config_step2/config_step3 preprocessing stages
- Per-filter removal-count report tallying spectra removed in both matchms rounds

## How to apply

Load the merged MS/MS library as MGF into the patched matchms environment: clone https://github.com/GuptaVishu2002/spectraverse.git, pip install -e ., then clone https://github.com/matchms/matchms and replace its original libraries with matchms/matchms.zip — this patch fixes _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that report a charge of zero. Execute round 1 with matchms/library_cleaning_1.yaml (HARMONIZE_METADATA_FIELD_NAMES, DERIVE_METADATA_IN_WRONG_FIELD, HARMONIZE_METADATA_ENTRIES, DERIVE_MISSING_METADATA, REPAIR_ANNOTATION, NORMALIZE_INTENSITY) so that adduct and ionization-mode fields are harmonized before mass derivation. Then run round 2 with matchms/library_cleaning_2.yaml, which retains round-1 filters and adds REQUIRE_COMPLETE_METADATA (valid precursor m/z and ionization mode) and REQUIRE_COMPLETE_ANNOTATION, verifying the parent mass against the SMILES within 0.1 Da and that the adduct matches both the precursor m/z and ionization mode within 0.1 Da. The rationale is that a neutral mass computed from an adduct like [M]+ with charge 0 is chemically incoherent and would corrupt downstream parent-mass checks, so such spectra are excluded from derivation and can be flagged/removed. Tally spectra removed per filter across both rounds into a per-filter removal-count report alongside the output MGF.

## Related tools

- **matchms** (Patched version performs the metadata harmonization, annotation coherence checks, and neutral-mass derivation (_get_neutral_mass) driven by the library_cleaning YAML configs) — https://github.com/matchms/matchms
- **spectraverse-analysis** (Repository supplying the patched matchms zip, library_cleaning_1.yaml/library_cleaning_2.yaml configs, and pipeline scripts) — https://github.com/skinniderlab/spectraverse-analysis
- **spectraverse** (Source pipeline repository cloned and installed in development mode to run the two-round cleaning) — https://github.com/GuptaVishu2002/spectraverse

## Examples

```
git clone https://github.com/GuptaVishu2002/spectraverse.git && pip install -e . && conda activate spectraverse
```

## Evaluation signals

- No spectrum with a [M]+ or [M]− adduct reporting net charge zero contributes a neutral mass after the patched _get_neutral_mass runs
- REQUIRE_COMPLETE_ANNOTATION passes only spectra whose parent mass matches the SMILES-derived mass within 0.1 Da and whose adduct matches precursor m/z and ionization mode within 0.1 Da
- Every round-1 filter (HARMONIZE_METADATA_FIELD_NAMES through NORMALIZE_INTENSITY) has a logged per-filter removal count in the report
- Output MGF exists in data/ and is consumable by the config_step2/config_step3 stages in the prescribed interleaved order

## Limitations

- The patch must be applied by replacing original matchms libraries with matchms/matchms.zip; running stock matchms silently restores the buggy _get_neutral_mass behavior
- Curation relied on manual review of only several thousand spectra, so annotation pitfalls may remain beyond what was reviewed
- Spectra missing precursor m/z or ionization mode are removed in round 2, so legitimate spectra with incomplete metadata are lost

## Evidence

- [methods] Patched _get_neutral_mass fix: "this version fixes the function _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that had a reported charge of zero."
- [methods] Round 1 YAML filter execution: "Run matchms using filters specified in matchms/library_cleaning_1.yaml."
- [methods] Round 2 YAML filter execution: "Run matchms again using filters specified in matchms/library_cleaning_2.yaml."
- [methods] Patched matchms installation procedure: "use the zip file at matchms/matchms.zip and replace it original matchms libraries"
- [readme] Repository purpose: "This repository contains code used to preprocess and harmonize MS/MS spectra and their associated metadata in the compilation of Spectraverse."
