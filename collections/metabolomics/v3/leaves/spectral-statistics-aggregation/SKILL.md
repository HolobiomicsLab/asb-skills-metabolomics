---
name: spectral-statistics-aggregation
description: Use when you have paired peak lists from mixture NMR spectra and their
  pure-component references (ppm axis) and need to quantify chemical shift deviation
  statistics or derive a tolerance interval for peak-pairing.
when_to_use_negative:
- Spectra were acquired in a solvent system without a well-defined residual solvent
  peak (DMSO-d6/HDO) or contain no active-hydrogen signals — the exclusion step does
  not apply as specified
- The goal is compound identification/classification of mixtures rather than quantifying
  chemical shift variability — use the pSCNN prediction workflow instead
- Input is already an aggregated statistics table with no raw spectral peak pairs
  available for recomputation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: Python 3.8.13
  role: Runtime environment (with TensorFlow 2.5.0-GPU via Anaconda) in which spectra
    are loaded, peaks paired, and deviation statistics computed with numpy/pandas
  repo: https://github.com/yuxuanliao/pSCNN
- name: pSCNN
  role: Source of the NMR model and example mixture/pure-component spectra data used
    for the chemical shift deviation analysis; the deviation interval it tolerates
    is what this skill quantifies
  repo: https://github.com/yuxuanliao/pSCNN
- name: Zenodo
  role: Hosts the trained model and all NMR data referenced by the analysis (DOI 10.5281/zenodo.6504814)
  repo: https://doi.org/10.5281/zenodo.6504814
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:07135a62e64ea2dc3bc26a5a48c318d43e60d360c887ceff949080f7a7f466cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN
  tools:
  - Python
  - Python 3.8.13
  - Zenodo
  - pSCNN
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# spectral-statistics-aggregation

## Summary

Quantifies chemical shift variation between experimental NMR mixture spectra and their pure-component references by aggregating per-pair ppm deviations into mean, standard deviation, and a mean ± 1.5 × SD tolerance interval. This characterizes how much peak positions drift across mixtures, which is the problem pSCNN's translational invariance is meant to absorb.

## When to use

Use when you have paired peak lists from mixture NMR spectra and their pure-component references (ppm axis) and need to quantify chemical shift deviation statistics or derive a tolerance interval for peak-pairing.

## When NOT to use

- Spectra were acquired in a solvent system without a well-defined residual solvent peak (DMSO-d6/HDO) or contain no active-hydrogen signals — the exclusion step does not apply as specified
- The goal is compound identification/classification of mixtures rather than quantifying chemical shift variability — use the pSCNN prediction workflow instead
- Input is already an aggregated statistics table with no raw spectral peak pairs available for recomputation

## Inputs

- Experimental NMR mixture spectra (ppm axis, from the flavor mixtures / additional flavor mixture datasets)
- Pure-component reference NMR spectra for the compounds in each mixture
- Peak-pair list (mixture peak vs. component reference peak) with chemical shifts in ppm
- List of signals to exclude (active-hydrogen signals, DMSO-d6/HDO residual solvent peaks)

## Outputs

- Per-pair chemical shift deviations in ppm
- Aggregated statistics: mean (−0.0016 ppm), standard deviation (0.0077 ppm), and mean ± 1.5 × SD interval (−0.013, 0.010) ppm
- chemical_shift_deviation_stats.csv (deviation statistics table)

## How to apply

Load the experimental mixture spectra and corresponding pure-component reference spectra (NMR data downloadable from Zenodo DOI 10.5281/zenodo.6504814) in Python 3.8.13 with numpy/pandas. Detect and pair peaks between each mixture spectrum and its component reference across the ppm axis, then filter out active-hydrogen signals and residual solvent signals (DMSO-d6/HDO) before computing deviations, since these do not reflect true chemical shift variation. For each retained spectral pair, compute the chemical shift deviation in ppm and aggregate the mean, standard deviation, and the mean ± 1.5 × SD interval, writing the results to a CSV statistics table. The rationale is to empirically measure how far the same compound's peaks shift in different mixtures — a variation interval of (−0.013, 0.010) ppm was observed in this study and motivates shift-tolerant identification methods.

## Related tools

- **Python 3.8.13** (Runtime environment (with TensorFlow 2.5.0-GPU via Anaconda) in which spectra are loaded, peaks paired, and deviation statistics computed with numpy/pandas) — https://github.com/yuxuanliao/pSCNN
- **pSCNN** (Source of the NMR model and example mixture/pure-component spectra data used for the chemical shift deviation analysis; the deviation interval it tolerates is what this skill quantifies) — https://github.com/yuxuanliao/pSCNN
- **Zenodo** (Hosts the trained model and all NMR data referenced by the analysis (DOI 10.5281/zenodo.6504814)) — https://doi.org/10.5281/zenodo.6504814

## Examples

```
python pSCNN.py  # after: conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt
```

## Evaluation signals

- Reproduced mean chemical shift deviation matches −0.0016 ppm within numerical tolerance
- Reproduced standard deviation matches 0.0077 ppm within numerical tolerance
- Computed mean ± 1.5 × SD interval equals (−0.013, 0.010) ppm
- All active-hydrogen and DMSO-d6/HDO residual solvent peak pairs are absent from the deviation set before aggregation
- chemical_shift_deviation_stats.csv is written and its values agree with the article-reported statistics

## Limitations

- The deviation interval was derived from specific experimental flavor-mixture NMR datasets acquired with DMSO-d6/HDO solvent signals; it may not generalize to other solvents, field strengths, or compound classes
- Active-hydrogen signals and residual solvent signals must be identified and excluded manually/via filters; misclassification of such peaks would bias the statistics
- No discussion-section content was available documenting reproducibility gaps or limitations of the deviation analysis

## Evidence

- [intro] it is difficult to identify compounds in mixtures because of chemical shift variation of the same compound in different mixtures and peak overlapping among molecules: "is difficult to identify compounds in mixtures because of chemical shift variation of the same compound in different mixtures and peak overlapping among molecules"
- [abstract] We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra.: "We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra."
- [methods] Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo.: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo. Download the model and these example data, pSCNN can be reload and predict easily."
- [readme] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [abstract] Two experimental NMR datasets (flavor mixtures and additional flavor mixture) were acquired to benchmark its performance in real applications.: "Two experimental NMR datasets (flavor mixtures and additional flavor mixture) were acquired to benchmark its performance in real applications."
