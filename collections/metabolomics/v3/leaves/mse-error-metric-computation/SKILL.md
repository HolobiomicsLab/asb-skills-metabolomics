---
name: mse-error-metric-computation
description: Use when simulated or spiked data sets have known ground-truth metabolite
  concentrations and you must quantify how well a deconvolution/quantification method
  (e.g. SAND, CRAFT, peak intensity, numerical integration) recovers them.
when_to_use_negative:
- Input is experimental data with no known ground-truth concentrations — regression
  against ground truth cannot be computed
- Input is already a feature table of computed error metrics rather than raw quantification
  estimates
- The goal is peak clustering/annotation quality (e.g. correlation-network cluster
  recovery) rather than concentration-quantification accuracy
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3435
tools:
- name: SAND
  role: Produces the quantification estimates under test, via hybrid Markov chain
    Monte Carlo time-domain decomposition, which are regressed against ground truth
  repo: github.com/edisonomics/SAND
- name: NMRBox
  role: Cloud environment in which SAND (and thus the benchmark workflow) is run via
    a single terminal command
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:9723002ab47245009833d9489244d8765dedd52a9e393763c699a892adc2dd64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NMRBox
  - SAND
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3435
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mse-error-metric-computation

## Summary

Regression-based accuracy assessment of metabolite quantification methods by regressing estimated concentrations on known ground-truth values and computing MSE, relative MSE, slope k, and Pearson correlation, as used to benchmark SAND against peak-intensity, numerical-integral, and CRAFT baselines on simulated NMR data sets.

## When to use

Use when simulated or spiked data sets have known ground-truth metabolite concentrations and you must quantify how well a deconvolution/quantification method (e.g. SAND, CRAFT, peak intensity, numerical integration) recovers them.

## When NOT to use

- Input is experimental data with no known ground-truth concentrations — regression against ground truth cannot be computed
- Input is already a feature table of computed error metrics rather than raw quantification estimates
- The goal is peak clustering/annotation quality (e.g. correlation-network cluster recovery) rather than concentration-quantification accuracy

## Inputs

- Simulated NMR spectra data sets with known ground-truth metabolite concentrations
- Quantification estimates from SAND (time-domain MCMC decomposition)
- Quantification estimates from baseline methods: peak intensity, numerical integral, and CRAFT deconvolution

## Outputs

- Per-method, per-dataset MSE, relative MSE, regression slope k, and Pearson correlation values
- quantification_metrics_comparison.csv containing all computed metrics

## How to apply

For each method and each simulated data set, take the quantification estimates produced by the method (e.g. SAND's time-domain MCMC decomposition, CRAFT deconvolution, peak-intensity or numerical-integral readouts) and regress them against the ground-truth concentrations of the simulated mixtures. Record four metrics per method/data-set pair: mean squared error (MSE), relative MSE, the regression slope k, and the Pearson correlation coefficient. Interpret slope k near 1 and correlation near 1 as accurate recovery; in the SAND benchmark, SAND achieved a correlation of approximately 0.88 and slope k of approximately 0.9 against ground truth on the more complex simulated data sets, making these values the reference targets. Compare methods per data set (e.g. CRAFT slightly outperformed SAND on two data sets) rather than only in aggregate, and write all per-method, per-dataset metrics to a CSV (quantification_metrics_comparison.csv) for reproducible comparison.

## Related tools

- **SAND** (Produces the quantification estimates under test, via hybrid Markov chain Monte Carlo time-domain decomposition, which are regressed against ground truth) — github.com/edisonomics/SAND
- **NMRBox** (Cloud environment in which SAND (and thus the benchmark workflow) is run via a single terminal command)

## Examples

```
python scripts/compare_quantification.py --methods SAND,CRAFT,peak_intensity,numerical_integral --datasets simulated_test_examples --ground-truth concentrations.csv --out quantification_metrics_comparison.csv
```

## Evaluation signals

- SAND's estimates on the complex simulated data sets reach a Pearson correlation of approximately 0.88 with ground truth
- Regression slope k against ground truth is approximately 0.9 for SAND on the complex simulated data sets
- quantification_metrics_comparison.csv exists and contains one row per method per simulated data set with MSE, relative MSE, k, and correlation columns populated
- Per-dataset comparison identifies the two data sets where CRAFT slightly outperforms SAND, confirming the metrics are computed at data-set resolution rather than aggregated

## Limitations

- Metrics are only meaningful when ground-truth concentrations are known, so this validation applies to simulated (or rigorously spiked) data sets
- In the SAND benchmark, CRAFT was slightly better on two of the simulated data sets, so no single method dominates every case
- Correlation and slope summarize agreement but do not capture per-metabolite errors; report MSE and relative MSE alongside them

## Evidence

- [results] Among the automatic quantification methods, SAND was consistently better on the more complex simulated data sets, achieving a correlation of 0.88 and a slope k of around 0.9 against ground truth.: [evidence span withheld — anchor status: not_found]
