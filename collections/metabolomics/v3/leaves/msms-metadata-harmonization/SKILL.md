---
name: msms-metadata-harmonization
description: Use when a raw or merged MS/MS spectral library in MGF format contains
  inconsistent metadata fields, missing precursor m/z or ionization mode, or annotations
  whose parent mass/adduct contradict the precursor m/z and SMILES.
when_to_use_negative:
- Input spectra are already fully harmonized and annotated (e.g. output of a prior
  library_cleaning_2 pass) — reapplying the round-2 hard filters would only re-derive
  removal counts without new benefit
- The task is noise-ion removal or spectral similarity scoring (e.g. spectral denoising
  or cosine/entropy similarity), not metadata curation — those are separate skills
  in this pipeline
- Data are not MS/MS spectral libraries (e.g. feature tables or LC-MS chromatograms
  without peak lists and SMILES annotations), since REQUIRE_COMPLETE_ANNOTATION depends
  on SMILES-derived parent masses
tools:
- name: matchms
  role: Python package whose filter pipeline (configured via library_cleaning_1.yaml
    and library_cleaning_2.yaml) performs the metadata harmonization, annotation repair,
    and incoherent-annotation filtering; a patched copy (matchms.zip) replaces the
    original matchms libraries to fix _get_neutral_mass
  repo: https://github.com/matchms/matchms
- name: Python
  role: Runtime (Python 3.10 or higher) for executing the two-round matchms preprocessing
    workflow
- name: spectraverse-analysis
  role: Repository supplying the environment.yml, filter YAML configurations, patched
    matchms zip, and preprocessing scripts for this stage
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:285efc4f1362b325ef5b11878f2592b6bdc911d10493072cb602b72e63684170
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - matchms
  - spectraverse-analysis
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# msms-metadata-harmonization

## Summary

A two-round matchms filtering stage from the Spectraverse pipeline that harmonizes MS/MS metadata (field names, entries, adducts, ionization modes), repairs annotations, and then removes spectra with incomplete metadata or chemically incoherent annotations, writing a cleaned MGF plus per-filter removal counts.

## When to use

Use when a raw or merged MS/MS spectral library in MGF format contains inconsistent metadata fields, missing precursor m/z or ionization mode, or annotations whose parent mass/adduct contradict the precursor m/z and SMILES.

## When NOT to use

- Input spectra are already fully harmonized and annotated (e.g. output of a prior library_cleaning_2 pass) — reapplying the round-2 hard filters would only re-derive removal counts without new benefit
- The task is noise-ion removal or spectral similarity scoring (e.g. spectral denoising or cosine/entropy similarity), not metadata curation — those are separate skills in this pipeline
- Data are not MS/MS spectral libraries (e.g. feature tables or LC-MS chromatograms without peak lists and SMILES annotations), since REQUIRE_COMPLETE_ANNOTATION depends on SMILES-derived parent masses

## Inputs

- Merged MS/MS spectral library in MGF format (raw data files placed in the data/ directory)
- matchms/library_cleaning_1.yaml declarative filter configuration
- matchms/library_cleaning_2.yaml declarative filter configuration
- Patched matchms installation (matchms.zip replacing the original matchms libraries)

## Outputs

- Filtered, metadata-repaired MGF written to the data/ directory, feeding subsequent config_step2/config_step3 preprocessing stages
- Per-filter removal-count report tallying spectra removed by each filter across both rounds

## How to apply

Load the merged MS/MS library in MGF format together with the declarative filter configurations matchms/library_cleaning_1.yaml and library_cleaning_2.yaml. Execute round 1 with the filters HARMONIZE_METADATA_FIELD_NAMES, DERIVE_METADATA_IN_WRONG_FIELD, HARMONIZE_METADATA_ENTRIES, DERIVE_MISSING_METADATA, REPAIR_ANNOTATION, and NORMALIZE_INTENSITY, running under a patched matchms whose _get_neutral_mass ignores spectra annotated with [M]+ or [M]− adducts reporting a charge of zero. Then run round 2 with library_cleaning_2.yaml, which retains the round-1 filters and additionally applies REQUIRE_COMPLETE_METADATA (valid precursor m/z and ionization mode) and REQUIRE_COMPLETE_ANNOTATION, verifying parent mass against the SMILES within 0.1 Da, and the adduct against precursor m/z within 0.1 Da and against the ionization mode. Write the filtered MGF to the data/ directory so it feeds the subsequent config_step2/config_step3 preprocessing stages, and tally spectra removed by each filter in both rounds into a per-filter removal-count report alongside the output.

## Related tools

- **matchms** (Python package whose filter pipeline (configured via library_cleaning_1.yaml and library_cleaning_2.yaml) performs the metadata harmonization, annotation repair, and incoherent-annotation filtering; a patched copy (matchms.zip) replaces the original matchms libraries to fix _get_neutral_mass) — https://github.com/matchms/matchms
- **Python** (Runtime (Python 3.10 or higher) for executing the two-round matchms preprocessing workflow)
- **spectraverse-analysis** (Repository supplying the environment.yml, filter YAML configurations, patched matchms zip, and preprocessing scripts for this stage) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
conda activate spectraverse; python -m matchms --config matchms/library_cleaning_1.yaml; python -m matchms --config matchms/library_cleaning_2.yaml
```

## Evaluation signals

- Output MGF spectra all carry valid precursor m/z and ionization mode metadata (REQUIRE_COMPLETE_METADATA invariant)
- For every retained spectrum, the parent mass computed from the SMILES matches the annotation within 0.1 Da, and the adduct is consistent with precursor m/z within 0.1 Da and with the ionization mode
- No retained spectrum is annotated with [M]+ or [M]− adducts reporting a charge of zero (the _get_neutral_mass patch condition)
- The per-filter removal-count report lists counts for every filter in both rounds and their sum reconciles with input-minus-output spectrum counts
- The output MGF lands in the data/ directory and is consumable by the subsequent config_step2/config_step3 stages in the prescribed interleaved order

## Limitations

- The round-2 REQUIRE_COMPLETE_ANNOTATION filters hard-remove spectra; valid spectra lacking complete metadata or SMILES-verifiable parent masses within 0.1 Da will be discarded rather than repaired
- Curation relied on manual review of only several thousand spectra; pitfalls may remain beyond what was manually reviewed
- The patched _get_neutral_mass behavior depends on replacing the original matchms libraries with the provided matchms.zip; using unpatched matchms will not ignore [M]+/[M]− adducts with a reported charge of zero
- No dataset names or accession IDs are prescribed in the methods — the user must supply the raw MGF files in data/ themselves

## Evidence

- [methods] library_cleaning_1.yaml filters: "Run matchms using filters specified in matchms/library_cleaning_1.yaml."
- [methods] _get_neutral_mass fix: "this version fixes the function _get_neutral_mass to ignore spectra annotated with [M]+ or [M]− adducts that had a reported charge of zero."
- [readme] patched matchms replacement: "use the zip file at matchms/matchms.zip and replace it original matchms libraries"
- [readme] MGF input preparation: "Prepare your data: Place raw data files (MGF format) in the `data/` directory."
- [methods] library_cleaning_2 filters: "Run matchms again using filters specified in matchms/library_cleaning_2.yaml."
