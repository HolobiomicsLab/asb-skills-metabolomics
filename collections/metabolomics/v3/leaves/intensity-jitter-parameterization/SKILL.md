---
name: intensity-jitter-parameterization
description: Use when preparing a binned training corpus for MS2DeepScore (or a similar
  Siamese spectral-embedding model) and you need stochastic intensity variability
  applied to the square-root-transformed, 10,000-bin spectra between 10 and 1000 m/z.
when_to_use_negative:
- When processing validation or test spectra — augmentation is applied to binned training
  spectra only, and applying jitter to the 3601-spectrum test set would bias the Tanimoto
  RMSE estimate of ~0.15
- When spectra are still in raw peak-list format — jitter operates on the binned,
  square-root-transformed representation, so it must not be applied before the binning
  and sqrt-transformation preprocessing steps
- When you need biologically meaningful intensity calibration rather than robustness
  regularization — jitter is a bounded random perturbation (±40%), not a normalization
  of peak intensities
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MS2DeepScore
  role: The Siamese neural network whose training data pipeline consumes the jittered
    binned spectra; jitter exists to increase the model's robustness to moderate random
    changes of peak intensities
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Used for spectrum import, cleaning and handling upstream of binning, producing
    the spectra that then get binned and augmented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:aebbd4d612ef57d911463bfa4458725b9963908f0d661219f04e8ee80864bd3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# intensity-jitter-parameterization

## Summary

Parameterizes the intensity-jitter component of MS2DeepScore's training-time data augmentation, which multiplies each non-zero bin of a binned, square-root-transformed spectrum by a random factor in [0.6, 1.4] to make the Siamese network robust to moderate random changes of peak intensities.

## When to use

Use when preparing a binned training corpus for MS2DeepScore (or a similar Siamese spectral-embedding model) and you need stochastic intensity variability applied to the square-root-transformed, 10,000-bin spectra between 10 and 1000 m/z.

## When NOT to use

- When processing validation or test spectra — augmentation is applied to binned training spectra only, and applying jitter to the 3601-spectrum test set would bias the Tanimoto RMSE estimate of ~0.15
- When spectra are still in raw peak-list format — jitter operates on the binned, square-root-transformed representation, so it must not be applied before the binning and sqrt-transformation preprocessing steps
- When you need biologically meaningful intensity calibration rather than robustness regularization — jitter is a bounded random perturbation (±40%), not a normalization of peak intensities

## Inputs

- Binned mass spectra: numpy arrays of 10,000 equal-width bins spanning 10–1000 m/z with square-root-transformed intensities
- Original (pre-binning) spectra with intensities prior to square-root transformation, to identify bins whose pre-transformation intensity is below 0.4 for the companion removal step
- Augmentation configuration values: jitter range (e.g. 0.6–1.4), removal fraction bound (≤ 0.2), added-peak count bound (≤ 10), added-peak intensity bound (≤ 0.01)

## Outputs

- Intensity-jittered binned spectra of identical shape and dtype to the input, with non-zero bins multiplied by a factor in [0.6, 1.4]
- An augmented training corpus suitable for feeding to the Siamese network's training loop (Adam optimizer, MSE loss, batch size 32, learning rate 0.001)

## How to apply

First bin the cleaned spectra into 10,000 equally-sized bins ranging from 10 to 1000 m/z and transform intensities with a square root, since jitter acts on binned spectra, not raw peak lists. For each training example, independently draw a multiplicative factor per epoch/batch and multiply the non-zero bins by that random factor, keeping the jitter bounded at up to ±40% (factor within 0.6–1.4) relative to the original transformed intensity. Apply intensity jitter as one of the three augmentations in sequence with the other two — random removal of low-intensity peaks (those with pre-transformation intensity below 0.4) and addition of up to 10 random peaks with intensities drawn between 0 and 0.01 — so the model learns robustness to moderate random changes of peak intensities plus removal and addition of low intensity peaks. Expose each augmentation parameter (jitter range, removal fraction ≤ 0.2, added-peak count ≤ 10 and intensity ≤ 0.01) as configurable, bounded values rather than hard-coded constants. Validate that the augmented spectrum preserves the original shape and dtype so downstream pair sampling and training with the Adam optimizer (batch size 32, learning rate 0.001) is unaffected. The rationale is that randomizing intensities during training improves generalization on the held-out test set (3601 spectra of 500 unseen compounds) without changing the m/z bin structure.

## Related tools

- **MS2DeepScore** (The Siamese neural network whose training data pipeline consumes the jittered binned spectra; jitter exists to increase the model's robustness to moderate random changes of peak intensities) — https://github.com/matchms/ms2deepscore
- **matchms** (Used for spectrum import, cleaning and handling upstream of binning, producing the spectra that then get binned and augmented)

## Examples

```
jittered = binned_spectrum.copy(); nonzero = jittered > 0; jittered[nonzero] *= np.random.uniform(0.6, 1.4)
```

## Evaluation signals

- Every non-zero bin of the output spectrum equals the input bin multiplied by a factor within [0.6, 1.4]; zero bins remain exactly zero
- The augmented array has the same shape (10,000,) and dtype as the input binned spectrum — no padding, truncation, or dtype coercion
- The jitter bound is verified as configurable and capped at ±40% (0.6–1.4); a test with a forced factor outside this range should fail
- Augmentation leaves the 10–1000 m/z bin structure untouched — the m/z axis (bin index to mass mapping) is invariant between input and output
- Downstream training proceeds without error: the augmented corpus feeds the Adam-optimizer training loop (batch size 32, MSE loss) and the model still reaches test-set RMSE in the reported 0.13–0.2 band on held-out Tanimoto predictions

## Limitations

- Jitter is calibrated for square-root-transformed intensities of binned spectra; applying it to raw intensities or to a different binning scheme (e.g. different number of bins or m/z range) is out of scope
- The augmentation is applied only to training spectra — the article's benchmark of RMSE ~0.15 on 3600 spectra of 500 unseen compounds assumes the test set is unaugmented
- The article notes spectra come from different instrumentation types, are of varying quality, and are very unbalanced with regard to represented compound classes, so a jitter range tuned on this GNPS-derived corpus may transfer imperfectly to other libraries
- Intensity jitter is one of three coupled augmentations; applying it in isolation (without low-intensity removal and random peak addition) changes the effective augmentation distribution described in the paper

## Evidence

- [discussion] Data augmentation is applied to the binned training spectra to increase model robustness: "we applied three forms of data augmentation on the binned spectra."
- [discussion] Jitter operates on the binned, transformed representation: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] Moderate random changes of peak intensities plus removal and addition of low intensity peaks: [evidence span withheld — anchor status: not_found]
- [discussion] Model training hyperparameters that the augmented corpus feeds into: [evidence span withheld — anchor status: not_found]
- [discussion] Preprocessing steps that must precede jitter: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [abstract] Test-set performance the augmentation is meant to support: "MS2DeepScore is able to identify highly-reliable structural matches and to predict Tanimoto scores for pairs of molecules based on their fragment spectra"
