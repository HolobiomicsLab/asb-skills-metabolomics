---
name: poisson-noise-peak-sampling
description: Use when you need a labeled synthetic mass-spectrometry benchmark with
  controlled noise density (e.g. to compute ROC curves for signal-vs-noise classifiers
  like LSH vs. intensity thresholding) and no real-data peak-level ground truth exists.
when_to_use_negative:
- The goal is to evaluate classification on real timsTOF raw data — the article notes
  synthetic data cannot fully reflect real-data behavior and no gold-standard peak-level
  annotation of real data was available.
- 'The downstream task requires removing peaks inside signal windows: the synthetic
  noise_2 class (noise within signal windows) is explicitly not removed by the LSH
  approach and was excluded from classification-rate computation.'
- You need to model chemical/contaminant noise that is self-similar across windows
  — such noise would be misclassified as true signal by the LSH method and is not
  captured by this stochastic noise model.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: mzBucket
  role: Repository containing the createSynthetics module that implements the synthetic
    window benchmark generation (noise sampling and signal labeling)
  repo: https://github.com/hildebrandtlab/mzBucket
- name: proteolizard-data
  role: Python/C++ library (built on OpenTIMS via pybind11) providing data access,
    binning and window/spectrum structures on which the synthetic generation runs
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: proteolizard-algorithm
  role: Provides the LSH algorithm that the generated synthetic noise/signal datasets
    are used to evaluate
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
- name: Python
  role: Language of the createSynthetics scripts and the pyproteolizard interface
  repo: https://github.com/hildebrandtlab/mzBucket
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:e4966a2812ef9f49ba6c9c4dc91e532743de73ced9eb3039740d9da47fa90c77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - Python
  - mzBucket
  - proteolizard-algorithm
  - proteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Poisson noise peak sampling

## Summary

Stochastic generation of synthetic background-noise peaks for a timsTOF-style benchmark: per-window peak counts are drawn from a Poisson(4)+1 distribution and intensities from an exponential distribution with parameter 15, then labeled noise_1/noise_2 alongside averagine-modeled signal peaks so LSH classification can be evaluated against ground truth.

## When to use

Use when you need a labeled synthetic mass-spectrometry benchmark with controlled noise density (e.g. to compute ROC curves for signal-vs-noise classifiers like LSH vs. intensity thresholding) and no real-data peak-level ground truth exists.

## When NOT to use

- The goal is to evaluate classification on real timsTOF raw data — the article notes synthetic data cannot fully reflect real-data behavior and no gold-standard peak-level annotation of real data was available.
- The downstream task requires removing peaks inside signal windows: the synthetic noise_2 class (noise within signal windows) is explicitly not removed by the LSH approach and was excluded from classification-rate computation.
- You need to model chemical/contaminant noise that is self-similar across windows — such noise would be misclassified as true signal by the LSH method and is not captured by this stochastic noise model.

## Inputs

- Configuration for the createSynthetics module (window width 10 m/z, bin width 0.1 m/z, noise count Poisson(4)+1, noise intensity exponential(15), averagine range 150–5000 u, charge states 1–5)
- Six target maximum-intensity scales: 1000, 500, 250, 125, 64, 32
- Optionally a real timsTOF .d dataset (Bruker TDF, via PyTimsDataHandle) as a template for frame/scan structure

## Outputs

- Synthetic peak datasets for all six intensity scales, written to data/output_create_synthetics
- Per-peak ground-truth labels: signal, noise_1, or noise_2
- Windows in binned 0.1 m/z form suitable for hashing downstream

## How to apply

Configure the createSynthetics module of the proteolizard framework (accessed via pyproteolizard-data) to cut the mass axis into windows of 10 m/z with equally spaced 0.1 m/z bins. Stochastically sample noise peaks: draw the number of peaks per window from a Poisson(4)+1 distribution and each peak's intensity from an exponential distribution with parameter 15. Model the true-signal population with the averagine model for monoisotopic masses in 150–5000 u and charge states 1–5, and assign every synthetic peak one of three ground-truth labels: signal, noise_1, or noise_2. Repeat the whole generation at each of six maximum-intensity scales (1000, 500, 250, 125, 64, 32) so classifier performance can be probed across signal-to-noise regimes, writing each dataset to data/output_create_synthetics. The rationale is that the paper's LSH approach was validated on synthetic data against the common intensity-threshold (SNR) filter, which requires exactly this controlled noise model.

## Related tools

- **mzBucket** (Repository containing the createSynthetics module that implements the synthetic window benchmark generation (noise sampling and signal labeling)) — https://github.com/hildebrandtlab/mzBucket
- **proteolizard-data** (Python/C++ library (built on OpenTIMS via pybind11) providing data access, binning and window/spectrum structures on which the synthetic generation runs) — https://github.com/theGreatHerrLebert/proteolizard-data
- **proteolizard-algorithm** (Provides the LSH algorithm that the generated synthetic noise/signal datasets are used to evaluate) — https://github.com/theGreatHerrLebert/proteolizard-algorithm
- **Python** (Language of the createSynthetics scripts and the pyproteolizard interface) — https://github.com/hildebrandtlab/mzBucket

## Evaluation signals

- Output files for all six intensity scales (1000, 500, 250, 125, 64, 32) exist under data/output_create_synthetics.
- Noise peak counts per window follow the specified Poisson(4)+1 distribution; intensities follow an exponential distribution with parameter 15.
- Windows conform to the 10 m/z width with 0.1 m/z equally spaced bins before any hash computation.
- Every synthetic peak carries exactly one of the three labels signal, noise_1, or noise_2; averagine-modeled peaks fall in the 150–5000 u monoisotopic mass range with charge states 1–5.
- Downstream sanity check: running the LSH classifier (e.g. (m,n)=(64,32)) and intensity thresholding on the synthetic sets should reproduce the paper's qualitative result of superior LSH ROC performance.

## Limitations

- Synthetic data cannot fully reflect the behavior of real data, so benchmark results do not transfer directly to real timsTOF measurements.
- Chemical noise (repeated contaminants) is self-similar and would be misclassified as true signal by the LSH approach; this noise class is not represented in the stochastic sampling model.
- Signals with low relative intensity to noise peaks could be lost because the similarity measure underlying the LSH drops — at low intensity scales (e.g. 32) the benchmark probes this regime.
- The paper reports no numeric threshold for generation fidelity, so validation of the synthetic output is a qualitative recipe check.

## Evidence

- [methods] Stochastic generation of synthetic background-noise peaks for a timsTOF-style benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Averagine signal modeling with ground-truth labels: [evidence span withheld — anchor status: not_found]
- [abstract] Purpose of the synthetic benchmark: "On synthetic data, a superior performance compared to an intensity thresholding approach was achieved"
- [discussion] noise_2 class excluded from classification rates: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
- [readme] Where generated outputs live and how reproducibility is ensured: "Please unzip it into the data folder to ensure reproducible runs of the scripts."
