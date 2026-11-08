---
name: peak-recovery-quantification
description: 'Use when evaluating automated annotation of metabolomics NMR spectra:
  simulated mixtures with known ground-truth peak assignments are available and you
  need a per-compound recovery ratio for clustered SAND-decomposed peaks.'
when_to_use_negative:
- Input spectra lack ground-truth compound/peak assignments, so per-compound recovery
  cannot be computed against known clusters
- Spectra are not high-resolution metabolomics 1D NMR data; SAND is currently tuned
  only for high-resolution metabolomics spectra
- Input is already a feature table or peak list rather than raw/free-induction-decay
  NMR data requiring SAND decomposition
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: SAND
  role: Decomposes each 1D NMR spectrum into peak tables via hybrid Markov chain Monte
    Carlo optimization with time- and frequency-domain subsampling, providing the
    peaks fed into the correlation network
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Language of the current SAND implementation used to run the decomposition
    on the simulated mixture panel
- name: NMRbox
  role: Cloud environment where SAND is installed (install_SAND.sh) and from which
    the decomposition of the simulated spectra can be run via a single terminal command
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:df98b884821a6198f480f22486aba161418772ca9c8688fd04fe9a1a036e13f3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# peak-recovery-quantification

## Summary

Quantifies the fraction of ground-truth peaks per compound that are automatically recovered in a single cluster when SAND-decomposed NMR peaks are annotated via correlation networks, benchmarked against the reported mean recovery ratio of ~74%.

## When to use

Use when evaluating automated annotation of metabolomics NMR spectra: simulated mixtures with known ground-truth peak assignments are available and you need a per-compound recovery ratio for clustered SAND-decomposed peaks.

## When NOT to use

- Input spectra lack ground-truth compound/peak assignments, so per-compound recovery cannot be computed against known clusters
- Spectra are not high-resolution metabolomics 1D NMR data; SAND is currently tuned only for high-resolution metabolomics spectra
- Input is already a feature table or peak list rather than raw/free-induction-decay NMR data requiring SAND decomposition

## Inputs

- Simulated 1D NMR spectra from predefined compound groups (e.g. 50 spectra, 18 compound groups) with ground-truth peak assignments
- SAND peak tables decomposed from each spectrum
- Ground-truth compound-to-peak mapping

## Outputs

- Per-compound peak recovery ratios (peak_recovery_ratios.csv)
- Mean recovery ratio across compounds (expected ~74%)

## How to apply

Generate a panel of simulated 1D NMR spectra (e.g. 50 spectra from 18 predefined compound groups) holding within-group relative amplitudes fixed while sampling each compound's concentration down to ~20% of its maximum. Decompose every spectrum with SAND's hybrid Markov chain Monte Carlo optimization, which subsamples in both time and frequency domains. Compute pairwise correlations of every detected peak across the 50 samples and build the peak-correlation network at the thresholds stated in the paper, then partition the network with community detection and assign clusters to ground-truth compounds. For each compound, divide the number of its ground-truth peaks landing in a single cluster by its total ground-truth peak count and write the per-compound recovery ratios to peak_recovery_ratios.csv. Validate by checking the mean recovery ratio across compounds against the reported ~74%.

## Related tools

- **SAND** (Decomposes each 1D NMR spectrum into peak tables via hybrid Markov chain Monte Carlo optimization with time- and frequency-domain subsampling, providing the peaks fed into the correlation network) — https://github.com/edisonomics/SAND
- **MATLAB** (Language of the current SAND implementation used to run the decomposition on the simulated mixture panel)
- **NMRbox** (Cloud environment where SAND is installed (install_SAND.sh) and from which the decomposition of the simulated spectra can be run via a single terminal command)

## Evaluation signals

- Mean recovery ratio across the 18 compounds reproduces the reported ~74% (>60% of ground-truth peaks in a single cluster for most compounds)
- peak_recovery_ratios.csv contains one recovery ratio per ground-truth compound
- Each spectrum yields a SAND peak table with no failed decompositions across the 50-sample panel
- Correlation-network clusters partition such that each assigned cluster maps to a single ground-truth compound
- Concentration sampling per compound spans its maximum down to ~20% across the 50 simulated spectra

## Limitations

- SAND is tuned only for high-resolution metabolomics spectra; other data types are not yet accommodated
- SAND is not yet adapted to multidimensional applications or nonexponential signals
- Only ~74% of peaks per compound on average are recovered in a single cluster, so a substantial fraction of ground-truth peaks may be missed or split across clusters

## Evidence
