---
name: metabolite-peak-amplitude-quantification
description: Use when raw 1D NMR spectra (e.g., spiked urine FIDs at 0-80 uL spike-in
  volumes) must be deconvolved into peak tables so metabolite peak amplitudes can
  be correlated with concentration or volume series.
when_to_use_negative:
- Input is already a peak/feature table and only needs statistical analysis, not spectral
  deconvolution
- Spectra are multidimensional or nonuniformly sampled — SAND's time-domain subsampling
  gives it potential to be extended to higher dimensionality, but this is not yet
  realized
- Signals are strongly nonexponential — SAND does not yet handle nonexponential signals
  natively and is tuned for high-resolution metabolomics spectra
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: SAND
  role: Core tool that automatically processes and deconvolutes 1D NMR spectra into
    peak tables via MCMC hybrid optimization with time/frequency-domain subsampling
  repo: github.com/edisonomics/SAND
- name: NMRbox
  role: Cloud computing environment where SAND is hosted and executed for the quantification
    workflow
  repo: https://nmrbox.org
- name: NMRPipe
  role: Interface through which SAND can be used on the NMRbox resources, initiated
    via a single command from a terminal window or shell script
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:60a11d1a11266e153ea07a2f0f939aab46e166645f39b04b0bb90859d599a550
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NMRPipe
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolite-peak-amplitude-quantification

## Summary

Quantifies metabolites in 1D NMR spectra by deconvolving free induction decay (FID) data into peak tables using SAND's hybrid Markov chain Monte Carlo optimization with time- and frequency-domain subsampling, enabling amplitude-based linear-response analysis on spiked and unspiked samples.

## When to use

Use when raw 1D NMR spectra (e.g., spiked urine FIDs at 0-80 uL spike-in volumes) must be deconvolved into peak tables so metabolite peak amplitudes can be correlated with concentration or volume series.

## When NOT to use

- Input is already a peak/feature table and only needs statistical analysis, not spectral deconvolution
- Spectra are multidimensional or nonuniformly sampled — SAND's time-domain subsampling gives it potential to be extended to higher dimensionality, but this is not yet realized
- Signals are strongly nonexponential — SAND does not yet handle nonexponential signals natively and is tuned for high-resolution metabolomics spectra

## Inputs

- Raw 1D NMR spectra / FID time-domain data (e.g., human urine spectra spiked with 0, 20, 40, 60, 80 uL of a five-compound mixture)
- Spike-in volume series or concentration metadata for correlation/regression analysis

## Outputs

- Peak tables (deconvolved peak lists with amplitudes per spectrum)
- Pearson correlation values of peak amplitude vs. spike-in volume per compound
- Linear regression fits (slope, nonzero Y intercept) of amplitude vs. volume
- Quantification report (e.g., spike_in_linearity_report.csv)

## How to apply

Retrieve the 1D NMR spectra (e.g., the five spike-in human urine spectra from Metabolomics Workbench ST002718 at 0, 20, 40, 60, and 80 uL of a five-compound mixture) and run SAND on each to decompose them into peak tables. SAND uses hybrid optimization with Markov chain Monte Carlo methods, employing subsampling in both time and frequency domains, and randomly divides the time-domain data into training and validation sets to help avoid overfitting. Optionally pre-process with autophasing or manually adjusted phase values (preferred but not always needed), and note that peaks identified by SAND undergo a second round of optimization using an exponential + Gaussian model. For each compound of interest, compute Pearson correlation between SAND-quantified peak amplitude and spike-in volume, expecting correlation close to 1 (SAND reports ~0.9 correlation with ground truth on spiked urine); verify matrix-only peaks stay constant across the volume series. Fit linear regression of a compound like glucose amplitude versus volume and confirm a nonzero Y intercept reflecting endogenous glucose in the unspiked urine; write results (e.g., to spike_in_linearity_report.csv).

## Related tools

- **SAND** (Core tool that automatically processes and deconvolutes 1D NMR spectra into peak tables via MCMC hybrid optimization with time/frequency-domain subsampling) — github.com/edisonomics/SAND
- **NMRbox** (Cloud computing environment where SAND is hosted and executed for the quantification workflow) — https://nmrbox.org
- **NMRPipe** (Interface through which SAND can be used on the NMRbox resources, initiated via a single command from a terminal window or shell script)

## Examples

```
SAND --fid spike_in_urine_40uL.fid --out peak_table.csv  # run on NMRbox via NMRPipe from a terminal/shell script
```

## Evaluation signals

- Spike-in compound amplitude-vs-volume Pearson correlations should be close to 1 (SAND reports ~0.9 correlation with ground truth on spiked urine)
- Peak amplitudes of compounds present only in the urine matrix should remain constant across the 0-80 uL volume series with no systematic trend
- Linear regression of a compound present in the unspiked matrix (e.g., glucose) should yield a clearly nonzero Y intercept
- Training/validation split of time-domain data should be applied (random division) to confirm no overfitting
- Peak table should recover overlapped peaks — on average 74% of peaks per compound recoverable in single clusters via automated annotation with correlation networks

## Limitations

- SAND's correlation with ground truth is ~0.9, not perfect, on highly overlapped simulated data, two-compound mixtures, and spiked urine
- Currently implemented in slow MATLAB; a faster parallelizable C/C++ version is intended but not yet available
- Tuned only for high-resolution metabolomics spectra; other cases not yet accommodated
- Does not yet handle nonexponential signals natively or multidimensional applications
- Initial pre-processing by the user using either autophasing or manually adjusted phase values is preferred but not always needed — reliability of unphased input is unverified

## Evidence
