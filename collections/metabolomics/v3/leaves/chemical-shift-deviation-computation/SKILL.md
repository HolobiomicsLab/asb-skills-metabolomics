---
name: chemical-shift-deviation-computation
description: Use when you have paired experimental NMR mixture spectra and pure-component
  reference spectra and need to quantify how far identical peaks shift (ppm) between
  mixture and reference before applying shift-tolerant identification.
when_to_use_negative:
- You need compound identification of mixture spectra itself — use the pSCNN classification
  model, not this deviation statistics computation
- Your spectra are simulated/augmented training spectra with no experimental mixture-vs-reference
  pairing, since the deviation interval is defined only for experimental spectral
  pairs
- Peaks cannot be reliably paired between mixture and reference spectra (e.g., severe
  peak overlapping with no unambiguous correspondence)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: Python 3.8.13
  role: Runtime environment (with TensorFlow 2.5.0-GPU) in which spectra are loaded,
    peaks paired, and deviation statistics computed
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: pSCNN
  role: Source codebase and Zenodo-deposited model/NMR data supplying the mixture
    and pure-component spectra used for peak pairing and deviation calculation
  repo: https://github.com/yuxuanliao/pSCNN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:3a9baa5dcf0e682fb43d0c5fc59dfdfbdf416f7a67353e74e649c9343d1f19d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Python
  - Python 3.8.13
  - pSCNN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# chemical-shift-deviation-computation

## Summary

Computes the magnitude and interval of chemical shift deviations (in ppm) between experimental NMR mixture spectra and their pure-component reference spectra, after excluding active-hydrogen and DMSO-d6/HDO solvent signals, to quantify the chemical shift variation problem that motivates pSCNN.

## When to use

Use when you have paired experimental NMR mixture spectra and pure-component reference spectra and need to quantify how far identical peaks shift (ppm) between mixture and reference before applying shift-tolerant identification.

## When NOT to use

- You need compound identification of mixture spectra itself — use the pSCNN classification model, not this deviation statistics computation
- Your spectra are simulated/augmented training spectra with no experimental mixture-vs-reference pairing, since the deviation interval is defined only for experimental spectral pairs
- Peaks cannot be reliably paired between mixture and reference spectra (e.g., severe peak overlapping with no unambiguous correspondence)

## Inputs

- Experimental NMR spectra of mixtures (e.g., flavor mixtures dataset from Zenodo DOI 10.5281/zenodo.6504814)
- Pure-component NMR reference spectra corresponding to the mixture components
- List/annotation of active-hydrogen signals and residual solvent signals (DMSO-d6/HDO) to exclude

## Outputs

- Per-pair chemical shift deviations in ppm (mixture vs. pure-component peak pairs)
- chemical_shift_deviation_stats.csv with deviation statistics table (mean, standard deviation, mean ± 1.5 SD interval)

## How to apply

Download the deposited pSCNN model and NMR example data from Zenodo (DOI 10.5281/zenodo.6504814) and load the experimental mixture spectra and corresponding pure-component reference spectra in Python 3.8.13. Detect and pair peaks between each mixture spectrum and its component reference spectrum across the ppm axis, then exclude active-hydrogen signals and the residual solvent signals of DMSO-d6/HDO from the peak pairs before computing deviations. Calculate the chemical shift deviation in ppm for each retained spectral pair and aggregate the mean, standard deviation, and a mean ± 1.5 × SD interval with numpy/pandas. In the source article this yielded a mean of −0.0016 ppm, a standard deviation of 0.0077 ppm, and a mean ± 1.5 SD interval of (−0.013, 0.010) ppm; write the deviation statistics to chemical_shift_deviation_stats.csv. The rationale is to establish an empirical tolerance band for chemical shift variation, which the pSCNN method addresses architecturally via the translational invariance of convolutional neural networks.

## Related tools

- **Python 3.8.13** (Runtime environment (with TensorFlow 2.5.0-GPU) in which spectra are loaded, peaks paired, and deviation statistics computed) — https://github.com/yuxuanliao/pSCNN.git
- **pSCNN** (Source codebase and Zenodo-deposited model/NMR data supplying the mixture and pure-component spectra used for peak pairing and deviation calculation) — https://github.com/yuxuanliao/pSCNN

## Examples

```
python pSCNN.py  # after downloading model + NMR example data from Zenodo 10.5281/zenodo.6504814; pair peaks mixture-vs-reference, drop DMSO-d6/HDO pairs, write chemical_shift_deviation_stats.csv
```

## Evaluation signals

- Reproduced mean deviation equals −0.0016 ppm within numerical tolerance
- Reproduced standard deviation equals 0.0077 ppm within numerical tolerance
- Reproduced mean ± 1.5 × SD interval equals (−0.013, 0.010) ppm within numerical tolerance
-  chemical_shift_deviation_stats.csv is produced and contains the aggregated deviation statistics
- All active-hydrogen and residual DMSO-d6/HDO solvent peak pairs are absent from the retained deviation set

## Limitations

- Only experimental mixture-vs-reference spectral pairs are characterized; the interval may not cover shifts in augmented or simulated spectra
- Deviation estimates are biased if active-hydrogen signals or residual DMSO-d6/HDO solvent peaks are not excluded from the pairs
- Peak detection/pairing quality on the ppm axis determines which deviations enter the statistics; overlapping peaks may yield unreliable pairs

## Evidence

- [results] chemical shift variation interval of (-0.013, 0.010) ppm: [evidence span withheld — anchor status: not_found]
- [results] active-hydrogen and solvent signals excluded: [evidence span withheld — anchor status: not_found]
- [methods] Zenodo deposit with model and NMR data: "zenodo.org/badge/DOI/10.5281/zenodo.6504814.svg)](https://doi.org/10.5281/zenodo.6504814)"
- [readme] Python and TensorFlow versions: "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [intro] chemical shift variation problem statement: "is difficult to identify compounds in mixtures because of chemical shift variation of the same compound in different mixtures and peak overlapping among molecules"
