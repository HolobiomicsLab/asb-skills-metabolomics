---
name: spectral-data-augmentation
description: Use when training an MS2DeepScore-style Siamese model and you need to
  augment binned spectra (10,000 equal-width bins over 10–1000 m/z, square-root transformed
  intensities) to improve robustness.
when_to_use_negative:
- The spectra are still raw peak lists — filtering (removing peaks <0.1% max intensity,
  max 1000 peaks) and binning into the 10,000 bins must happen first
- You are only computing similarities with a pre-trained MS2DeepScore model; augmentation
  is a training-time technique, not needed at inference
- Augmenting validation/test spectra — the article applies augmentation to training
  spectra only, so applying it to evaluation data would invalidate the RMSE (~0.13–0.2)
  benchmarks
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore
  role: The training framework whose binned spectra are augmented and which consumes
    the augmented spectra as input to the Siamese neural network
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Provides the spectrum import/handling and cleaning pipeline that produces
    the cleaned spectra before binning and augmentation
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:ee9e7234dad1417b0716a51553d0e5caeebe28dd9cb795db02a249f820265f58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-data-augmentation

## Summary

Applies three random augmentations—low-intensity peak removal, intensity jitter, and new-peak addition—to binned MS/MS spectra during MS2DeepScore training, so the Siamese network learns structural similarity predictions that are robust to spectral noise rather than memorizing exact binned peak patterns.

## When to use

Use when training an MS2DeepScore-style Siamese model and you need to augment binned spectra (10,000 equal-width bins over 10–1000 m/z, square-root transformed intensities) to improve robustness.

## When NOT to use

- The spectra are still raw peak lists — filtering (removing peaks <0.1% max intensity, max 1000 peaks) and binning into the 10,000 bins must happen first
- You are only computing similarities with a pre-trained MS2DeepScore model; augmentation is a training-time technique, not needed at inference
- Augmenting validation/test spectra — the article applies augmentation to training spectra only, so applying it to evaluation data would invalidate the RMSE (~0.13–0.2) benchmarks

## Inputs

- Binned MS/MS training spectra (10,000 equal-width bins over 10–1000 m/z, square-root transformed intensities)
- Augmentation configuration (removal fraction bound, jitter factor range, number and intensity range of added peaks)

## Outputs

- Augmented binned spectra with identical shape and dtype to the input, consumed by the Siamese training loop each epoch/batch

## How to apply

Operate on the binned training spectra (after binning into 10,000 equal-width bins over 10–1000 m/z and square-root transformation), not on raw peak lists. Apply the three augmentations in sequence to each training example with independent random draws per epoch/batch: (1) peak removal — randomly delete a random fraction (0–20%) of non-zero bins whose pre-transformation intensity is below 0.4; (2) intensity jitter — multiply non-zero bins by a random factor drawn uniformly within 0.6–1.4 (jitter up to ±40%); (3) random peak addition — insert 0–10 random peaks with intensities drawn uniformly between 0 and 0.01. Implement these as an augmentation class/function inside the ms2deepscore package so each parameter is configurable and bounded (removal fraction ≤ 0.2, jitter ≤ 0.4, added peaks ≤ 10 with intensity ≤ 0.01). The rationale, per the article, is that moderate random changes of peak intensities plus removal and addition of low-intensity peaks increase model robustness for the Tanimoto-score prediction task.

## Related tools

- **MS2DeepScore** (The training framework whose binned spectra are augmented and which consumes the augmented spectra as input to the Siamese neural network) — https://github.com/matchms/ms2deepscore
- **matchms** (Provides the spectrum import/handling and cleaning pipeline that produces the cleaned spectra before binning and augmentation) — https://github.com/matchms/matchms

## Examples

```
import numpy as np; jittered = binned_spectrum * np.random.uniform(0.6, 1.4, size=binned_spectrum.shape)
```

## Evaluation signals

- Augmented spectrum has the same shape and dtype as the input binned spectrum (10,000 bins)
- Fraction of non-zero bins removed is ≤ 0.2, and removed bins all had pre-transformation intensity < 0.4
- Every jitter factor drawn falls within [0.6, 1.4]; non-zero bins are scaled, zero bins remain zero
- Number of added peaks is between 0 and 10, each with intensity in [0, 0.01]
- Augmentation parameters are configurable and bounded as specified, and draws are re-sampled independently per epoch/batch

## Limitations

- Augmentation operates on binned spectra only — it cannot recover information lost by the 10,000-bin equal-width binning over 10–1000 m/z
- Only low-intensity peaks (pre-transformation intensity < 0.4) are eligible for removal, so high-intensity peak corruption is not simulated
- The article reports no ablation isolating the contribution of each of the three augmentations to the reported RMSE (~0.13–0.2 on the test set)

## Evidence

- [discussion] three forms of augmentation: "we applied three forms of data augmentation on the binned spectra."
- [other] augmentation purpose: [evidence span withheld — anchor status: not_found]
- [readme] hyperparameter configurability: [evidence span withheld — anchor status: external_doc]
