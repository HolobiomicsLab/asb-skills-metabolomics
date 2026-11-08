---
name: binned-spectrum-peak-manipulation
description: Use when training an MS2DeepScore (or similar Siamese embedding) model
  on binned mass spectra and you need to augment each training example with moderate
  random peak-level perturbations.
when_to_use_negative:
- On validation or test spectra — augmentation is applied to training data only, so
  perturbing the 500-InChIKey validation or test sets would corrupt benchmark results
- On raw (unbinned) peak lists — the augmentation operates on binned spectra after
  the 10–1000 m/z binning and square-root transformation steps, not on raw peak lists
- When intensities have not been square-root transformed — the low-intensity removal
  threshold (pre-transformation intensity below 0.4) assumes transformed intensities,
  so applying it to linearly-scaled intensities misclassifies which bins are eligible
  for removal
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: MS2DeepScore
  role: Siamese neural network trained on the augmented binned spectra to predict
    Tanimoto scores between spectrum pairs; the augmentation lives in the ms2deepscore
    package
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Library in which MS2DeepScore is integrated for spectrum import, handling
    and comparisons, including the filtering (peak removal, 1000-peak cap) that precedes
    binning and augmentation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:51e267867b8a082498ad6d438e1a30bed819e2c493e589583e80ed5372110979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# binned-spectrum-peak-manipulation

## Summary

Data augmentation for MS2DeepScore training that perturbs binned MS/MS spectra (10,000 equal-width bins over 10–1000 m/z) by removing low-intensity peaks, jittering intensities, and adding new random peaks, to increase model robustness.

## When to use

Use when training an MS2DeepScore (or similar Siamese embedding) model on binned mass spectra and you need to augment each training example with moderate random peak-level perturbations.

## When NOT to use

- On validation or test spectra — augmentation is applied to training data only, so perturbing the 500-InChIKey validation or test sets would corrupt benchmark results
- On raw (unbinned) peak lists — the augmentation operates on binned spectra after the 10–1000 m/z binning and square-root transformation steps, not on raw peak lists
- When intensities have not been square-root transformed — the low-intensity removal threshold (pre-transformation intensity below 0.4) assumes transformed intensities, so applying it to linearly-scaled intensities misclassifies which bins are eligible for removal

## Inputs

- Binned spectrum arrays: 10,000 equal-width bins spanning 10–1000 m/z, square-root-transformed peak intensities
- Training split spectra (14,062 unique InChIKeys) drawn per batch of size 32

## Outputs

- Augmented binned spectrum arrays of identical shape and dtype, ready for the Siamese network embedding step
- Randomly varied copies of each training spectrum per epoch/batch

## How to apply

Operate on binned spectra produced by binning peaks into 10,000 equally-sized bins ranging from 10 to 1000 m/z, after square-root transformation of intensities. Apply three augmentations in sequence: (1) randomly remove a fraction of non-zero bins whose pre-transformation intensity is below 0.4 (removal fraction 0–20% of such bins); (2) multiply non-zero bin intensities by a random factor within 0.6–1.4, i.e. jitter up to ±40%; (3) insert 0–10 random new peaks with intensities drawn uniformly between 0 and 0.01. Draw random parameters independently for each training example so every epoch sees different augmentations. Rationale: moderate random changes of peak intensities plus removal and addition of low intensity peaks increase robustness without altering the spectrum's bin structure — the output array shape and dtype must remain unchanged, and all parameters should stay within the stated bounds (removal fraction ≤ 0.2, jitter ≤ 0.4, added peaks ≤ 10 with intensity ≤ 0.01).

## Related tools

- **MS2DeepScore** (Siamese neural network trained on the augmented binned spectra to predict Tanimoto scores between spectrum pairs; the augmentation lives in the ms2deepscore package) — https://github.com/matchms/ms2deepscore
- **matchms** (Library in which MS2DeepScore is integrated for spectrum import, handling and comparisons, including the filtering (peak removal, 1000-peak cap) that precedes binning and augmentation)

## Examples

```
from ms2deepscore.data_augmentation import DataAugmentationMS2DeepScore; aug = DataAugmentationMS2DeepScore(); aug(binned_spectrum)
```

## Evaluation signals

- Output array shape and dtype match the input binned spectrum exactly (10,000 bins over 10–1000 m/z)
- Fraction of removed low-intensity bins (pre-transformation intensity < 0.4) stays within 0–0.2
- Non-zero intensity jitter factors all fall within 0.6–1.4
- Number of newly added peaks is 0–10 with intensities drawn between 0 and 0.01
- Different random draws per epoch/batch produce non-identical augmentations of the same training spectrum

## Limitations

- Augmentation parameters were tuned for this pipeline's square-root-transformed, 10,000-bin representation; other bin resolutions or intensity scales would need re-tuning
- Training data are inherently biased toward certain compound classes and instrumentation types, so augmentation cannot fully compensate for unbalanced or low-quality spectra
- The augmentation cannot recover fingerprint features that are simply not represented in any of the fragments, limiting how much Tanimoto prediction error can be reduced

## Evidence

- [discussion] Data augmentation step description: "we applied three forms of data augmentation on the binned spectra."
- [discussion] Binning specification: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] Pre-binning filtering and intensity transform: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [other] Purpose of augmentation (README-style summary; also stated in article): [evidence span withheld — anchor status: not_found]
- [discussion] Training regime context for augmentation cadence: "Models were trained with the Adam optimizer [44, 45] that optimized the mean squared error (MSE) loss. We used a batch size of 32, and a learning rate of 0.001."
