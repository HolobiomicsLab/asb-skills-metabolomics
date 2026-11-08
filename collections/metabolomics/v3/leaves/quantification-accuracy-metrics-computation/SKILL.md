---
name: quantification-accuracy-metrics-computation
description: Use when fitted peak/concentration tables from automated NMR quantification
  (e.g. SAND output on simulated or spiked-mixture spectra) must be scored against
  ground-truth concentrations using Pearson correlation and regression-slope metrics.
when_to_use_negative:
- Input spectra are not high-resolution metabolomics 1D NMR data — the article states
  the current SAND version is tuned for high-resolution metabolomics spectra
- No ground-truth or known-composition reference exists, so correlation/slope against
  truth cannot be computed
- You need quantified peak feature tables rather than an accuracy assessment of existing
  ones
edam_operation: http://edamontology.org/operation_2447
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: SAND
  role: Produces the fitted peak tables whose estimated concentrations are scored
    against ground truth via correlation and slope
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud environment where SAND is hosted and analysis runs are initiated via
    a single terminal command
  repo: https://nmrbox.org
- name: MATLAB
  role: Implementation language of the SAND version used to generate quantification
    results for benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:5381fcef5102bb30714ebc804a28186d036148cbd677baefd92a891362513b1d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - C/C++
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_2447
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# quantification-accuracy-metrics-computation

## Summary

Benchmark metabolite quantification accuracy by correlating SAND-estimated concentrations against known ground truth (Pearson r, linear-regression slope k), as demonstrated on highly overlapped simulated 1D NMR spectra with ~100 peaks and on spiked urine samples.

## When to use

Use when fitted peak/concentration tables from automated NMR quantification (e.g. SAND output on simulated or spiked-mixture spectra) must be scored against ground-truth concentrations using Pearson correlation and regression-slope metrics.

## When NOT to use

- Input spectra are not high-resolution metabolomics 1D NMR data — the article states the current SAND version is tuned for high-resolution metabolomics spectra
- No ground-truth or known-composition reference exists, so correlation/slope against truth cannot be computed
- You need quantified peak feature tables rather than an accuracy assessment of existing ones

## Inputs

- SAND peak/concentration tables from 1D NMR spectra (simulated benchmark, two-compound mixture, or spiked urine)
- Ground-truth concentration table keyed by compound of origin (from simulated spectra or known spiking amounts)

## Outputs

- Pearson correlation between estimated and ground-truth concentrations
- Linear-regression slope (k) of estimated vs. ground-truth concentrations
- Per-compound error metrics and peak-splitting observations (e.g. around 7.3 and 7.95 ppm)

## How to apply

Generate or obtain spectra with known ground-truth concentrations — e.g. simulate ~100 peaks over a 1.50 ppm region with randomly sampled concentrations and exponentially damped sinusoids with lambda in the range 1-15. Run the quantification workflow (for SAND: hybrid Markov chain Monte Carlo optimization with random training/validation subsampling of the time-domain data) and extract its peak tables, mapping each fitted peak back to its compound of origin. Compute the Pearson correlation between estimated and true concentrations and the slope (k) of a linear regression of estimated vs. ground truth; SAND achieves a correlation of ~0.9 (reported 0.88) and a slope near 0.9 on highly overlapped simulated data sets, with the same validation applied to a two-compound mixture and a spiked urine sample. Flag runs falling below the ~0.9 correlation threshold as out-of-spec, and record qualitative behaviors such as peak splitting around 7.3 and 7.95 ppm in the simulated benchmark.

## Related tools

- **SAND** (Produces the fitted peak tables whose estimated concentrations are scored against ground truth via correlation and slope) — https://github.com/edisonomics/SAND
- **NMRbox** (Cloud environment where SAND is hosted and analysis runs are initiated via a single terminal command) — https://nmrbox.org
- **MATLAB** (Implementation language of the SAND version used to generate quantification results for benchmarking)

## Examples

```
sand_fit -in spectrum.fid -out peaks.csv  # then in Python: r, k = pearsonr(sand_conc, truth_conc), linregress(truth_conc, sand_conc).slope
```

## Evaluation signals

- Pearson correlation between SAND-estimated and ground-truth concentrations reaches ~0.9 on the highly overlapped simulated benchmark
- Linear-regression slope (k) of estimated vs. true concentrations is around 0.9
- Flag any benchmark run whose correlation drops below the ~0.9 threshold as out-of-spec
- Per-compound recovery behaves consistently, consistent with the reported ~74% of peaks per compound recoverable in single clusters via correlation-network annotation
- Peak-splitting artifacts around 7.3 and 7.95 ppm are observed and logged in the simulated benchmark, matching the reported behavior

## Limitations

- Accuracy was validated only on highly overlapped simulated data, a two-compound mixture, and one spiked urine sample; generalization to other matrices is untested
- The benchmark correlation reported for the simulated sets was 0.88, slightly below the ~0.9 headline figure
- SAND's current MATLAB implementation is slow and not parallelizable, which constrains benchmark throughput
- The accuracy figures depend on correct phasing — autophasing or manually adjusted phase values are preferred but not always needed, and unphased input reliability is unverified

## Evidence

- [results] On the complex simulated benchmark data sets, SAND outperformed other automatic quantification methods, achieving a Pearson correlation of 0.88: [evidence span withheld — anchor status: not_found]
