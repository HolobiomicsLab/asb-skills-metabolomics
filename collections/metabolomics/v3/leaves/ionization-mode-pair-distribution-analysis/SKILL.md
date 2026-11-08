---
name: ionization-mode-pair-distribution-analysis
description: Use when preparing training pairs for MS2DeepScore 2.0 Siamese model
  training and you need to rebalance pair selection over the Tanimoto score range
  and per-molecule frequencies, especially to reduce training bias and inspect ionization-mode
  pair coverage.
when_to_use_negative:
- 'At inference time: computing MS2DeepScore similarity matrices or embeddings between
  user spectra — pair sampling is a training-time bias-reduction step, not a prediction
  step'
- When the goal is filtering unreliable spectra with high predicted MSE — that is
  the embedding evaluator / uncertainty filtering, a separate skill
- When the aim is cross-ionization-mode molecular networking or UMAP embedding visualization
  of already-trained model outputs
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: ms2deepscore
  role: Provides the pair sampling algorithm, pair sampling tutorial, and training
    wrapper (train_ms2deepscore_wrapper) used to build balanced training pairs for
    MS2DeepScore 2.0
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Framework in which the MS2DeepScore model and its training (including sampled
    pair feeds) are implemented
- name: matchms
  role: Cleaning pipeline (DEFAULT_FILTERS) used to prepare spectra and their InChIKey/ionmode
    metadata before pair sampling and training
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:b4b0b86fc132ef491ca73657d666dfecfedefa8c3702bd30c09808e2b0318379
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - PyTorch
  - matchms
  - ms2deepscore
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# ionization-mode-pair-distribution-analysis

## Summary

Reconstructs and applies MS2DeepScore 2.0's balanced spectrum-pair sampling algorithm, which samples training pairs equally across 10 Tanimoto score bins while capping per-molecule sampling frequency disparity below 15%, and reports the distribution of selected pairs across ionization-mode combinations (positive-positive, negative-negative, positive-negative).

## When to use

Use when preparing training pairs for MS2DeepScore 2.0 Siamese model training and you need to rebalance pair selection over the Tanimoto score range and per-molecule frequencies, especially to reduce training bias and inspect ionization-mode pair coverage.

## When NOT to use

- At inference time: computing MS2DeepScore similarity matrices or embeddings between user spectra — pair sampling is a training-time bias-reduction step, not a prediction step
- When the goal is filtering unreliable spectra with high predicted MSE — that is the embedding evaluator / uncertainty filtering, a separate skill
- When the aim is cross-ionization-mode molecular networking or UMAP embedding visualization of already-trained model outputs

## Inputs

- Cleaned annotated MS/MS spectra with InChIKey and ionmode metadata (e.g., MGF/MSP/mzML after matchms DEFAULT_FILTERS)
- Structural fingerprints for Tanimoto score computation
- SettingsMS2Deepscore configuration (spectrum_file_path, additional_metadata such as ionmode CategoricalToBinary and precursor_mz StandardScaler, ionisation_mode='both')

## Outputs

- sampling_report.csv with per-Tanimoto-bin pair counts, per-molecule sampling counts, and max-vs-average count ratio
- Balanced pair selection per Tanimoto bin, annotated with ionization-mode pair combinations (positive-positive, negative-negative, positive-negative)
- Reusable pair-sampling implementation module

## How to apply

Compute Tanimoto structural similarity scores between candidate molecule pairs from their fingerprints, using InChIKeys and ionization-mode metadata carried on the cleaned spectra (e.g., MGF processed with the matchms cleaning pipeline). Partition the Tanimoto score range into 10 bins and set the objective of equal pair counts per bin. Implement a pair sampler that tracks a sampling count for every molecule and rejects any candidate pair whose inclusion would push a molecule above the maximum sampling count, limiting per-molecule frequency disparity to <15%. Iteratively draw pairs per bin until bin targets or the per-molecule cap is reached, recording which ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) were selected per bin. Write a sampling_report.csv summarizing per-bin pair counts, per-molecule sampling counts, and the max-vs-average count ratio; the article notes the algorithm balances Tanimoto bins and molecule frequency but does not enforce balanced sampling of ionization-mode pairs, so inspect this breakdown explicitly.

## Related tools

- **ms2deepscore** (Provides the pair sampling algorithm, pair sampling tutorial, and training wrapper (train_ms2deepscore_wrapper) used to build balanced training pairs for MS2DeepScore 2.0) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Framework in which the MS2DeepScore model and its training (including sampled pair feeds) are implemented)
- **matchms** (Cleaning pipeline (DEFAULT_FILTERS) used to prepare spectra and their InChIKey/ionmode metadata before pair sampling and training) — https://github.com/matchms/matchms

## Examples

```
settings = SettingsMS2Deepscore(spectrum_file_path=spectrum_file, ionisation_mode='both'); train_ms2deepscore_wrapper(settings, SettingsEmbeddingEvaluator())
```

## Evaluation signals

- sampling_report.csv exists and shows every one of the 10 Tanimoto bins contains sampled pairs with (near-)equal pair counts
- Every molecule's sampling count stays within the <15% disparity threshold relative to the maximum sampling count (max-vs-average count ratio ≤ ~1.15)
- Report records which ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) were selected per bin
- Per-molecule sampling counts are present for all molecules in the training set, with no molecule systematically excluded
- Re-running the sampler on the same spectra and settings yields a comparable bin/molecule balance (deterministic balance criteria hold regardless of random draw order)

## Limitations

- The new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs; unbalanced ionization-mode sampling may have limited negative vs. negative prediction performance
- Differences in Tanimoto score distributions may result in not having an equal number of pairs per Tanimoto bin for each ionization mode
- Pair sampling quality has to be checked and potentially re-optimized for new datasets; particularly for smaller training sets the pair sampling can be suboptimal if not checked

## Evidence

- [intro] Intro finding: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [discussion] Discussion on ionization-mode imbalance: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [discussion] Discussion on per-bin ionization-mode counts: "this might result in not having an equal number of pairs per Tanimoto bin for each ionization mode"
- [readme] README advice on checking pair sampling: [evidence span withheld — anchor status: external_doc]
