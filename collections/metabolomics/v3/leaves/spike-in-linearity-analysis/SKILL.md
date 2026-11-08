---
name: spike-in-linearity-analysis
description: Use when a 1D NMR dilution series (e.g., urine spectra spiked with 0,
  20, 40, 60, 80 μL of a five-compound mixture) is available and you must verify that
  automated peak-quantification output scales linearly with spike amount.
when_to_use_negative:
- Input is already a peak/feature table from a prior quantification step rather than
  raw 1D NMR time-domain data that SAND must deconvolve
- The spectra are not a quantitative dilution/spike series, so no volume variable
  exists to correlate amplitudes against
- Data are low-resolution or non-metabolomics spectra; SAND's current version is tuned
  for high-resolution metabolomics spectra
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: SAND
  role: Deconvolves each 1D NMR spectrum in the spike-in series into peak tables via
    hybrid Markov chain Monte Carlo optimization; the resulting peak amplitudes are
    the quantities correlated against spike-in volume
  repo: https://github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud environment where SAND is run (via NMRPipe resources) to process the
    series of spectra reproducibly
- name: NMRPipe
  role: Interface through which SAND is invoked on NMRbox to deconvolute the spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:933eba43e7fccf00a0093c03a3381074fef96b13dbbcb6ffd2d0ce308f9fa2ea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spike-in-linearity-analysis

## Summary

Validates automated NMR metabolite quantification (e.g., SAND peak amplitudes) by regressing quantified spike-in peak amplitudes against spike-in volume across a dilution series, checking correlation, matrix-peak stability, and nonzero intercepts.

## When to use

Use when a 1D NMR dilution series (e.g., urine spectra spiked with 0, 20, 40, 60, 80 μL of a five-compound mixture) is available and you must verify that automated peak-quantification output scales linearly with spike amount.

## When NOT to use

- Input is already a peak/feature table from a prior quantification step rather than raw 1D NMR time-domain data that SAND must deconvolve
- The spectra are not a quantitative dilution/spike series, so no volume variable exists to correlate amplitudes against
- Data are low-resolution or non-metabolomics spectra; SAND's current version is tuned for high-resolution metabolomics spectra

## Inputs

- Series of 1D 1H NMR spectra (FID/time-domain data) from a spike-in dilution series, e.g., Metabolomics Workbench study ST002718 urine spectra spiked with 0, 20, 40, 60, and 80 μL of a five-compound mixture
- List of spiked compounds and their expected peak positions/relative peak intensities
- List of matrix-only (urine) peaks to monitor for volume-independent stability

## Outputs

- Per-compound peak amplitude table from SAND deconvolution for each spectrum
- Pearson correlation of amplitude vs. spike-in volume per spiked compound
- Linear regression slope and Y intercept (e.g., glucose amplitude vs. volume, nonzero intercept from endogenous glucose)
- spike_in_linearity_report.csv containing all correlation, slope, and intercept values

## How to apply

Retrieve the series of 1D NMR spectra from the Metabolomics Workbench (e.g., study ST002718 spike-in urine) and deconvolve each spectrum into a peak table with SAND, which uses hybrid Markov chain Monte Carlo optimization with subsampling in both time and frequency domains and a random training/validation split of the time-domain data to avoid overfitting. For each spiked compound, compute the Pearson correlation between SAND-quantified peak amplitude and spike-in volume, expecting a correlation close to 1 (SAND reported ~0.9 correlation with ground truth on a spiked urine sample). Verify that urine-matrix-only peaks remain flat across the 0–80 μL series, i.e., no systematic trend with volume. Finally, fit a linear regression of a compound's peak amplitude versus spike-in volume and confirm the Y intercept is clearly nonzero when that compound is endogenously present (e.g., glucose in unspiked urine). Write all correlation, slope, and intercept values to spike_in_linearity_report.csv.

## Related tools

- **SAND** (Deconvolves each 1D NMR spectrum in the spike-in series into peak tables via hybrid Markov chain Monte Carlo optimization; the resulting peak amplitudes are the quantities correlated against spike-in volume) — https://github.com/edisonomics/SAND
- **NMRbox** (Cloud environment where SAND is run (via NMRPipe resources) to process the series of spectra reproducibly)
- **NMRPipe** (Interface through which SAND is invoked on NMRbox to deconvolute the spectra)

## Examples

```
sand deconvolute ST002718_spectrum_*.fid --output spike_in_peak_tables/ && python spike_in_linearity.py --peak-tables spike_in_peak_tables/ --volumes 0,20,40,60,80 --report spike_in_linearity_report.csv
```

## Evaluation signals

- Pearson correlation between SAND-quantified spike-in peak amplitude and spike-in volume is close to 1 for each of the five spiked compounds
- Slope of amplitude vs. volume is defined by the compound's relative peak intensity
- Urine-only (matrix) peak amplitudes show no systematic trend across the 0–80 μL volume series
- Linear regression of glucose peak amplitude vs. spike-in volume yields a clearly nonzero Y intercept reflecting endogenous glucose in the unspiked urine
- Quantified amplitude-vs-truth correlation falls near the reported ~0.9 for spiked urine samples

## Limitations

- SAND provides correlation of ~0.9 with ground truth, not perfect agreement, on overlapped/spiked datasets
- The current SAND version is tuned for high-resolution metabolomics spectra and implemented in slow MATLAB, not yet parallelized in C/C++
- Autophasing or manually adjusted phase values are preferred pre-processing but not always needed; reliability of unphased input is unverified

## Evidence
