---
name: tanimoto-similarity-pair-sampling
description: Use when training MS2DeepScore-style Siamese spectral similarity models
  and you need to reduce training bias by sampling spectrum pairs equally over Tanimoto
  bins while limiting per-molecule sampling frequency differences.
when_to_use_negative:
- When training on a very small custom dataset without first re-checking and re-optimizing
  the pair sampling — the README warns pair sampling can be suboptimal if not checked,
  and recommends merging small datasets with larger public libraries before training
  from scratch
- When balanced sampling across ionization modes is strictly required — the algorithm
  balances Tanimoto bins and molecule frequencies but did not enforce balanced sampling
  of the different ionization mode pairs
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore 2.0
  role: Provides the new pair sampling algorithm used to generate bias-reduced training
    pairs for the Siamese model
  repo: https://github.com/matchms/ms2deepscore
- name: PyTorch
  role: Framework in which the MS2DeepScore model consuming the sampled pairs is implemented
- name: matchms
  role: Recommended cleaning pipeline to prepare spectra before pair sampling and
    training
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.03.25.586580
    title: 'Bridging polarities in metabolomics: Cross-ionization mode chemical similarity
      prediction between tandem mass spectra'
  source_content_hash: sha256:27027925f6ab89a4eb0caa0c32e349cae6d04b37151ce3f8c739ff6687b143c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore 2.0
  - PyTorch
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tanimoto-similarity-pair-sampling

## Summary

A balanced pair-sampling algorithm for training MS2DeepScore 2.0 Siamese models that draws spectrum pairs equally across 10 Tanimoto-score bins while capping per-molecule sampling-frequency disparity to <15%, reducing training biases introduced by over-sampled molecules and under-represented Tanimoto ranges.

## When to use

Use when training MS2DeepScore-style Siamese spectral similarity models and you need to reduce training bias by sampling spectrum pairs equally over Tanimoto bins while limiting per-molecule sampling frequency differences.

## When NOT to use

- When training on a very small custom dataset without first re-checking and re-optimizing the pair sampling — the README warns pair sampling can be suboptimal if not checked, and recommends merging small datasets with larger public libraries before training from scratch
- When balanced sampling across ionization modes is strictly required — the algorithm balances Tanimoto bins and molecule frequencies but did not enforce balanced sampling of the different ionization mode pairs

## Inputs

- Cleaned (matchms-filtered) training spectra with InChIKeys and ionmode metadata (e.g. combined_libraries.mgf)
- Structural fingerprints of the candidate molecules for Tanimoto score computation
- Sampling settings (number of Tanimoto bins = 10, maximum per-molecule sampling count) via SettingsMS2Deepscore

## Outputs

- Sampled spectrum pairs balanced across 10 Tanimoto bins, stratified by ionization-mode pair combination (pos-pos, neg-neg, pos-neg)
- Sampling statistics report (per-bin pair counts, per-molecule sampling counts, max-vs-average count ratio), e.g. sampling_report.csv

## How to apply

Load annotated spectral metadata (InChIKeys and ionization modes) for the training spectra and compute Tanimoto scores between candidate molecule pairs from their structural fingerprints. Partition the Tanimoto score range into 10 bins and set the objective of equal pair counts per bin. Run the sampler, which tracks a sampling count for every molecule and rejects candidate pairs whose inclusion would push a molecule's count above the maximum sampling count, keeping per-molecule frequency disparity below 15%. Iteratively draw pairs per bin until bin targets or the per-molecule cap is reached, recording the ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) selected per bin. Check the sampling quality and potentially re-optimize parameters for new datasets — particularly for smaller training sets the pair sampling can be suboptimal if not checked — then hand the sampled pairs to model training in PyTorch.

## Related tools

- **MS2DeepScore 2.0** (Provides the new pair sampling algorithm used to generate bias-reduced training pairs for the Siamese model) — https://github.com/matchms/ms2deepscore
- **PyTorch** (Framework in which the MS2DeepScore model consuming the sampled pairs is implemented)
- **matchms** (Recommended cleaning pipeline to prepare spectra before pair sampling and training) — https://github.com/matchms/matchms

## Examples

```
from ms2deepscore.SettingsMS2Deepscore import SettingsMS2Deepscore; settings = SettingsMS2Deepscore(spectrum_file_path="combined_libraries.mgf", ionisation_mode="both")
```

## Evaluation signals

- Each of the 10 Tanimoto bins contains sampled pairs (per-bin pair counts reported)
- Every molecule's sampling count stays within the <15% disparity bound relative to the maximum/average sampling count
- The sampling report records which ionization-mode pair combinations (positive-positive, negative-negative, positive-negative) were selected per bin
- Per-bin pair counts are approximately equal, acknowledging unequal counts can arise across ionization modes due to differences in Tanimoto score distributions

## Limitations

- The sampling algorithm did not enforce balanced sampling of the different ionization mode pairs; more balanced sampling might improve negative vs. negative ionization mode prediction performance
- Unequal numbers of pairs per Tanimoto bin per ionization mode can result from differences in Tanimoto score distributions
- For smaller training sets, pair sampling can be suboptimal if not checked and potentially re-optimized

## Evidence

- [intro] new pair sampling algorithm reduces training bias: "we introduce a new pair sampling algorithm that reduces biases introduced during model training"
- [discussion] ionization-mode pair balance not enforced: "our new sampling algorithm did not enforce balanced sampling of the different ionization mode pairs"
- [discussion] unequal per-bin pair counts per ionization mode: "this might result in not having an equal number of pairs per Tanimoto bin for each ionization mode"
- [readme] pair sampling quality must be checked per dataset: [evidence span withheld — anchor status: external_doc]
