---
name: intensity-threshold-baseline-comparison
description: Use when a binary signal/noise classifier (e.g. LSH on mass-spectrometry
  windows) needs a ROC-based comparison against intensity/SNR threshold filtering
  on labeled synthetic windows with per-panel intensity settings.
when_to_use_negative:
- Evaluation is on real raw data without gold-standard peak-level ground truth — the
  paper notes no such annotation was available, so ROC comparison is only well-posed
  on labeled synthetic windows.
- The goal is filtering real data for size reduction (e.g. with the strong (32,32)
  filter), not measuring classifier discrimination.
- Peaks of interest include chemical noise contaminants, which the paper warns are
  self-similar and would be misclassified as true signal by this approach.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyproteolizard-algorithm
  role: Provides the LSH classifier applied across (m,n) hash settings for the treatment
    arm of the ROC comparison
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: pyproteolizard-data
  role: Data-access layer for timsTOF raw data underlying the synthetic windows
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: Python
  role: Language of the src/python/roc scripts used to compute and plot the ROC curves
- name: mzBucket
  role: Repository containing the src/python/roc module, createSynthetics outputs,
    and the plots/ output folder
  repo: https://github.com/hildebrandtlab/mzBucket
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:ebbc9c68735b4ce2f9e85634ac854af36ad3474f1e1729d880d40ba628944c32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theGreatHerrLebert/proteolizard-algorithm
  tools:
  - Python
  - mzBucket
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# intensity-threshold-baseline-comparison

## Summary

Benchmark an LSH-based signal classifier against the common intensity (SNR) thresholding baseline by sweeping thresholds and comparing ROC curves per maximum-signal-intensity panel (I = 500, 250, 125, 64, 32) on labeled synthetic timsTOF windows.

## When to use

Use when a binary signal/noise classifier (e.g. LSH on mass-spectrometry windows) needs a ROC-based comparison against intensity/SNR threshold filtering on labeled synthetic windows with per-panel intensity settings.

## When NOT to use

- Evaluation is on real raw data without gold-standard peak-level ground truth — the paper notes no such annotation was available, so ROC comparison is only well-posed on labeled synthetic windows.
- The goal is filtering real data for size reduction (e.g. with the strong (32,32) filter), not measuring classifier discrimination.
- Peaks of interest include chemical noise contaminants, which the paper warns are self-similar and would be misclassified as true signal by this approach.

## Inputs

- Labeled synthetic windows from createSynthetics (data/output_create_synthetics of Zenodo record 5036526)
- LSH hash settings (m,n) such as (32,16), (64,32), (32,32)
- Peak labels including signal, noise, and noise_2 categories
- Intensity/SNR threshold sweep values

## Outputs

- Per-panel ROC curves for the LSH classifier (varying (m,n))
- Per-panel ROC curves for the intensity/SNR thresholding baseline
- Combined ROC figure saved as plots/roc_curve.png

## How to apply

Load the labeled synthetic windows produced by createSynthetics from data/output_create_synthetics of the unzipped Zenodo record 5036526, using the Python scripts in src/python/roc of the mzBucket repository. For each maximum signal-intensity panel (I = 500, 250, 125, 64, 32, as in Additional file 2), classify windows as signal or noise with the LSH approach from pyproteolizard-algorithm across reported hash settings (m,n) = (32,16), (64,32), (32,32), where m are OR connectives and n are AND connectives controlling false-positive and false-negative rates. For the same panels, sweep the intensity threshold to compute classification rates for the SNR-thresholding baseline. Exclude peaks labeled noise_2 from the rate computation for both methods, since noise peaks inside signal windows are not removed by either approach. Compute true-positive and false-positive rates per threshold, plot the LSH and intensity-thresholding ROC curves together per panel, and save the combined figure as plots/roc_curve.png.

## Related tools

- **pyproteolizard-algorithm** (Provides the LSH classifier applied across (m,n) hash settings for the treatment arm of the ROC comparison) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **pyproteolizard-data** (Data-access layer for timsTOF raw data underlying the synthetic windows) — https://github.com/theGreatHerrLebert/proteolizard-data
- **Python** (Language of the src/python/roc scripts used to compute and plot the ROC curves)
- **mzBucket** (Repository containing the src/python/roc module, createSynthetics outputs, and the plots/ output folder) — https://github.com/hildebrandtlab/mzBucket

## Examples

```
python src/python/roc/plot_roc.py --input data/output_create_synthetics --output plots/roc_curve.png  # per mzBucket: scripts in src/python/roc, outputs to plots/roc_curve.png
```

## Evaluation signals

- plots/roc_curve.png exists in the plots folder of the mzBucket workflow
- The LSH ROC curve dominates the intensity-thresholding curve across the panels, matching the reported finding of superior LSH performance on synthetic data
- All five intensity panels (I = 500, 250, 125, 64, 32) are represented, matching Additional file 2
- Peaks labeled noise_2 are excluded from classification-rate computation for both methods
- The (64,32) LSH setting shows significantly improved discrimination relative to (32,16), and (32,32) trades lower true-positive rate for a lower false-positive rate

## Limitations

- Synthetic data cannot fully reflect the behavior of real data, so the ROC advantage may not transfer to real acquisitions.
- Noise peaks within signal windows (label noise_2) are not removed by either method and require subsequent processing steps.
- Only the overall ROC shape is similar between methods; the LSH advantage is method-level, not a guarantee for any single (m,n) setting — (32,16) yields very high false-positive rates.
- Vendor-selected peaks used as reference on real data are biased toward high-intensity, fragmented peptides.

## Evidence

- [results] The performance of our approach was compared to the common approach of filtering signal by an intensity threshold. While the overall behavior of its ROC curve is similar to the one of our approach,: "The performance of our approach was compared to the common approach of filtering signal by an intensity threshold. While the overall behavior of its ROC curve is similar to the one of our approach,"
- [discussion] peaks with label noise_2 were not considered for the computation of classification rates both for the intensity thresholding and our approach.: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
- [abstract] Through appropriate choice of algorithm parameters it is possible to balance false-positive and false-negative rates: "Through appropriate choice of algorithm parameters it is possible to balance false-positive and false-negative rates"
- [readme] The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder to ensure reproducible runs of the scripts."
- [results] When setting a low threshold, e.g., (m,n) = (32,16) ... a large fraction of windows are considered signal, resulting in very high true-positive and false-positive rates.: [evidence span withheld — anchor status: not_found]
