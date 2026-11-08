---
name: simulated-spectra-ground-truth-comparison
description: Use when evaluating quantification accuracy of NMR spectral deconvolution
  methods on simulated spectra where ground-truth metabolite concentrations are known
  and methods can be regressed against truth.
when_to_use_negative:
- Input is experimental (e.g., urine or worm) data with no known ground-truth concentrations
  — the regression-against-truth metrics cannot be computed
- Spectra are not high-resolution metabolomics 1D data; the current SAND version is
  tuned only for high-resolution metabolomics spectra
- The goal is peak annotation/clustering rather than quantification accuracy benchmarking
edam_operation: http://edamontology.org/operation_0335
tools:
- name: SAND
  role: Produces the quantification estimates under test via hybrid MCMC time-domain
    decomposition of the simulated spectra
  repo: github.com/edisonomics/SAND
- name: NMRBox
  role: Cloud environment where SAND runs and where the simulated test-example datasets
    are executed for the benchmark
- name: CRAFT
  role: Baseline deconvolution method providing comparison quantifications on the
    same simulated spectra
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:8f70283d14271bdfc2c712d5f2d889904689f49125a11f85a100144fc7e71de2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CRAFT
  - NMRBox
  - SAND
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# simulated-spectra-ground-truth-comparison

## Summary

Benchmarks SAND's metabolite quantification against peak-intensity, numerical-integral, and CRAFT baselines on simulated 1D NMR datasets with known ground-truth concentrations, scoring each method by MSE, relative MSE, slope k, and Pearson correlation.

## When to use

Use when evaluating quantification accuracy of NMR spectral deconvolution methods on simulated spectra where ground-truth metabolite concentrations are known and methods can be regressed against truth.

## When NOT to use

- Input is experimental (e.g., urine or worm) data with no known ground-truth concentrations — the regression-against-truth metrics cannot be computed
- Spectra are not high-resolution metabolomics 1D data; the current SAND version is tuned only for high-resolution metabolomics spectra
- The goal is peak annotation/clustering rather than quantification accuracy benchmarking

## Inputs

- Simulated 1D NMR spectra (high-resolution metabolomics test-example datasets) with known ground-truth metabolite concentrations
- Baseline quantifications from peak intensity, numerical integration, and CRAFT deconvolution on the same simulated spectra
- SAND quantification estimates from the hybrid MCMC time-domain decomposition

## Outputs

- quantification_metrics_comparison.csv with per-method, per-dataset MSE, relative MSE, slope k, and Pearson correlation
- Regression fits of each method's estimates against ground-truth concentrations
- Identification of the datasets where CRAFT slightly outperforms SAND

## How to apply

Run the SAND pipeline (hybrid Markov chain Monte Carlo time-domain decomposition with subsampling in time and frequency domains) on the simulated test-example datasets to obtain metabolite quantification estimates. Produce baseline quantifications from the same simulated spectra using peak intensity, numerical integration, and CRAFT deconvolution. For each method and each simulated dataset, regress the estimates on the ground-truth concentrations and compute MSE, relative MSE, slope k, and Pearson correlation. On the more complex simulated datasets, verify SAND reaches a correlation of approximately 0.88 with slope k of approximately 0.9 against ground truth, and note the datasets where CRAFT performs slightly better. Write all per-method, per-dataset metrics to quantification_metrics_comparison.csv.

## Related tools

- **SAND** (Produces the quantification estimates under test via hybrid MCMC time-domain decomposition of the simulated spectra) — github.com/edisonomics/SAND
- **NMRBox** (Cloud environment where SAND runs and where the simulated test-example datasets are executed for the benchmark)
- **CRAFT** (Baseline deconvolution method providing comparison quantifications on the same simulated spectra)

## Evaluation signals

- SAND achieves Pearson correlation of approximately 0.88 against ground truth on the more complex simulated datasets
- SAND's regression slope k against ground truth is approximately 0.9
- quantification_metrics_comparison.csv contains metrics for all four methods (SAND, peak intensity, numerical integral, CRAFT) across every simulated dataset
- MSE and relative MSE are computed for every method-dataset pair from regression on ground-truth concentrations
- The two datasets where CRAFT is slightly better than SAND are identified and reported

## Limitations

- Ground-truth-validated performance is demonstrated only on simulated datasets, a two-compound mixture, and a spiked urine sample; correlation is ~0.9, not perfect
- CRAFT slightly outperforms SAND on some of the simulated datasets
- The SAND implementation used is a slow MATLAB version, not yet parallelizable in C/C++, which constrains benchmark throughput

## Evidence
