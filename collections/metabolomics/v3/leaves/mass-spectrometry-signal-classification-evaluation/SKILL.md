---
name: mass-spectrometry-signal-classification-evaluation
description: Use when labeled synthetic mass-spectrometry windows (signal vs. noise
  peaks) are available and you need ROC-based comparison of a classification method
  against intensity-threshold filtering to justify a denoising prefilter.
when_to_use_negative:
- 'Evaluating on real vendor raw data: the paper notes ''synthetic data cannot fully
  reflect the behavior of real data'' and no gold-standard peak-level annotation of
  real data was available'
- 'Quantifying removal of noise_2 peaks: noise peaks inside signal windows are not
  removed by either the LSH approach or intensity thresholding and must be excluded
  from classification rates'
- 'When the goal is measuring real-data reduction rather than discrimination quality:
  the paper''s real-data evaluation instead measures data-point reduction while retaining
  vendor-selected DDA peaks'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-algorithm
  role: Provides the LSH classifier (window hashing with n AND and m OR connectives)
    used to classify windows as signal or noise at settings (32,16), (64,32), (32,32)
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: pyproteolizard-data
  role: Data-access layer over opentims used to read timsTOF/TDF frames feeding the
    classification comparison
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: mzBucket
  role: Repository whose src/python/roc module contains the scripts that load synthetic
    windows and compute the ROC comparison
  repo: https://github.com/hildebrandtlab/mzBucket
- name: Python
  role: Language used for the roc scripts and plotting of combined ROC curves (matplotlib/scikit-learn-style
    rate curves)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:65fbe5c34d6a536d51b2fa3d2196c33837458c702f3215b0668cf5d3f752288f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-algorithm
  tools:
  - Python
  - mzBucket
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mass-spectrometry-signal-classification-evaluation

## Summary

Evaluates LSH-based signal classification of timsTOF raw data against an intensity (SNR) thresholding baseline by computing and plotting ROC curves on labeled synthetic windows across intensity panels, demonstrating superior discrimination of true signal versus noise.

## When to use

Use when labeled synthetic mass-spectrometry windows (signal vs. noise peaks) are available and you need ROC-based comparison of a classification method against intensity-threshold filtering to justify a denoising prefilter.

## When NOT to use

- Evaluating on real vendor raw data: the paper notes 'synthetic data cannot fully reflect the behavior of real data' and no gold-standard peak-level annotation of real data was available
- Quantifying removal of noise_2 peaks: noise peaks inside signal windows are not removed by either the LSH approach or intensity thresholding and must be excluded from classification rates
- When the goal is measuring real-data reduction rather than discrimination quality: the paper's real-data evaluation instead measures data-point reduction while retaining vendor-selected DDA peaks

## Inputs

- Labeled synthetic windows from createSynthetics (data/output_create_synthetics folder of Zenodo record 5036526, unzipped into the data folder)
- Peak labels distinguishing signal, noise_1, and noise_2 peaks
- LSH hash settings (m,n) pairs, e.g. (32,16), (64,32), (32,32)
- Intensity panel definitions (maximum signal intensity I = 500, 250, 125, 64, 32)

## Outputs

- Per-threshold true-positive and false-positive rate curves for the LSH approach and the intensity thresholding baseline
- Combined ROC figure saved as plots/roc_curve.png with one panel per intensity level

## How to apply

Load the labeled synthetic windows produced by createSynthetics from the data/output_create_synthetics folder of the unzipped Zenodo dataset (record 5036526) using the Python scripts in mzBucket's src/python/roc module. For each maximum signal-intensity panel (I = 500, 250, 125, 64, 32 as in Additional file 2), classify windows as signal or noise with the pyproteolizard-algorithm LSH approach across reported hash settings such as (m,n) = (32,16), (64,32), and (32,32), where n AND connectives and m OR connectives tune the false-positive and false-negative rates. For the same panels, compute baseline classification rates by sweeping the intensity/SNR threshold. Exclude peaks labeled noise_2 from classification-rate computation for both methods, since noise peaks inside signal windows are not removed by either approach. Compute per-threshold true-positive and false-positive rates, build ROC curves for both methods per panel, plot them together (e.g. with matplotlib), and save the combined figure as plots/roc_curve.png.

## Related tools

- **pyproteolizard-algorithm** (Provides the LSH classifier (window hashing with n AND and m OR connectives) used to classify windows as signal or noise at settings (32,16), (64,32), (32,32)) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **pyproteolizard-data** (Data-access layer over opentims used to read timsTOF/TDF frames feeding the classification comparison) — https://github.com/theGreatHerrLebert/proteolizard-data
- **mzBucket** (Repository whose src/python/roc module contains the scripts that load synthetic windows and compute the ROC comparison) — https://github.com/hildebrandtlab/mzBucket
- **Python** (Language used for the roc scripts and plotting of combined ROC curves (matplotlib/scikit-learn-style rate curves))

## Examples

```
cd mzBucket && python src/python/roc/roc.py  # loads data/output_create_synthetics, sweeps LSH (m,n) and intensity thresholds, writes plots/roc_curve.png
```

## Evaluation signals

- plots/roc_curve.png exists in the plots folder after running the src/python/roc scripts
- The LSH ROC curve dominates the intensity-thresholding curve across the intensity panels (I = 500, 250, 125, 64, 32), matching the paper's reported finding
- The (m,n)=(64,32) setting shows significantly improved classification performance relative to the low threshold (32,16), which yields very high true-positive AND false-positive rates
- The very high threshold (m,n)=(32,32) lowers the true-positive rate but also shrinks the false-positive rate, as reported
- noise_2 peaks are excluded from classification-rate computation for both the LSH approach and the intensity thresholding baseline

## Limitations

- Synthetic data cannot fully reflect the behavior of real data
- The LSH approach misclassifies self-similar chemical noise (repeated contaminants) as true signal
- Signals with low relative intensity to noise peaks could be lost because the underlying similarity measure drops
- Vendor-selected DDA peaks used as a real-data reference are biased: only fragmented peptides, likely high-intensity signals

## Evidence

- [results] The performance of our approach was compared to the common approach of filtering signal by an intensity threshold.: "The performance of our approach was compared to the common approach of filtering signal by an intensity threshold. While the overall behavior of its ROC curve is similar to the one of our approach,"
- [results] By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates.: "By using an augmented LSH with n AND connectives and m OR connectives, the algorithm allows for tuning of false-positive and false-negative rates."
- [discussion] peaks with label 'noise_2' were not considered for the computation of classification rates both for the intensity thresholding and our approach.: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
- [readme] The data set used is available for download here (Zenodo record 5036526). Please unzip it into the data folder: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder to ensure reproducible runs of the scripts."
- [abstract] On synthetic data, a superior performance compared to an intensity thresholding approach was achieved.: "On synthetic data, a superior performance compared to an intensity thresholding approach was achieved"
