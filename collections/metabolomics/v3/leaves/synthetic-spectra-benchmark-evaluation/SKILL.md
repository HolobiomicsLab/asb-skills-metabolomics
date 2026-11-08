---
name: synthetic-spectra-benchmark-evaluation
description: Use when validating a peak-fitting implementation on spectra where true
  signal parameters are known, or when deciding whether to fit whole multiplets as
  single signal models versus independent peaks.
when_to_use_negative:
- Input is experimental spectra without known ground-truth parameters — accuracy cannot
  be computed, so the benchmark acceptance criterion cannot be evaluated
- The tool under test does not support whole-multiplet signal models or semi-automated
  peak picking with multiplicity definition, making the two-strategy comparison impossible
- Goal is routine quantitative fitting of experimental 1H, 13C, or 31P datasets rather
  than method validation/benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MultiNMRFit
  role: Fitting engine used to fit the synthetic spectra under both strategies (whole
    multiplet as single signal model vs independent peaks) and to report parameter
    uncertainties as standard deviations
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Installs the multinmrfit package needed to run the benchmark fits
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python
  role: Runtime (Python 3.8+) in which MultiNMRFit is executed as a library to script
    the two fitting strategies and the accuracy/precision computation
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:6892240bf6e7640ea3cd543ef26a8513957a18800adf255d035cd79be39b6c76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# synthetic-spectra-benchmark-evaluation

## Summary

Benchmark a spectral fitting tool (MultiNMRFit) against synthetic 1D NMR spectra with known ground-truth parameters, comparing whole-multiplet fitting versus per-peak fitting to quantify accuracy and precision of estimated parameters.

## When to use

Use when validating a peak-fitting implementation on spectra where true signal parameters are known, or when deciding whether to fit whole multiplets as single signal models versus independent peaks.

## When NOT to use

- Input is experimental spectra without known ground-truth parameters — accuracy cannot be computed, so the benchmark acceptance criterion cannot be evaluated
- The tool under test does not support whole-multiplet signal models or semi-automated peak picking with multiplicity definition, making the two-strategy comparison impossible
- Goal is routine quantitative fitting of experimental 1H, 13C, or 31P datasets rather than method validation/benchmarking

## Inputs

- Synthetic 1D NMR spectra as tabulated text files with known true signal parameters
- Ground-truth parameter tables (intensity, area, chemical shift, linewidth, coupling constants) for the synthetic signals

## Outputs

- multiplet_vs_peak_comparison.csv tabulating accuracy (deviation from ground truth) and precision (parameter standard deviations) per fitting strategy
- Summary figure multiplet_vs_peak_figure3.png reproducing the supplementary Fig. 3 comparison via visual inspection of fitted curves

## How to apply

Generate or obtain synthetic 1D spectra provided as tabulated text files for which the true signal parameters (intensity, area, chemical shift, linewidth, coupling constants) are known. Fit each spectrum with MultiNMRFit (installed via pip on Python 3.8+) defining each whole multiplet as a single signal model, using the semi-automated peak picking to define multiplicity while accounting for overlaps between peaks and zero-order baseline correction. Refit the identical spectra defining each peak as an independent signal. For both strategies, extract the estimated parameters and their uncertainties reported as standard deviations, then compute accuracy as deviation from ground truth and precision as parameter standard deviations per strategy. Accept the benchmark only if the multiplet-fitting strategy shows lower parameter deviation from ground truth and smaller estimated standard deviations than the per-peak strategy.

## Related tools

- **MultiNMRFit** (Fitting engine used to fit the synthetic spectra under both strategies (whole multiplet as single signal model vs independent peaks) and to report parameter uncertainties as standard deviations) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Installs the multinmrfit package needed to run the benchmark fits) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python** (Runtime (Python 3.8+) in which MultiNMRFit is executed as a library to script the two fitting strategies and the accuracy/precision computation) — https://github.com/NMRTeamTBI/MultiNMRFit/

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- The multiplet-fitting strategy yields lower parameter deviation from ground truth than the per-peak strategy for intensity, area, chemical shift, linewidth, and coupling constants
- The multiplet-fitting strategy yields smaller estimated parameter standard deviations (better precision) than the per-peak strategy
- Fitted curves for both strategies pass visual inspection, matching the supplementary Fig. 3 trend
- Estimated parameters and uncertainties (standard deviations) are successfully extracted for both fitting strategies across all synthetic spectra
- Reproduced comparison table (multiplet_vs_peak_comparison.csv) matches the reported increase in accuracy and precision for multiplet fitting

## Limitations

- Validation relies on synthetic spectra whose true parameters are known by construction; conclusions may not transfer directly to experimental spectra with unknown truth
- The semi-automated peak picking defines multiplicity for each signal, which can be confounded by overlaps between peaks
- Zero-order baseline correction only accounts for a constant baseline offset, not higher-order baselines

## Evidence

- [results] Finding: multiplet fitting improves accuracy and precision: [evidence span withheld — anchor status: not_found]
- [results] Workflow: extract parameters and uncertainties: [evidence span withheld — anchor status: not_found]
- [readme] Tool features supporting the benchmark: "estimation of several parameters for each signal of interest (**intensity**, **area**, **chemical shift**, **linewidth**, **coupling constant(s)**, etc)"
- [readme] Tool features: peak picking, overlaps, baseline: "account for **overlaps** between peaks and **zero-order baseline correction**"
- [methods] Install command: "Use `pip` to **install PhysioFit from GitHub**:

bash
$ pip install multinmrfit"
