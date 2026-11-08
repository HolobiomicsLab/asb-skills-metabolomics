---
name: ground-truth-peak-labeling
description: Use when building labeled synthetic benchmark data for signal-vs-noise
  classification on mass spectrometry raw data, especially when no gold-standard peak-level
  annotation of real timsTOF data is available.
when_to_use_negative:
- Input is real timsTOF/TDF raw data with no peak-level ground truth — the article
  notes no gold-standard peak-level annotation exists, and vendor-selected DDA peaks
  (N=6902) are a biased reference of likely high-intensity fragmented peptides.
- 'Goal is to model self-similar chemical noise: the paper notes chemical noise is
  self-similar and would be classified as ''true'' signal, which the synthetic signal/noise_1/noise_2
  labeling scheme does not capture.'
- Output is already a labeled feature/peak table — re-labeling synthetic peaks adds
  nothing.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Python
  role: Language in which the createSynthetics module implementing the labeled synthetic-window
    generation is written
  repo: https://github.com/hildebrandtlab/mzBucket
- name: mzBucket
  role: Repository providing the createSynthetics source tree and the data/output_create_synthetics
    folder for the labeled synthetic datasets
  repo: https://github.com/hildebrandtlab/mzBucket
- name: proteolizard-data
  role: Framework layer for timsTOF raw-data access and binned frame/spectrum structures
    on top of which synthetic windows are generated
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:53d00530ad17d5066f20125447c1e67ce0ddf7d3f0ffe4dfe7f8c37c2204ee60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - Python
  - mzBucket
  - proteolizard-data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ground-truth-peak-labeling

## Summary

Generates synthetic timsTOF-like MS windows with per-peak ground-truth labels (signal, noise_1, noise_2) so that peak-level classification methods, e.g. LSH vs. intensity thresholding, can be evaluated by ROC analysis without real-data gold standards.

## When to use

Use when building labeled synthetic benchmark data for signal-vs-noise classification on mass spectrometry raw data, especially when no gold-standard peak-level annotation of real timsTOF data is available.

## When NOT to use

- Input is real timsTOF/TDF raw data with no peak-level ground truth — the article notes no gold-standard peak-level annotation exists, and vendor-selected DDA peaks (N=6902) are a biased reference of likely high-intensity fragmented peptides.
- Goal is to model self-similar chemical noise: the paper notes chemical noise is self-similar and would be classified as 'true' signal, which the synthetic signal/noise_1/noise_2 labeling scheme does not capture.
- Output is already a labeled feature/peak table — re-labeling synthetic peaks adds nothing.

## Inputs

- createSynthetics module configuration (window width 10 m/z, bin size 0.1 m/z, Poisson(4)+1 noise counts, exponential(15) noise intensities)
- Averagine model parameters (monoisotopic mass range 150–5000 u, charge states 1–5)
- Six maximum-intensity scale values: 1000, 500, 250, 125, 64, 32
- Python environment with the proteolizard framework (pyproteolizard-data for data access)

## Outputs

- Synthetic peak datasets with per-peak ground-truth labels (signal, noise_1, noise_2), written one per intensity scale to data/output_create_synthetics
- Labeled windows suitable for ROC comparison of LSH hashing vs. intensity (SNR) thresholding

## How to apply

Configure the createSynthetics module (Python, proteolizard framework) to cut the mass axis into windows of 10 m/z with equally spaced 0.1 m/z binning. Sample noise peaks stochastically, drawing peak counts from a Poisson(4)+1 distribution and peak intensities from an exponential distribution with parameter 15. Model isotope patterns with the averagine model for monoisotopic masses in the range 150–5000 u and charge states 1–5, then assign each synthetic peak one of three ground-truth labels: signal, noise_1, or noise_2. Repeat the generation at each of six maximum-intensity scales (1000, 500, 250, 125, 64, 32) to probe the intensity regime, and write the datasets to data/output_create_synthetics. When later computing classification rates, exclude noise_2 peaks (noise inside signal windows) for both the LSH approach and intensity thresholding so the comparison is consistent.

## Related tools

- **Python** (Language in which the createSynthetics module implementing the labeled synthetic-window generation is written) — https://github.com/hildebrandtlab/mzBucket
- **mzBucket** (Repository providing the createSynthetics source tree and the data/output_create_synthetics folder for the labeled synthetic datasets) — https://github.com/hildebrandtlab/mzBucket
- **proteolizard-data** (Framework layer for timsTOF raw-data access and binned frame/spectrum structures on top of which synthetic windows are generated) — https://github.com/theGreatHerrLebert/proteolizard-data

## Evaluation signals

- Output files for all six intensity scales (1000, 500, 250, 125, 64, 32) exist under data/output_create_synthetics.
- Generated peaks conform to the specified window/binning recipe: 10 m/z windows with 0.1 m/z equally spaced bins.
- Noise peak counts follow a Poisson(4)+1 distribution and noise intensities follow an exponential distribution with parameter 15 (qualitative distribution check).
- Averagine-modeled isotope patterns fall in the 150–5000 u monoisotopic mass range with charge states 1–5.
- Every synthetic peak carries exactly one of the three ground-truth labels: signal, noise_1, or noise_2 (the paper reports no numeric fidelity threshold, so checks are qualitative).

## Limitations

- Synthetic data cannot fully reflect the behavior of real data, so benchmark conclusions must be corroborated on real timsTOF acquisitions.
- Noise peaks inside signal windows (label noise_2) are not removed by downstream LSH classification and require subsequent processing steps.
- No gold-standard ground-truth peak-level annotation of real data exists, so this labeling scheme is only directly verifiable on synthetic data.

## Evidence

- [methods] Sample noise peaks stochastically, drawing peak counts from a Poisson(4)+1 distribution and peak intensities from an exponential distribution with parameter 15.: [evidence span withheld — anchor status: not_found]
- [methods] Assign each synthetic peak one of the three ground-truth labels: signal, noise_1, or noise_2.: [evidence span withheld — anchor status: not_found]
- [discussion] peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach.: "peaks with label “noise_2” were not considered for the computation of classification rates both for the intensity thresholding and our approach."
- [readme] The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder to ensure reproducible runs of the scripts.: "The data set used is available for download [here](https://zenodo.org/record/5036526#.YlWE91xBw5l). Please unzip it into the data folder to ensure reproducible runs of the scripts."
