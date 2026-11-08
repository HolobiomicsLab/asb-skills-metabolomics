---
name: regression-slope-correlation-analysis
description: Use when quantification estimates (e.g. deconvoluted 1D NMR peak tables)
  must be judged against known ground-truth concentrations on simulated or spiked
  datasets, using slope k and Pearson correlation as accuracy metrics.
when_to_use_negative:
- No ground-truth concentrations exist (experimental-only data without simulated or
  spiked references), so regression against truth is not possible
- Input is still raw FID/spectral data rather than per-metabolite quantification estimates
  produced by SAND, peak intensity, numerical integration, or CRAFT
- Spectra are not high-resolution metabolomics 1D NMR data, since the current SAND
  version is tuned only for high-resolution metabolomics spectra
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SAND
  role: Produces the metabolite quantification estimates (via hybrid MCMC time-domain
    decomposition) that are regressed against ground-truth concentrations
  repo: https://github.com/edisonomics/SAND
- name: NMRBox
  role: Cloud environment where SAND is run on the simulated test-example datasets
    for the benchmark comparison
  repo: https://nmrbox.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:3cb85d3faf71a88cd007d861fd877753fbce9065e0f2dcea4163a062d36ffe2a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - NMRBox
  - SAND
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# regression-slope-correlation-analysis

## Summary

Validates metabolite quantification methods by regressing their estimates against known ground-truth concentrations and reporting MSE, relative MSE, slope k, and Pearson correlation, as used to benchmark SAND against peak-intensity, numerical-integral, and CRAFT baselines on simulated NMR data.

## When to use

Use when quantification estimates (e.g. deconvoluted 1D NMR peak tables) must be judged against known ground-truth concentrations on simulated or spiked datasets, using slope k and Pearson correlation as accuracy metrics.

## When NOT to use

- No ground-truth concentrations exist (experimental-only data without simulated or spiked references), so regression against truth is not possible
- Input is still raw FID/spectral data rather than per-metabolite quantification estimates produced by SAND, peak intensity, numerical integration, or CRAFT
- Spectra are not high-resolution metabolomics 1D NMR data, since the current SAND version is tuned only for high-resolution metabolomics spectra

## Inputs

- Simulated 1D NMR spectra with known ground-truth metabolite concentrations (test-example datasets)
- Quantification estimates (peak tables) from SAND via hybrid MCMC time-domain decomposition
- Baseline quantifications from peak intensity, numerical integration, and CRAFT deconvolution on the same spectra

## Outputs

- quantification_metrics_comparison.csv containing per-method, per-dataset MSE, relative MSE, slope k, and Pearson correlation

## How to apply

Run the SAND pipeline (hybrid Markov chain Monte Carlo time-domain decomposition with subsampling in time and frequency domains) on simulated test-example spectra to obtain metabolite quantification estimates. Obtain baseline quantifications from the same simulated spectra using peak intensity, numerical integration, and CRAFT deconvolution so all methods are scored on identical inputs. For each method and each simulated dataset, regress the estimates on the ground-truth concentrations and compute MSE, relative MSE, slope k, and Pearson correlation. Judge SAND's result against the reported benchmark — a correlation of approximately 0.88 and slope k of around 0.9 on the more complex simulated datasets — and identify any datasets where a baseline (e.g. CRAFT) is slightly better. Write all per-method, per-dataset metrics to quantification_metrics_comparison.csv for downstream comparison.

## Related tools

- **SAND** (Produces the metabolite quantification estimates (via hybrid MCMC time-domain decomposition) that are regressed against ground-truth concentrations) — https://github.com/edisonomics/SAND
- **NMRBox** (Cloud environment where SAND is run on the simulated test-example datasets for the benchmark comparison) — https://nmrbox.org

## Examples

```
python -c "from scipy.stats import linregress, pearsonr; lr = linregress(truth, est); r = pearsonr(est, truth)[0]; print(lr.slope, r)"
```

## Evaluation signals

- SAND's quantification estimates reach a Pearson correlation of approximately 0.88 with ground truth on the more complex simulated datasets
- SAND's regression slope k is approximately 0.9 against ground truth on the complex simulated datasets
- Metrics are computed per method and per dataset, covering SAND, peak intensity, numerical integration, and CRAFT
- The two datasets where CRAFT slightly outperforms SAND are identifiable from the comparison table
- All per-method, per-dataset metrics are present in quantification_metrics_comparison.csv

## Limitations

- Requires datasets with known ground-truth concentrations (simulated or spiked samples); not applicable to unvalidated experimental data
- Even SAND's best reported benchmark leaves residual error (correlation ~0.88, slope ~0.9, not perfect agreement)
- Underlying quantification depends on SAND, which is currently a slow MATLAB implementation not yet parallelizable, and is tuned only for high-resolution metabolomics spectra

## Evidence
