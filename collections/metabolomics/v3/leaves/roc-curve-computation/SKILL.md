---
name: roc-curve-computation
description: Use when two binary signal/noise classifiers on labeled synthetic mass-spectrometry
  windows (e.g., LSH vs. intensity thresholding) must be compared by receiver operating
  characteristic performance.
when_to_use_negative:
- Input is real (non-synthetic) timsTOF data without gold-standard peak-level ground-truth
  labels — the article notes no such annotation was available, so ROC evaluation is
  not well-defined there.
- The task is data reduction (e.g., applying the strong (32,32) filter to shrink large
  datasets) rather than classifier performance comparison.
- Peak labels are absent, so noise_2 peaks cannot be excluded and classification rates
  would be biased.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-algorithm
  role: provides the LSH classification (hash settings such as (32,16), (64,32), (32,32))
    applied to the synthetic windows
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: pyproteolizard-data
  role: data-access layer for timsTOF raw data used to construct and classify windows
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: Python
  role: runs the src/python/roc scripts and computes/plots the ROC curves (e.g., matplotlib/scikit-learn-style
    rate curves)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:c8fa11367ee562caf165622a3f500c8349686c042fb566fc0d0117a47d5ed7f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-algorithm
  tools:
  - Python
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# roc-curve-computation

## Summary

Computes and plots ROC curves comparing the LSH-based signal classifier against the intensity (SNR) thresholding baseline on labeled synthetic timsTOF windows, with per-threshold true-positive and false-positive rates excluding noise_2 peaks.

## When to use

Use when two binary signal/noise classifiers on labeled synthetic mass-spectrometry windows (e.g., LSH vs. intensity thresholding) must be compared by receiver operating characteristic performance.

## When NOT to use

- Input is real (non-synthetic) timsTOF data without gold-standard peak-level ground-truth labels — the article notes no such annotation was available, so ROC evaluation is not well-defined there.
- The task is data reduction (e.g., applying the strong (32,32) filter to shrink large datasets) rather than classifier performance comparison.
- Peak labels are absent, so noise_2 peaks cannot be excluded and classification rates would be biased.

## Inputs

- labeled synthetic windows from data/output_create_synthetics (unzipped Zenodo record 5036526 dataset)
- LSH hash settings (m,n) = (32,16), (64,32), (32,32)
- maximum signal-intensity panels I = 500, 250, 125, 64, 32
- peak labels distinguishing noise_2 peaks from other peaks

## Outputs

- per-threshold true-positive and false-positive rates for LSH and intensity thresholding
- combined ROC figure saved as plots/roc_curve.png with one panel per intensity level

## How to apply

Load the labeled synthetic windows produced by createSynthetics from data/output_create_synthetics of the unzipped Zenodo record 5036526 using the Python scripts in the repository's src/python/roc module. Classify windows with pyproteolizard-algorithm's LSH across the reported hash settings (m,n) = (32,16), (64,32), (32,32), and compute the intensity-thresholding baseline by sweeping the intensity threshold for each maximum signal-intensity panel (I = 500, 250, 125, 64, 32, as in Additional file 2). Exclude peaks labeled noise_2 from the classification-rate computation for both methods, since noise peaks inside signal windows are not removed by either approach. From the per-threshold true-positive and false-positive rates, compute ROC curves for both methods and plot them together per intensity panel with matplotlib/scikit-learn-style rate curves, saving the combined figure as plots/roc_curve.png.

## Related tools

- **pyproteolizard-algorithm** (provides the LSH classification (hash settings such as (32,16), (64,32), (32,32)) applied to the synthetic windows) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **pyproteolizard-data** (data-access layer for timsTOF raw data used to construct and classify windows) — https://github.com/theGreatHerrLebert/proteolizard-data
- **Python** (runs the src/python/roc scripts and computes/plots the ROC curves (e.g., matplotlib/scikit-learn-style rate curves))

## Examples

```
python src/python/roc/roc_curve.py  # loads data/output_create_synthetics, classifies LSH vs intensity threshold, writes plots/roc_curve.png
```

## Evaluation signals

- plots/roc_curve.png exists in the plots folder after the run
- the LSH ROC curve dominates the intensity-thresholding ROC curve across the I = 500, 250, 125, 64, 32 panels, consistent with the reported finding of considerably better LSH performance on synthetic data
- curve shapes match Additional file 2's panel layout (one ROC per maximum signal-intensity panel)
- noise_2 peaks are excluded from both methods' classification-rate computations, so the two methods' rates are computed on the same peak set
- varying (m,n) reproduces the expected behavior: low threshold (32,16) gives very high true- and false-positive rates, (64,32) improves discrimination, and the very high threshold (32,32) lowers both rates

## Limitations

- Noise peaks labeled noise_2 (inside signal windows) are not removed by either method and must be excluded from the rate computation.
- Synthetic data cannot fully reflect the behavior of real data, so ROC superiority on synthetics does not transfer automatically.
- The choice of (m,n) trades off false-positive and false-negative rates: (32,16) yields high FPR, (32,32) loses some windows and reduces TPR.

## Evidence

- [results] While the LSH approach and the intensity thresholding filter produced ROC curves of similar overall shape, the LSH approach performed considerably better on the synthetic classification task.: [evidence span withheld — anchor status: not_found]
- [discussion] peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach.: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
- [results] When setting a low threshold, e.g., $$(m,n) = (32,16)$$: [evidence span withheld — anchor status: not_found]
- [readme] Please unzip it into the data folder to ensure reproducible runs of the scripts.: "Please unzip it into the data folder to ensure reproducible runs of the scripts."
