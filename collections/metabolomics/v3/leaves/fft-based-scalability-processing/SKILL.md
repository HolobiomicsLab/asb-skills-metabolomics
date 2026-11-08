---
name: fft-based-scalability-processing
description: Use when a deposited GC-MS dataset is too large for direct NNMF auto-deconvolution
  in MSHub, and the analysis goal is deconvoluted fragmentation spectra at repository
  scale for molecular networking.
when_to_use_negative:
- The dataset is small enough that direct NNMF deconvolution completes without hitting
  scalability limits, so the FFT-based strategy is unnecessary overhead.
- The input is LC-MS/MS data handled by standard feature-based molecular networking
  rather than GC-MS electron-impact data requiring spectral deconvolution.
- Spectra are already deconvoluted compound features (e.g., an existing .mgf plus
  quantification table from a prior MSHub job).
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MSHub
  role: Machine-learning engine performing auto-deconvolution of GC-MS fragmentation
    patterns via unsupervised NNMF, with the FFT-based strategy for scalability
  repo: https://github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform hosting the MSHub feature-based molecular networking workflow; exports
    .mgf spectra summary and quantification table artifacts
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:5c20505511fe206fb56053e3292860d8491cf6c83113a09710b19727ad6596c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GNPS_GC
  tools:
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# FFT-based scalability processing

## Summary

A Fast Fourier Transform-based strategy built into the GNPS MSHub workflow that overcomes the scalability limitations of unsupervised non-negative matrix factorization (NNMF) when auto-deconvolving GC-MS fragmentation patterns across repository-scale datasets.

## When to use

Use when a deposited GC-MS dataset is too large for direct NNMF auto-deconvolution in MSHub, and the analysis goal is deconvoluted fragmentation spectra at repository scale for molecular networking.

## When NOT to use

- The dataset is small enough that direct NNMF deconvolution completes without hitting scalability limits, so the FFT-based strategy is unnecessary overhead.
- The input is LC-MS/MS data handled by standard feature-based molecular networking rather than GC-MS electron-impact data requiring spectral deconvolution.
- Spectra are already deconvoluted compound features (e.g., an existing .mgf plus quantification table from a prior MSHub job).

## Inputs

- Deposited GC-MS dataset from the GNPS/MassIVE repository (e.g., MSV000084033)
- Raw GC-MS chromatogram/spectra files from the input sample set

## Outputs

- .mgf spectra summary file of deconvoluted compound fragmentation spectra
- Accompanying quantification table mapping deconvolved features to input samples

## How to apply

Retrieve the deposited GC-MS dataset (e.g., MSV000084033 from MassIVE) and launch the MSHub feature-based molecular networking workflow on GNPS, which auto-deconvolves compound fragmentation patterns via unsupervised non-negative matrix factorization. Because full-dataset NNMF deconvolution does not scale, apply the FFT-based strategy built into the workflow to make the decomposition tractable on the complete sample set. The workflow then quantifies reproducibility of fragmentation patterns across all samples using the 'balance score' to group consensus spectra into deconvolved compound features. Finally, export the .mgf spectra summary file and the accompanying quantification table and verify that deconvolved feature rows correspond to the input sample set per the GNPS job summary.

## Related tools

- **MSHub** (Machine-learning engine performing auto-deconvolution of GC-MS fragmentation patterns via unsupervised NNMF, with the FFT-based strategy for scalability) — https://github.com/bittremieux/GNPS_GC
- **GNPS** (Platform hosting the MSHub feature-based molecular networking workflow; exports .mgf spectra summary and quantification table artifacts) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Evaluation signals

- Both workflow artifacts (the .mgf spectra summary file and the quantification table) are produced and non-empty.
- Deconvolved spectra/feature rows correspond to the input sample set as reported by the MSHub/GNPS job summary.
- Consensus spectra are grouped into deconvolved compound features via the balance score quantifying fragmentation-pattern reproducibility across all samples.
- The deconvolution completed on the full dataset (FFT strategy applied), i.e., the job did not fail or truncate on scalability grounds.

## Limitations

- Unsupervised NNMF deconvolution has scalability limitations on large GC-MS datasets, which is the specific problem the FFT-based strategy addresses.
- The companion GNPS_GC repository provides no changelog or detailed parameter documentation beyond manuscript citation.

## Evidence

- [readme] Companion repository: [evidence span withheld — anchor status: not_found]
