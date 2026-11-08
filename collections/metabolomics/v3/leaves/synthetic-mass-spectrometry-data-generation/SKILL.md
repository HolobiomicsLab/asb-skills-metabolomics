---
name: synthetic-mass-spectrometry-data-generation
description: Use when you need labeled synthetic mass spectrometry peak data with
  known signal/noise ground truth (e.g., to compute ROC curves for a classification
  method) because real peak-level annotations are unavailable.
when_to_use_negative:
- The input is real timsTOF raw data in Bruker .d format — access it via proteolizard-data/OpenTIMS
  instead of generating synthetic windows
- You need to model chemical/repeated-contaminant noise, which is self-similar and
  not captured by the Poisson noise model used here
- You need a validated real-data gold standard for evaluation — the paper notes no
  gold-standard peak-level annotation of real data was available, which is precisely
  why synthetics are generated
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: mzBucket
  role: Hosts the createSynthetics Python module that implements the synthetic data
    generation recipe
  repo: https://github.com/hildebrandtlab/mzBucket
- name: proteolizard-data
  role: Python/C++ framework component used by createSynthetics for data structures
    and data access (windows, spectra, binning)
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: Python
  role: Implementation language of the createSynthetics module in the proteolizard
    framework
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:71b4683155ea800aa88475174c2baf5afa4cb161ac35e064f10e54737462fb98
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# synthetic-mass-spectrometry-data-generation

## Summary

Generates ground-truth-annotated synthetic timsTOF-style mass spectrometry windows (noise-only and signal+noise) using the createSynthetics module of the mzBucket/proteolizard framework, with Poisson-distributed noise peaks and averagine-modeled isotope patterns, to benchmark signal-classification methods such as LSH against intensity thresholding.

## When to use

Use when you need labeled synthetic mass spectrometry peak data with known signal/noise ground truth (e.g., to compute ROC curves for a classification method) because real peak-level annotations are unavailable.

## When NOT to use

- The input is real timsTOF raw data in Bruker .d format — access it via proteolizard-data/OpenTIMS instead of generating synthetic windows
- You need to model chemical/repeated-contaminant noise, which is self-similar and not captured by the Poisson noise model used here
- You need a validated real-data gold standard for evaluation — the paper notes no gold-standard peak-level annotation of real data was available, which is precisely why synthetics are generated

## Inputs

- Configuration of the createSynthetics module (window width 10 m/z, bin width 0.1 m/z, noise distributions, averagine mass/charge ranges, intensity scales)
- Averagine isotope-model parameters (monoisotopic masses 150–5000 u, charge states 1–5)

## Outputs

- Synthetic noise-only and signal+noise window datasets at six maximum-intensity scales (1000, 500, 250, 125, 64, 32)
- Per-peak ground-truth labels: signal, noise_1, or noise_2
- Files written under the data/output_create_synthetics folder of the mzBucket repository

## How to apply

Configure the createSynthetics Python module in the proteolizard framework to cut the mass axis into windows of 10 m/z with equally spaced 0.1 m/z bins. Sample noise stochastically: peak counts from a Poisson(4)+1 distribution and peak intensities from an exponential distribution with parameter 15. Model signal as isotope patterns using the averagine model for monoisotopic masses in the 150–5000 u range and charge states 1–5, then assign each peak one of three ground-truth labels: signal, noise_1, or noise_2. Repeat the whole generation at each of the six maximum-intensity scales (1000, 500, 250, 125, 64, 32) so classification performance can be evaluated as a function of signal-to-noise ratio, and write results to data/output_create_synthetics. These labeled sets then serve as reference inputs for both LSH-based classification and intensity-threshold filtering so ROC curves are comparable.

## Related tools

- **mzBucket** (Hosts the createSynthetics Python module that implements the synthetic data generation recipe) — https://github.com/hildebrandtlab/mzBucket
- **proteolizard-data** (Python/C++ framework component used by createSynthetics for data structures and data access (windows, spectra, binning)) — https://github.com/theGreatHerrLebert/proteolizard-data
- **Python** (Implementation language of the createSynthetics module in the proteolizard framework)

## Evaluation signals

- Output files for all six intensity scales (1000, 500, 250, 125, 64, 32) exist under data/output_create_synthetics
- Generated noise peaks follow the specified distributions: counts ~ Poisson(4)+1, intensities ~ exponential with parameter 15
- Windows conform to the 10 m/z width with 0.1 m/z equally spaced binning recipe
- Averagine-modeled signals fall within 150–5000 u monoisotopic mass and charge states 1–5
- Every generated peak carries exactly one of the three ground-truth labels (signal, noise_1, noise_2) — a qualitative acceptance check, since the paper reports no numeric threshold for generation fidelity

## Limitations

- Synthetic data cannot fully reflect the behavior of real data
- Noise peaks located within signal windows (label noise_2) are not removed by downstream LSH classification and require subsequent processing steps
- The paper reports no numeric threshold for synthetic generation fidelity, so validation is a qualitative parameter-conformance check

## Evidence

- [methods] Sample noise peaks stochastically, drawing peak counts from a Poisson(4)+1 distribution and peak intensities from an exponential distribution with parameter 15.: [evidence span withheld — anchor status: not_found]
- [methods] Assign each synthetic peak one of the three ground-truth labels.: [evidence span withheld — anchor status: not_found]
- [methods] Model isotope patterns with the averagine model for monoisotopic masses in the range 150–5000 u and charge states 1–5.: [evidence span withheld — anchor status: not_found]
- [abstract] Generated data and code are available at the mzBucket repository.: "Generated data and code are available at https://github.com/hildebrandtlab/mzBucket"
- [discussion] Synthetic data does not fully capture real-data behavior.: "synthetic data cannot fully reflect the behavior of real data"
- [readme] README confirms the createSynthetics module location within the repository structure.: "├── src
│   ├── python
│   │   ├── createSynthetics"
