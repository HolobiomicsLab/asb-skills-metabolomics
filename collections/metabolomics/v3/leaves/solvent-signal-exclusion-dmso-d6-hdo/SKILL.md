---
name: solvent-signal-exclusion-dmso-d6-hdo
description: Use when comparing experimental NMR mixture spectra against pure-component
  reference spectra in DMSO-d6 and you need chemical shift deviations free of active-hydrogen
  and residual solvent (DMSO-d6/HDO) contamination.
when_to_use_negative:
- Spectra recorded in a deuterated solvent other than DMSO-d6 — the specific exclusion
  of DMSO-d6/HDO residual signals will not match the artifact peaks present
- The goal is pure compound identification/prediction with the trained pSCNN model
  rather than quantifying chemical shift deviation statistics
- Only synthetic/augmented training spectra are being processed, where no real residual
  solvent peaks exist to exclude
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: Python 3.8.13
  role: Runtime environment in which the peak pairing, solvent-signal exclusion, and
    deviation aggregation (numpy/pandas) are implemented
- name: pSCNN
  role: Provides the deposited model and example NMR data (Zenodo 6504814) whose mixture
    vs. reference spectra are the inputs to the exclusion workflow
  repo: https://github.com/yuxuanliao/pSCNN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:fc158d7d7f1eb8343dcce05149511772cefae119eb6515af34fb6b7d7ff2230e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN
  tools:
  - Python
  - Python 3.8.13
  - pSCNN
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# solvent-signal-exclusion-dmso-d6-hdo

## Summary

A peak-pair filtering step that removes active-hydrogen signals and residual solvent peaks (DMSO-d6/HDO) from mixture-vs-reference NMR spectra before quantifying chemical shift deviations. This ensures the deviation statistics (mean −0.0016 ppm, SD 0.0077 ppm, mean ± 1.5 × SD interval of (−0.013, 0.010) ppm) reflect genuine chemical shift variation rather than solvent artifacts.

## When to use

Use when comparing experimental NMR mixture spectra against pure-component reference spectra in DMSO-d6 and you need chemical shift deviations free of active-hydrogen and residual solvent (DMSO-d6/HDO) contamination.

## When NOT to use

- Spectra recorded in a deuterated solvent other than DMSO-d6 — the specific exclusion of DMSO-d6/HDO residual signals will not match the artifact peaks present
- The goal is pure compound identification/prediction with the trained pSCNN model rather than quantifying chemical shift deviation statistics
- Only synthetic/augmented training spectra are being processed, where no real residual solvent peaks exist to exclude

## Inputs

- Experimental 1H NMR mixture spectra (ppm axis, acquired in DMSO-d6) from the flavor mixtures dataset
- Pure-component reference NMR spectra corresponding to the compounds in each mixture
- Peak-pair listings (mixture peak vs. component reference peak positions in ppm)

## Outputs

- Filtered peak-pair set with active-hydrogen and DMSO-d6/HDO solvent signals removed
- chemical_shift_deviation_stats.csv containing per-pair deviations plus mean (−0.0016 ppm), SD (0.0077 ppm), and mean ± 1.5 × SD interval (−0.013, 0.010 ppm)

## How to apply

Download the deposited pSCNN model and NMR example data from Zenodo (DOI 10.5281/zenodo.6504814), then load the experimental mixture spectra and their pure-component reference spectra in Python 3.8.13. Detect and pair peaks between each mixture spectrum and its component reference across the ppm axis. Before computing deviations, explicitly exclude active-hydrogen signals and the residual solvent signals of DMSO-d6/HDO from the peak pairs — these peaks shift for reasons unrelated to the compound's chemical environment. For each retained pair, compute the chemical shift deviation in ppm and aggregate mean, standard deviation, and the mean ± 1.5 × SD interval using numpy/pandas. The exclusion rationale follows from the paper's premise that chemical shift variation of the same compound across mixtures and peak overlapping among molecules make compound identification difficult, so non-compound peaks must not inflate the measured variation.

## Related tools

- **Python 3.8.13** (Runtime environment in which the peak pairing, solvent-signal exclusion, and deviation aggregation (numpy/pandas) are implemented)
- **pSCNN** (Provides the deposited model and example NMR data (Zenodo 6504814) whose mixture vs. reference spectra are the inputs to the exclusion workflow) — https://github.com/yuxuanliao/pSCNN

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python pSCNN.py
```

## Evaluation signals

- Reproduced mean chemical shift deviation equals −0.0016 ppm within numerical tolerance
- Reproduced standard deviation equals 0.0077 ppm within numerical tolerance
- Mean ± 1.5 × SD interval equals (−0.013, 0.010) ppm
- No DMSO-d6/HDO residual solvent peaks or active-hydrogen signals remain in the peak-pair set used for the statistics
- Statistics table is written to chemical_shift_deviation_stats.csv with per-pair deviations retained

## Limitations

- The exclusion thresholds and the reported interval are specific to spectra acquired in DMSO-d6; HDO position depends on temperature and water content
- No discussion-section content was available, so limitations and reproducibility gaps for this workflow were not documented in the article

## Evidence

- [intro] Chemical shift variation interval: "is difficult to identify compounds in mixtures because of chemical shift variation of the same compound in different mixtures and peak overlapping among molecules"
- [abstract] Translational invariance solves shift variation: "We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra."
- [readme] Zenodo data availability: "The model and these data have been uploaded at Zenodo. Download the model and these example data, pSCNN can be reload and predict easily."
- [intro] Environment specification: "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
