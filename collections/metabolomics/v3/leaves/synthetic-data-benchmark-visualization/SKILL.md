---
name: synthetic-data-benchmark-visualization
description: Use when labeled synthetic data with signal/noise peak classes (e.g.
  createSynthetics output from Zenodo record 5036526) must be compared to a baseline
  filter such as an intensity (SNR) threshold, and ROC curves are the required evidence
  per intensity panel.
when_to_use_negative:
- When the goal is peak-level ground-truth evaluation on real timsTOF data — the paper
  notes no gold-standard peak-level annotation of real data was available and vendor-selected
  DDA peaks are a biased, high-intensity reference.
- When the benchmark target is chemical (self-similar, repeated contaminant) noise,
  which the LSH approach would classify as true signal.
- When the task is assessing noise_2 peak removal — noise peaks inside signal windows
  are excluded from classification-rate computation because neither method removes
  them.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pyproteolizard-algorithm
  role: Provides the LSH-based signal/noise classification of windows at tunable (m,n)
    hash settings for the ROC sweep
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: pyproteolizard-data
  role: Data-access layer for timsTOF raw-data frames/spectra feeding the classification
    workflow
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: mzBucket
  role: Repository hosting the src/python/roc scripts and the plots folder used to
    compute and save the ROC figure
  repo: https://github.com/hildebrandtlab/mzBucket
- name: Python
  role: Implementation language for the ROC computation and matplotlib-style plotting
    of LSH vs. intensity-thresholding curves
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:9b8e1007d6a4586abee9e70940268d1a0a3e982427b04f314e00a9526a31c0af
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# synthetic-data-benchmark-visualization

## Summary

Benchmark a classification method (LSH signal classification) against an intensity/SNR-thresholding baseline on labeled synthetic timsTOF windows by sweeping thresholds, computing per-threshold true/false-positive rates, and visualizing the paired ROC curves across intensity panels. It grounds method-vs-baseline claims in a reproducible figure (plots/roc_curve.png) built from Zenodo-hosted synthetic data.

## When to use

Use when labeled synthetic data with signal/noise peak classes (e.g. createSynthetics output from Zenodo record 5036526) must be compared to a baseline filter such as an intensity (SNR) threshold, and ROC curves are the required evidence per intensity panel.

## When NOT to use

- When the goal is peak-level ground-truth evaluation on real timsTOF data — the paper notes no gold-standard peak-level annotation of real data was available and vendor-selected DDA peaks are a biased, high-intensity reference.
- When the benchmark target is chemical (self-similar, repeated contaminant) noise, which the LSH approach would classify as true signal.
- When the task is assessing noise_2 peak removal — noise peaks inside signal windows are excluded from classification-rate computation because neither method removes them.

## Inputs

- Labeled synthetic windows produced by createSynthetics, located in data/output_create_synthetics after unzipping the Zenodo record 5036526 dataset
- LSH hash parameter settings (m,n) pairs: (32,16), (64,32), (32,32)
- Maximum signal-intensity panel values I = 500, 250, 125, 64, 32 (Additional file 2)
- Peak labels distinguishing signal, noise, and noise_2 (noise inside signal windows)

## Outputs

- Per-threshold true-positive and false-positive rates for the LSH approach and the intensity/SNR-thresholding baseline
- Combined ROC curves per intensity panel, comparing LSH vs. intensity thresholding
- plots/roc_curve.png in the mzBucket plots folder

## How to apply

Load the labeled synthetic windows produced by createSynthetics from data/output_create_synthetics of the unzipped Zenodo 5036526 dataset using the scripts in the mzBucket repository's src/python/roc module. For each maximum signal-intensity panel (I = 500, 250, 125, 64, 32 as in Additional file 2), classify windows as signal or noise with the LSH approach from pyproteolizard-algorithm across reported hash settings such as (m,n) = (32,16), (64,32), and (32,32), where m OR-connectives and n AND-connectives trade true-positive against false-positive rates. For the same panels, compute classification rates for the intensity/SNR-thresholding baseline by sweeping the intensity threshold. Exclude peaks labeled noise_2 from classification-rate computation for both methods, since noise peaks inside signal windows are not removed by either approach; then compute ROC curves from per-threshold true-positive and false-positive rates and plot LSH vs. intensity-thresholding curves together per panel, saving the combined figure as plots/roc_curve.png. The expected outcome is that the LSH curve dominates the intensity-thresholding curve across panels, matching the paper's reported superior performance on synthetic data.

## Related tools

- **pyproteolizard-algorithm** (Provides the LSH-based signal/noise classification of windows at tunable (m,n) hash settings for the ROC sweep) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **pyproteolizard-data** (Data-access layer for timsTOF raw-data frames/spectra feeding the classification workflow) — https://github.com/theGreatHerrLebert/proteolizard-data
- **mzBucket** (Repository hosting the src/python/roc scripts and the plots folder used to compute and save the ROC figure) — https://github.com/hildebrandtlab/mzBucket
- **Python** (Implementation language for the ROC computation and matplotlib-style plotting of LSH vs. intensity-thresholding curves)

## Examples

```
python src/python/roc/roc.py  # loads data/output_create_synthetics windows, sweeps LSH (m,n) and intensity thresholds, writes plots/roc_curve.png
```

## Evaluation signals

- plots/roc_curve.png exists in the mzBucket plots folder after running the src/python/roc scripts.
- The LSH ROC curve dominates the intensity-thresholding ROC curve across the intensity panels, consistent with the paper's reported finding.
- Peaks labeled noise_2 are excluded from classification-rate computation for both methods.
- Low-threshold setting (m,n)=(32,16) yields very high true-positive and false-positive rates; stricter (m,n)=(64,32) improves performance significantly; very high (m,n)=(32,32) lowers the true-positive rate while also shrinking the false-positive rate.
- ROC panels correspond to the reported maximum signal-intensity values I = 500, 250, 125, 64, 32.

## Limitations

- Synthetic data cannot fully reflect the behavior of real data.
- Chemical noise (repeated contaminants) is self-similar and would be misclassified as true signal by the LSH approach.
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops.
- Noise peaks within signal windows (noise_2) are not removed by either method and require subsequent processing steps.

## Evidence

- [abstract] Superior performance compared to an intensity thresholding approach on synthetic data: "On synthetic data, a superior performance compared to an intensity thresholding approach was achieved"
- [results] ROC comparison result: "The performance of our approach was compared to the common approach of filtering signal by an intensity threshold."
- [results] m,n parameter behavior: [evidence span withheld — anchor status: not_found]
- [discussion] noise_2 exclusion: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
- [readme] Repo data layout: "├── data
│   ├── output_create_synthetics
│   └── output_scalability_study"
