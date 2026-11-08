---
name: parameter-uncertainty-estimation
description: Use when fitting 1D/pseudo-2D NMR spectra where quantitative signal parameters
  and their standard-deviation uncertainties are required, especially to decide whether
  to fit whole multiplets versus independent peaks.
when_to_use_negative:
- Spectra without well-resolved or identifiable multiplets where semi-automated peak
  picking cannot define multiplicity reliably
- Analyses where only peak detection/annotation is needed and no quantitative parameter
  estimation or uncertainty is required
- Data requiring higher-order baseline correction beyond the zero-order baseline correction
  MultiNMRFit accounts for
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MultiNMRFit
  role: Fits whole multiplets or individual peaks in 1D/pseudo-2D NMR spectra and
    reports estimated parameter standard deviations; used to run both fitting strategies
    for the comparison
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Installs the multinmrfit package used for the fitting workflow
- name: Python 3.8
  role: Runtime environment for MultiNMRFit, also usable as a library for scripted
    fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:81c7b95796a1d68b3473ea1fabc9c772fd577feba30f76133741d985425104c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - Python 3.8
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# parameter-uncertainty-estimation

## Summary

Fit 1D or pseudo-2D NMR spectra with MultiNMRFit using whole-multiplet signal models to estimate signal parameters (intensity, area, chemical shift, linewidth, coupling constants) together with their uncertainties as standard deviations, and benchmark the multiplet strategy against per-peak fitting on synthetic spectra with known ground truth.

## When to use

Use when fitting 1D/pseudo-2D NMR spectra where quantitative signal parameters and their standard-deviation uncertainties are required, especially to decide whether to fit whole multiplets versus independent peaks.

## When NOT to use

- Spectra without well-resolved or identifiable multiplets where semi-automated peak picking cannot define multiplicity reliably
- Analyses where only peak detection/annotation is needed and no quantitative parameter estimation or uncertainty is required
- Data requiring higher-order baseline correction beyond the zero-order baseline correction MultiNMRFit accounts for

## Inputs

- 1D (or pseudo-2D) NMR spectra, e.g. synthetic spectra provided as tabulated text files with known ground-truth signal parameters
- Signal models for multiplets (built-in or user-defined) defined via semi-automated peak picking

## Outputs

- Estimated parameters (intensity, area, chemical shift, linewidth, coupling constants) per signal with uncertainties as standard deviations
- Accuracy/precision comparison table (multiplet_vs_peak_comparison.csv)
- Summary comparison figure of fitted curves (multiplet_vs_peak_figure3.png reproducing supplementary Fig. 3)

## How to apply

Install the environment with `pip install multinmrfit` on Python 3.8+, then load the spectra (e.g. synthetic 1D spectra provided as tabulated text files for which true parameters are known). Fit each whole multiplet as a single signal model with MultiNMRFit, letting semi-automated peak picking define multiplicity while accounting for overlaps between peaks and zero-order baseline correction. Refit the same spectra defining each peak as an independent signal, then extract estimated parameters (intensity, area, chemical shift, linewidth, coupling constants) and their reported standard deviations for both strategies. Compute accuracy (deviation from ground truth) and precision (parameter standard deviations) per strategy and tabulate them (multiplet_vs_peak_comparison.csv), and visually inspect fitted curves to produce a summary figure. Accept the comparison if the multiplet-fitting strategy shows lower parameter deviation from ground truth and smaller standard deviations than the per-peak strategy, reproducing the supplementary Fig. 3 trend that multiplet fitting increased accuracy and precision.

## Related tools

- **MultiNMRFit** (Fits whole multiplets or individual peaks in 1D/pseudo-2D NMR spectra and reports estimated parameter standard deviations; used to run both fitting strategies for the comparison) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Installs the multinmrfit package used for the fitting workflow)
- **Python 3.8** (Runtime environment for MultiNMRFit, also usable as a library for scripted fitting)

## Examples

```
pip install multinmrfit && nmrfit
```

## Evaluation signals

- Multiplet-fitting strategy shows lower deviation of estimated parameters from ground truth than per-peak fitting on synthetic spectra
- Estimated parameter standard deviations (precision) are smaller for the whole-multiplet strategy than for independent-peak fitting
- Per-signal parameters (intensity, area, chemical shift, linewidth, coupling constants) are reported together with standard-deviation uncertainties
- Comparison table multiplet_vs_peak_comparison.csv is produced with accuracy and precision tabulated per strategy
- Visual inspection of fitted curves matches the supplementary Fig. 3 trend (multiplet fitting increased accuracy and precision)

## Limitations

- Fitting each peak independently reduces the accuracy and precision of estimated parameters compared with fitting whole multiplets
- Overlaps between peaks and baseline must be accounted for (MultiNMRFit handles overlaps and zero-order baseline correction); stronger baseline structure may not be corrected
- No discussion section text available; no limitations, missing artifacts, or reproducibility gaps could be extracted from the article

## Evidence

- [other] multiplet-vs-individual-peak finding: [evidence span withheld — anchor status: not_found]
- [readme] readme-parameters: "estimation of several parameters for each signal of interest (**intensity**, **area**, **chemical shift**, **linewidth**, **coupling constant(s)**, etc),"
- [readme] readme-uncertainty: "estimation of **uncertainty** on estimated parameters (standard deviation),"
- [abstract] abstract-models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [methods] methods-install: "Use `pip` to **install PhysioFit from GitHub**:

bash
$ pip install multinmrfit"
