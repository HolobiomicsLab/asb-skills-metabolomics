---
name: per-molecule-sampling-count-tracking
description: Use when preparing training pairs for a Siamese spectral similarity model
  (e.g., MS2DeepScore) and you must avoid bias from uneven Tanimoto-score coverage
  or over-sampled molecules in a spectral library.
when_to_use_negative:
- When the training dataset is very small, pair sampling may be suboptimal and should
  be checked/re-optimized rather than run with defaults
- When the goal is balanced ionization-mode pair sampling (e.g., improving negative
  vs. negative mode predictions) — the published algorithm did not enforce balanced
  sampling of ionization-mode pairs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: MS2DeepScore 2.0
  role: Model whose training pipeline uses the balanced pair sampler; settings and
    training wrapper configure the sampling
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Implementation framework for the model and its training/data-sampling code
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:4b11892cbf4740f8bb531d1df95130832442330acf2b3b59c8ea6902bd07b918
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# per-molecule-sampling-count-tracking

## Summary

A pair-sampling procedure for training MS2DeepScore 2.0 that tracks a sampling count for every molecule and enforces equal pair counts across 10 Tanimoto-score bins, so no molecule is sampled disproportionately and training-data biases are reduced.

## When to use

Use when preparing training pairs for a Siamese spectral similarity model (e.g., MS2DeepScore) and you must avoid bias from uneven Tanimoto-score coverage or over-sampled molecules in a spectral library.

## When NOT to use

- When the training dataset is very small, pair sampling may be suboptimal and should be checked/re-optimized rather than run with defaults
- When the goal is balanced ionization-mode pair sampling (e.g., improving negative vs. negative mode predictions) — the published algorithm did not enforce balanced sampling of ionization-mode pairs

## Inputs

- Annotated training spectra with metadata (InChIKeys, ionization mode, precursor m/z), e.g., a combined .mgf library file
- Structural fingerprints for computing Tanimoto scores between candidate molecule pairs
- Sampling settings (e.g., SettingsMS2Deepscore with additional_metadata for ionmode and precursor_mz)

## Outputs

- A balanced set of training pairs with (approximately) equal counts per Tanimoto bin (10 bins) and per-molecule sampling counts within <15% of the maximum
- A sampling statistics report (e.g., sampling_report.csv) with per-bin pair counts, per-molecule sampling counts, and max-vs-average count ratio
- Per-bin records of ionization-mode pair combinations (positive-positive, negative-negative, positive-negative)

## How to apply

Compute Tanimoto scores between candidate molecule pairs from their structural fingerprints (InChIKeys from the annotated spectral metadata) and partition the Tanimoto range into 10 bins with the objective of equal pair counts per bin. Implement a sampler that maintains a sampling count for every molecule and rejects candidate pairs whose inclusion would push a molecule's count above the current maximum sampling count, capping per-molecule frequency disparity at <15% relative to the average. Iteratively draw pairs per bin until bin targets or the per-molecule cap is reached, recording which molecules and which ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) were selected per bin. Write per-bin pair counts, per-molecule sampling counts, and the max-vs-average count ratio to a report file (e.g., sampling_report.csv) for validation. Note that the published algorithm does not enforce balanced sampling across ionization-mode pairs, so check the mode-pair distribution separately, and re-optimize the sampling for new datasets — particularly small training sets — using the provided pair sampling tutorial.

## Related tools

- **MS2DeepScore 2.0** (Model whose training pipeline uses the balanced pair sampler; settings and training wrapper configure the sampling) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Implementation framework for the model and its training/data-sampling code)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; from ms2deepscore.wrapper_functions.training_wrapper_functions import train_ms2deepscore_wrapper; settings = SettingsMS2Deepscore(spectrum_file_path='./combined_libraries.mgf', ionisation_mode='both'); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Each of the 10 Tanimoto bins contains sampled pairs (no empty bins)
- Every molecule's sampling count stays within the <15% disparity of the maximum sampling count relative to the average
- The max-vs-average count ratio in the sampling report is ≈1 (≤ ~1.15)
- Per-bin pair counts are approximately equal across the Tanimoto range
- The distribution of ionization-mode pair combinations per bin is inspected explicitly, since balance over modes is NOT guaranteed by the algorithm

## Limitations

- The new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs
- Differences in Tanimoto score distributions between ionization modes can result in an unequal number of pairs per Tanimoto bin for each ionization mode
- Pair sampling quality has to be checked and potentially re-optimized for new datasets, and can be suboptimal for small training sets
- More balanced sampling over ionization modes might be needed to improve negative vs. negative ionization mode prediction performance

## Evidence

- [discussion] Pair sampling algorithm balances pairs over Tanimoto bins and molecule sampling frequency but did not enforce balanced sampling of ionization mode pairs: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [intro] Motivation for the sampler: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [readme] Need to check and re-optimize pair sampling per dataset: [evidence span withheld — anchor status: external_doc]
- [discussion] Unequal pairs per bin per ionization mode: "this might result in not having an equal number of pairs per Tanimoto bin for each ionization mode"
- [discussion] Benefit of more balanced ionization-mode sampling: "Having more balanced sampling over the ionization modes might improve model performance for negative vs. negative ionization mode predictions"
