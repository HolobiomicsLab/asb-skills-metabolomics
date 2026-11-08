---
name: precursor-mass-consistency-checking
description: Use when you need to quantify how often adduct annotations in a mass
  spectral library (e.g., GNPS MGF spectra with SMILES) are wrong or can be corrected
  from SMILES plus precursor m/z alone.
when_to_use_negative:
- You need plausibility checks that consider both metadata and measured fragments
  — the article states current filters lack such checks and wrong chemical annotations
  consistent with the measured mass will go unnoticed.
- The spectra lack valid SMILES annotations, so repair_adduct_based_on_smiles cannot
  derive a neutral monoisotopic mass.
- You are repairing parent mass errors where the parent mass was calculated from the
  molar mass instead of the monoisotopic mass — that is the separate Repair Parent
  Mass Is Molar Mass filter.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: matchms
  role: Provides the repair_adduct_based_on_smiles filter and the Spectrum loading/filtering
    API (version 0.26.4) used to run the error-rate estimation
  repo: https://github.com/matchms/matchms
- name: RDKit
  role: Loads and validates SMILES annotations so the filter can derive the neutral
    monoisotopic mass from the structure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00878-1
    title: Reproducible MS/MS library cleaning pipeline in matchms
  source_content_hash: sha256:e66a79d7676a28afd717987641166b4adb34f6ef9a092d42b2253c0cbc58a389
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - RDKit
  - matchms
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# precursor-mass-consistency-checking

## Summary

This skill reproduces the error-rate estimation workflow for the 'repair adduct and parent mass based on SMILES' filter in the matchms library-cleaning pipeline, which corrects a spectrum's adduct by testing candidate adducts against the SMILES-derived neutral monoisotopic mass and the observed precursor m/z.

## When to use

Use when you need to quantify how often adduct annotations in a mass spectral library (e.g., GNPS MGF spectra with SMILES) are wrong or can be corrected from SMILES plus precursor m/z alone.

## When NOT to use

- You need plausibility checks that consider both metadata and measured fragments — the article states current filters lack such checks and wrong chemical annotations consistent with the measured mass will go unnoticed.
- The spectra lack valid SMILES annotations, so repair_adduct_based_on_smiles cannot derive a neutral monoisotopic mass.
- You are repairing parent mass errors where the parent mass was calculated from the molar mass instead of the monoisotopic mass — that is the separate Repair Parent Mass Is Molar Mass filter.

## Inputs

- GNPS public mass spectral library spectra with SMILES annotations, precursor m/z, and modified adduct metadata (error-estimation subset)
- matchms Spectrum objects loaded via matchms 0.26.4

## Outputs

- Per-spectrum filter outcome counts (adduct not derivable vs. corrected/confirmed vs. still incorrect)
- Derived/confirmed-adduct rate and residual incorrect-adduct rate (reported: 99.98% and 0.024%)
- A results file, e.g. repair_adduct_report.json

## How to apply

Load the GNPS error-estimation subset of spectra using matchms 0.26.4, with each spectrum carrying a SMILES annotation, a deliberately modified adduct field, and a precursor m/z. Apply the matchms.filtering function repair_adduct_based_on_smiles, which derives the neutral monoisotopic mass from the SMILES via RDKit and tests which common adduct (e.g. [M+H]+, [M-H]-, [M+Na]+) best explains the observed precursor m/z, then overwrites the adduct field accordingly. Count per-spectrum outcomes: spectra where the adduct could not be derived at all versus spectra whose post-filter adduct remains inconsistent with the SMILES and precursor m/z. Compute the derived/confirmed-adduct rate and the residual incorrect-adduct rate as fractions of the subset and verify against the reported values (99.98% and 0.024% respectively). The rationale is that the filter relies purely on mass consistency between the structure annotation and the precursor m/z, so any error it leaves behind is a residual annotation-inconsistency rate, not a fragment-based plausibility check.

## Related tools

- **matchms** (Provides the repair_adduct_based_on_smiles filter and the Spectrum loading/filtering API (version 0.26.4) used to run the error-rate estimation) — https://github.com/matchms/matchms
- **RDKit** (Loads and validates SMILES annotations so the filter can derive the neutral monoisotopic mass from the structure)

## Examples

```
from matchms.filtering import repair_adduct_based_on_smiles; s = repair_adduct_based_on_smiles(spectrum)
```

## Evaluation signals

- The adduct could not be derived for 0.02% of spectra in the subset
- Of the remaining 99.98% of spectra, 0.024% had an incorrect adduct after the filter
- Matchms version 0.26.4 was used to run these pipelines
- Every spectrum processed should carry a valid SMILES, InChI and InChIKey that agree with each other before filtering

## Limitations

- The filter performs no plausibility checks that consider both metadata and measured fragments; wrong chemical annotations that are consistent with the measured mass will go unnoticed.
- Fragment-matching filters (checking if fragments match the given annotation) are only a possible future expansion, not implemented.
- The filter is mass-based only, so it cannot detect a wrong SMILES that happens to be isobaric with the true structure.

## Evidence

- [methods] Corrects the adduct of a spectrum based on the SMILES and precursor m/z.: "Corrects the adduct of a spectrum based on the SMILES and precursor m/z."
- [results] For the 'Repair adduct and parent mass based on SMILES' filter, the adduct could not be derived for 0.02% of spectra, and of the remaining 99.98% of spectra, 0.024% had an incorrect adduct after the filter.: [evidence span withheld — anchor status: not_found]
- [methods] Matchms version 0.26.4 was used to run these pipelines.: "Matchms version 0.26.4 was used to run these pipelines."
- [methods] SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other.: "SMILES, InChI and InChIKey are loaded by RDKit [18] and compared to each other."
- [discussion] the instrument type or collision energies could be important fields that might require additional standardization: "the instrument type or collision energies could be important fields that might require additional standardization"
