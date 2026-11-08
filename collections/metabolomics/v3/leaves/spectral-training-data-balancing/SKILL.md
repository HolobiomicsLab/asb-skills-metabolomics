---
name: spectral-training-data-balancing
description: Use when preparing training pairs for a Siamese spectral similarity model
  (e.g. MS2DeepScore) where biased pair selection over Tanimoto score ranges or molecule
  frequencies would skew model training.
when_to_use_negative:
- When the training set is very small — the README warns the pair sampling 'can be
  suboptimal if not checked', particularly for smaller training sets, and may need
  re-optimization
- When the goal is balanced sampling of ionization-mode pair combinations — the paper
  explicitly states the sampling algorithm 'did not enforce balanced sampling of the
  different ionization mode pairs'
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore 2.0
  role: Model whose training pipeline uses the balanced pair sampler to reduce training
    bias
  repo: https://github.com/matchms/MS2DeepScore
- name: ms2deepscore
  role: Python library providing the pair sampling tutorial and training wrapper (train_ms2deepscore_wrapper)
    that consumes the sampled pairs
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Framework in which the MS2DeepScore model consuming the balanced pairs is
    implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:286011493ccb31d5a87b4de3d2687710e03b4beddc057e8003b0bc7d3f24356b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/MS2DeepScore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-training-data-balancing

## Summary

A pair sampling algorithm for training MS2DeepScore 2.0 that balances selected spectrum pairs equally across 10 Tanimoto score bins while capping per-molecule sampling frequency disparity to <15%, reducing training bias.

## When to use

Use when preparing training pairs for a Siamese spectral similarity model (e.g. MS2DeepScore) where biased pair selection over Tanimoto score ranges or molecule frequencies would skew model training.

## When NOT to use

- When the training set is very small — the README warns the pair sampling 'can be suboptimal if not checked', particularly for smaller training sets, and may need re-optimization
- When the goal is balanced sampling of ionization-mode pair combinations — the paper explicitly states the sampling algorithm 'did not enforce balanced sampling of the different ionization mode pairs'

## Inputs

- Annotated training spectra with metadata (InChIKeys, ionization mode, precursor m/z), e.g. combined_libraries.mgf
- Structural fingerprints for computing Tanimoto scores between molecule pairs

## Outputs

- Balanced set of training spectrum pairs with equal counts across 10 Tanimoto bins
- Sampling report (e.g. sampling_report.csv) with per-bin pair counts, per-molecule sampling counts, and max-vs-average count ratio
- Per-bin record of ionization-mode pair combinations selected (positive-positive, negative-negative, positive-negative)

## How to apply

Compute Tanimoto scores between candidate molecule pairs from their structural fingerprints (InChIKeys) of the annotated training spectra, including ionization-mode metadata. Partition the Tanimoto score range into 10 bins and target equal pair counts per bin. Implement a sampler that tracks a sampling count for every molecule and rejects candidate pairs whose inclusion would push a molecule above the maximum sampling count, keeping per-molecule frequency disparity within <15%. Iteratively draw pairs per bin until bin targets or the per-molecule cap is reached, recording ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) per bin. Write a statistics summary (per-bin pair counts, per-molecule counts, max-vs-average ratio) and validate that every bin contains pairs and molecule counts stay within the <15% disparity. Note that the published algorithm did not enforce balanced sampling of ionization-mode pair combinations, so check mode balance separately if needed.

## Related tools

- **MS2DeepScore 2.0** (Model whose training pipeline uses the balanced pair sampler to reduce training bias) — https://github.com/matchms/MS2DeepScore
- **ms2deepscore** (Python library providing the pair sampling tutorial and training wrapper (train_ms2deepscore_wrapper) that consumes the sampled pairs) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Framework in which the MS2DeepScore model consuming the balanced pairs is implemented)

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; train_ms2deepscore_wrapper(SettingsMS2Deepscore(spectrum_file_path='combined_libraries.mgf', ionisation_mode='both'), SettingsEmbeddingEvaluator())
```

## Evaluation signals

- Each of the 10 Tanimoto bins contains sampled pairs (equal pair counts per bin)
- Every molecule's sampling count stays within <15% disparity of the maximum sampling count relative to the average
- Sampling report records per-bin pair counts and per-molecule sampling counts, with max-vs-average count ratio computed
- Ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) per bin are recorded so mode balance can be audited

## Limitations

- The sampler did not enforce balanced sampling of ionization mode pairs; more balanced mode sampling might improve negative-vs-negative prediction performance
- Unequal numbers of pairs per Tanimoto bin per ionization mode can result from differences in Tanimoto score distributions
- Pair sampling quality must be checked and potentially re-optimized for new datasets, especially smaller training sets

## Evidence

- [discussion] sampling balances pairs over Tanimoto bins and molecule frequency: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [intro] pair sampling reduces training bias: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [readme] check pair sampling before training: [evidence span withheld — anchor status: external_doc]
- [results] sampling report validation: [evidence span withheld — anchor status: not_found]
