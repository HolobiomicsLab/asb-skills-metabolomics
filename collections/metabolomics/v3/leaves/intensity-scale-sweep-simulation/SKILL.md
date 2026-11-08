---
name: intensity-scale-sweep-simulation
description: Use when you need labeled synthetic MS windows with known ground-truth
  peaks at multiple intensity scales to benchmark signal/noise classification methods
  where real-data ground truth is unavailable.
when_to_use_negative:
- Evaluating on real timsTOF raw data — synthetic data cannot fully reflect real-data
  behavior, and no gold-standard peak-level annotation of real data exists; use vendor-selected
  DDA precursors instead
- When the downstream method needs peaks inside signal windows removed — the synthetic
  noise_2 label marks noise peaks within signal windows that are intentionally retained
  for subsequent processing steps
- When the analysis goal is real-data reduction rather than controlled classifier
  benchmarking
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Language of the createSynthetics module used to generate the synthetic labeled
    window datasets
- name: mzBucket
  role: Repository containing the createSynthetics module and the data/output_create_synthetics
    output folder for the synthetic data
  repo: https://github.com/hildebrandtlab/mzBucket
- name: pyproteolizard-data
  role: proteolizard framework component for data access on which the implementation
    and synthetic-data generation rest
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:95cab75b834ad1896d00c75cad52a5b5114078a19a1704592346d07d233a967e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - Python
  - mzBucket
  - pyproteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# intensity-scale-sweep-simulation

## Summary

Generates synthetic timsTOF-style mass-spectrometry window data with ground-truth signal/noise labels across six maximum-intensity scales (1000 down to 32), enabling ROC-based benchmarking of signal classifiers such as LSH versus intensity thresholding.

## When to use

Use when you need labeled synthetic MS windows with known ground-truth peaks at multiple intensity scales to benchmark signal/noise classification methods where real-data ground truth is unavailable.

## When NOT to use

- Evaluating on real timsTOF raw data — synthetic data cannot fully reflect real-data behavior, and no gold-standard peak-level annotation of real data exists; use vendor-selected DDA precursors instead
- When the downstream method needs peaks inside signal windows removed — the synthetic noise_2 label marks noise peaks within signal windows that are intentionally retained for subsequent processing steps
- When the analysis goal is real-data reduction rather than controlled classifier benchmarking

## Inputs

- Configuration for createSynthetics (window width 10 m/z, 0.1 m/z binning, Poisson(4)+1 noise counts, exponential noise intensities parameter 15, averagine range 150–5000 u, charges 1–5)
- Six maximum-intensity scale values: 1000, 500, 250, 125, 64, 32
- Optionally the Zenodo record 5036526 raw dataset unzipped into the data folder for accompanying reproducible runs

## Outputs

- Synthetic window datasets for all six intensity scales written to data/output_create_synthetics
- Per-peak ground-truth labels: signal, noise_1, noise_2
- Binned window peaks suitable for downstream LSH hashing and ROC analysis against intensity-thresholding baselines

## How to apply

Configure the createSynthetics module of the mzBucket repository (which has been shifted to the proteolizard framework, using pyproteolizard-data for data access). Cut the mass axis into windows of 10 m/z with 0.1 m/z equally spaced binning; sample noise peak counts from Poisson(4)+1 and noise intensities from an exponential distribution with parameter 15. Model isotope patterns with the averagine model restricted to monoisotopic masses of 150–5000 u and charge states 1–5, and assign every synthetic peak one of three ground-truth labels: signal, noise_1, or noise_2. Repeat the whole generation at each of six maximum-intensity scales — 1000, 500, 250, 125, 64, 32 — so downstream classifiers can be probed over a 30-fold intensity range, and write each dataset to data/output_create_synthetics. Downstream ROC comparisons should exclude noise_2 peaks (noise inside signal windows) from classification-rate computation, as the paper does for both LSH and intensity-thresholding methods.

## Related tools

- **Python** (Language of the createSynthetics module used to generate the synthetic labeled window datasets)
- **mzBucket** (Repository containing the createSynthetics module and the data/output_create_synthetics output folder for the synthetic data) — https://github.com/hildebrandtlab/mzBucket
- **pyproteolizard-data** (proteolizard framework component for data access on which the implementation and synthetic-data generation rest) — https://github.com/theGreatHerrLebert/proteolizard-data

## Examples

```
git clone https://github.com/hildebrandtlab/mzBucket && cd mzBucket && mkdir -p data/output_create_synthetics
```

## Evaluation signals

- Output files for all six intensity scales (1000, 500, 250, 125, 64, 32) exist under data/output_create_synthetics
- Generated peaks conform to the window/binning recipe: 10 m/z windows with 0.1 m/z equally spaced bins
- Noise peaks follow Poisson(4)+1 count distribution and exponential intensity distribution with parameter 15
- Isotope patterns fall within averagine monoisotopic range 150–5000 u with charge states 1–5
- Every generated peak carries exactly one of the three labels: signal, noise_1, or noise_2

## Limitations

- Synthetic data cannot fully reflect the behavior of real data
- Peaks labeled noise_2 (noise within signal windows) are not removed by the LSH approach and were excluded from classification-rate computation for both methods
- Only a qualitative acceptance check is possible at generation time — the paper reports no numeric threshold for generation fidelity

## Evidence

- [abstract] On synthetic data, a superior performance compared to an intensity thresholding approach was achieved: "On synthetic data, a superior performance compared to an intensity thresholding approach was achieved"
- [full_text] Sample noise peaks stochastically, drawing peak counts from a Poisson(4)+1 distribution and peak intensities from an exponential distribution with parameter 15.: [evidence span withheld — anchor status: not_found]
- [discussion] peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach.: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
- [readme] ├── data
│   ├── output_create_synthetics: "├── data
│   ├── output_create_synthetics"
- [readme] The implementation has been updated and shifted to the proteolizard framework, more precisely the components [pyproteolizard-data]: "The implementation has been updated and shifted to the proteolizard framework, more precisely the components [pyproteolizard-data]"
