---
name: spectral-peak-deconvolution
description: Use when 1D NMR spectra (1H, 13C, 31P, or any nucleus) contain overlapping
  peaks that must be deconvolved into quantitative signal parameters, e.g. metabolomics
  or isotope-labeling spectra provided as tabulated text files.
when_to_use_negative:
- The task is peak integration of well-separated, non-overlapping peaks where a full
  deconvolution fit adds unnecessary complexity
- The spectra are not 1D or pseudo-2D NMR (e.g. mass spectra or Raman spectra), which
  the MultiNMRFit signal models were not designed for
- A fully automated, GUI-free batch analysis is required without semi-automated peak-picking
  validation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MultiNMRFit
  role: Fits 1D/pseudo-2D NMR spectra with multiplet signal models, semi-automated
    peak picking, overlap handling, zero-order baseline correction, and uncertainty
    estimation; shipped as a Python library with a graphical user interface
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python 3.8
  role: Runtime requirement; MultiNMRFit is implemented in Python 3 and installable
    as a library for scripted deconvolution workflows
- name: pip
  role: Installs the multinmrfit package from GitHub/PyPI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:f2b793f179758e36b4eb79302355010668266160eda585217da1517d46ef6c18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - Python 3.8
  - pip
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# spectral-peak-deconvolution

## Summary

Fit 1D and pseudo-2D NMR spectra with MultiNMRFit to deconvolve overlapping peaks into whole multiplet signal models, extracting intensity, area, chemical shift, linewidth, and coupling constants with standard-deviation uncertainties. Fitting multiplets rather than individual peaks improves accuracy and precision of estimated parameters.

## When to use

Use when 1D NMR spectra (1H, 13C, 31P, or any nucleus) contain overlapping peaks that must be deconvolved into quantitative signal parameters, e.g. metabolomics or isotope-labeling spectra provided as tabulated text files.

## When NOT to use

- The task is peak integration of well-separated, non-overlapping peaks where a full deconvolution fit adds unnecessary complexity
- The spectra are not 1D or pseudo-2D NMR (e.g. mass spectra or Raman spectra), which the MultiNMRFit signal models were not designed for
- A fully automated, GUI-free batch analysis is required without semi-automated peak-picking validation

## Inputs

- 1D NMR spectra acquired individually or as pseudo-2D experiments
- Spectra provided as tabulated text files
- Synthetic or experimental spectra of 1H, 13C, 15N, 31P, or other nuclei
- Optionally, tabulated ground-truth signal parameters for synthetic validation spectra

## Outputs

- Estimated signal parameters per multiplet: intensity, area, chemical shift, linewidth, coupling constant(s)
- Uncertainty on estimated parameters (standard deviations)
- Fitted curves for visual inspection
- Tabulated parameter comparison (e.g. multiplet_vs_peak_comparison.csv)

## How to apply

Install the fitting environment with `pip install multinmrfit` on Python 3.8+ and launch the graphical interface with `nmrfit`. Load the spectrum (individual 1D, pseudo-2D, or tabulated text file) and use the semi-automated peak picking to define multiplicity for each signal, letting MultiNMRFit account for overlaps between peaks and apply zero-order baseline correction. Define each whole multiplet as a single signal model — using built-in or user-defined models — rather than fitting each peak independently, since multiplet fitting was shown on synthetic spectra with known ground truth to increase both accuracy (deviation from true parameters) and precision (estimated standard deviations). Fit the spectra and extract the estimated parameters (intensity, area, chemical shift, linewidth, coupling constants) with their uncertainties reported as standard deviations. Validate results by visual inspection of the fitted curves, which the GUI supports.

## Related tools

- **MultiNMRFit** (Fits 1D/pseudo-2D NMR spectra with multiplet signal models, semi-automated peak picking, overlap handling, zero-order baseline correction, and uncertainty estimation; shipped as a Python library with a graphical user interface) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python 3.8** (Runtime requirement; MultiNMRFit is implemented in Python 3 and installable as a library for scripted deconvolution workflows)
- **pip** (Installs the multinmrfit package from GitHub/PyPI)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- On synthetic spectra with known ground-truth parameters, the multiplet-fitting strategy shows lower deviation from true values and smaller estimated standard deviations than fitting each peak independently
- Estimated parameters include intensity, area, chemical shift, linewidth, and coupling constants, each with a reported standard deviation
- Visual inspection of fitted curves confirms the fitted model reproduces peak positions, multiplicities, and overlaps
- The software runs on Python 3.8 or higher and on Windows, MacOS, and Unix platforms

## Limitations

- Peak picking and multiplicity definition are semi-automated, so manual curation is needed for complex or strongly overlapped regions
- Baseline correction is zero-order only; spectra with higher-order baseline distortions may fit poorly
- Multiplet fitting was validated against synthetic spectra with known parameters; behavior on highly irregular experimental multiplets should be checked by visual inspection

## Evidence

- [abstract] MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models that account for"
- [abstract] We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
- [readme] semi-automated analysis for peak picking and definition of multiplicity for each signal: "**semi-automated analysis** for **peak picking** and **definition of multiplicity** for each signal,"
- [readme] account for overlaps between peaks and zero-order baseline correction: "account for **overlaps** between peaks and **zero-order baseline correction**,"
- [readme] MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix): "MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix)."
- [other] Fitting multiplets rather than individual peaks increased the accuracy and precision of the estimated parameters: [evidence span withheld — anchor status: not_found]
