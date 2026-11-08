---
name: metabolic-time-course-quantification-from-pseudo-2d-nmr
description: Use when you have a series of 1D NMR spectra (acquired individually or
  as a pseudo-2D experiment) from a metabolic time course — e.g. real-time glycolysis
  monitoring on 31P or 1H — and need fitted intensities, areas, chemical shifts, linewidths,
  and coupling constants per signal per time point.
when_to_use_negative:
- Input is 2D NMR data rather than a series of 1D spectra or a pseudo-2D experiment
  — MultiNMRFit is designed for 1D and pseudo-2D spectra only
- You need fully automated peak detection with no manual curation — the workflow prescribes
  semi-automated peak picking plus visual inspection of fitted curves by the analyst
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MultiNMRFit
  role: 'Core fitting tool: fits series of 1D/pseudo-2D NMR spectra with multiplet
    signal models, semi-automated peak picking, overlap handling, zero-order baseline
    correction, and uncertainty estimation'
  repo: https://github.com/NMRTeamTBI/MultiNRMRit
- name: Python
  role: Runtime and library interface (Python 3.8+); MultiNMRFit is implemented in
    Python 3 and shipped as a library with a graphical user interface
- name: pip
  role: Installation channel for multinmrfit from PyPI/GitHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:e9c0e1d883910fdb29624c160f8138d633c6e4db32dc26bcf952dc2f93d81464
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNRMRit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolic-time-course-quantification-from-pseudo-2d-nmr

## Summary

Quantitative time-course extraction from 1D NMR spectra acquired as pseudo-2D experiments (e.g. real-time glycolysis assays) using MultiNMRFit, an open-source Python 3 fitting tool with multiplet signal models, semi-automated peak picking, overlap handling, and standard-deviation uncertainties. Demonstrated on 1H, 13C, and 31P datasets.

## When to use

Use when you have a series of 1D NMR spectra (acquired individually or as a pseudo-2D experiment) from a metabolic time course — e.g. real-time glycolysis monitoring on 31P or 1H — and need fitted intensities, areas, chemical shifts, linewidths, and coupling constants per signal per time point.

## When NOT to use

- Input is 2D NMR data rather than a series of 1D spectra or a pseudo-2D experiment — MultiNMRFit is designed for 1D and pseudo-2D spectra only
- You need fully automated peak detection with no manual curation — the workflow prescribes semi-automated peak picking plus visual inspection of fitted curves by the analyst

## Inputs

- Series of 1D NMR spectra acquired as a pseudo-2D experiment (e.g. during a real-time glycolysis assay)
- Individual 1D NMR spectra or tabulated text files of spectra
- Spectra for any nucleus (1H, 13C, 15N, 31P, etc.)

## Outputs

- Parameter table per signal and time point: intensity, area, chemical shift, linewidth, coupling constant(s), each with standard-deviation uncertainty (e.g. exported to 31p_fit_results.csv)
- Fitted curves overlaid on experimental spectra for visual inspection (e.g. exported to 31p_fitted_spectra.png)

## How to apply

Install MultiNMRFit from PyPI (pip install multinmrfit; requires Python 3.8+, runs on Unix, Windows, MacOS) and launch the nmrfit graphical interface or use it as a Python library. Load the spectra series acquired as a pseudo-2D experiment (or as tabulated text files), then use the semi-automated peak picking to detect resonance positions and define multiplicity for each signal of interest. Fit each spectrum in the series with built-in (or user-defined) multiplet signal models, enabling accounting for overlaps between peaks and zero-order baseline correction. Extract the estimated intensity, area, chemical shift, linewidth, and coupling constant(s) for each signal together with their standard-deviation uncertainties, and visually inspect the fitted curves overlaid on the experimental spectra before exporting the parameter table and fitted-spectra plots (e.g. 31p_fit_results.csv and 31p_fitted_spectra.png). The tool works for all nuclei, including 31P, 13C, and 15N in addition to 1H.

## Related tools

- **MultiNMRFit** (Core fitting tool: fits series of 1D/pseudo-2D NMR spectra with multiplet signal models, semi-automated peak picking, overlap handling, zero-order baseline correction, and uncertainty estimation) — https://github.com/NMRTeamTBI/MultiNRMRit
- **Python** (Runtime and library interface (Python 3.8+); MultiNMRFit is implemented in Python 3 and shipped as a library with a graphical user interface)
- **pip** (Installation channel for multinmrfit from PyPI/GitHub)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Visual inspection of the fitted curves overlaid on the experimental 31P spectra confirms the fit quality, as the tool prescribes
- Fit converges and each estimated parameter (intensity, area, chemical shift, linewidth, coupling constant) is reported together with its standard-deviation uncertainty
- Peak picking correctly identifies the resonance positions and defined multiplicities for each signal of interest in the pseudo-2D series
- Parameter table exports cleanly (e.g. 31p_fit_results.csv) with one set of parameter estimates per spectrum in the time-course series
- Model accounts for observed peak overlaps and applies zero-order baseline correction without residual baseline offsets in the fitted plots

## Limitations

- Acceptance of fits is judged by visual inspection of the fitted curves, a subjective criterion rather than an automated threshold
- Documentation/source note 'No changelog found', and no discussion-section limitations were extractable from the article
- Signals with ambiguous multiplicities or extensive overlap may require manual curation of peak picking and multiplicity definitions

## Evidence

- [methods] MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [abstract] robustness and flexibility on 1H, 13C, and 31P datasets: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
- [readme] README key features list: "semi-automated analysis for **peak picking** and **definition of multiplicity** for each signal"
- [readme] README fit features: "account for **overlaps** between peaks and **zero-order baseline correction**"
- [methods] README platform requirements: "MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix)"
