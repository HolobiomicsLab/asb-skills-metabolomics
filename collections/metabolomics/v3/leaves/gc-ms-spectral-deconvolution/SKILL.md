---
name: gc-ms-spectral-deconvolution
description: Use when raw GC-MS/EI datasets (e.g. a deposited MassIVE dataset such
  as MSV000084033) contain co-eluting compound fragments that must be separated into
  deconvoluted spectra before molecular networking or repository-scale analysis.
when_to_use_negative:
- Input is already a deconvoluted feature table or .mgf consensus spectra — deconvolution
  has already been performed and re-running MSHub would be redundant
- Data are LC-MS/MS MS2 spectra intended for classical (non-feature-based) molecular
  networking; MS2 motif/chemical-class annotation tooling such as MS2LDA/pyMolNetEnhancer
  targets MS2 networks, not EI GC-MS deconvolution
- The goal is manual, single-injection EI deconvolution of a few chromatograms rather
  than repository-scale processing across all samples in a dataset
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSHub
  role: Machine-learning engine that performs auto-deconvolution of GC-MS compound
    fragmentation patterns via unsupervised non-negative matrix factorization
  repo: https://github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform hosting the MSHub feature-based molecular networking workflow used
    to run the deconvolution and export .mgf and quantification artifacts
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: pyMolNetEnhancer
  role: Optional downstream module for integrating chemical class and substructure
    information into the resulting molecular networks
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:1b9afe8f3fa4adf0c67e0bfc8e05c8927062600b73d4c47c5324ce9d2148c1bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GNPS_GC
  tools:
  - GNPS
  - MSHub
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gc-ms-spectral-deconvolution

## Summary

Auto-deconvolution of GC-MS (electron ionization) fragmentation data using MSHub on the GNPS/MassIVE platform, which factors overlapping compound spectra via unsupervised non-negative matrix factorization (NNMF) and produces deconvoluted consensus spectra plus a quantification table suitable for repository-scale molecular networking.

## When to use

Use when raw GC-MS/EI datasets (e.g. a deposited MassIVE dataset such as MSV000084033) contain co-eluting compound fragments that must be separated into deconvoluted spectra before molecular networking or repository-scale analysis.

## When NOT to use

- Input is already a deconvoluted feature table or .mgf consensus spectra — deconvolution has already been performed and re-running MSHub would be redundant
- Data are LC-MS/MS MS2 spectra intended for classical (non-feature-based) molecular networking; MS2 motif/chemical-class annotation tooling such as MS2LDA/pyMolNetEnhancer targets MS2 networks, not EI GC-MS deconvolution
- The goal is manual, single-injection EI deconvolution of a few chromatograms rather than repository-scale processing across all samples in a dataset

## Inputs

- Deposited GC-MS dataset from the GNPS/MassIVE repository (e.g. MSV000084033)
- GC-MS electron ionization raw runs with co-eluting, overlapping compound fragmentation patterns

## Outputs

- Deconvoluted compound fragmentation spectra (.mgf spectra summary file)
- Accompanying quantification table of deconvoluted features across samples
- Feature-based molecular network of GC-MS data

## How to apply

Retrieve the deposited GC-MS dataset from the GNPS/MassIVE repository and launch the MSHub feature-based molecular networking workflow on GNPS, which performs auto-deconvolution of compound fragmentation patterns via unsupervised non-negative matrix factorization. Because naive NNMF deconvolution does not scale to repository-sized datasets, rely on the workflow's built-in Fast Fourier Transform-based strategy for scalability. The workflow quantifies the reproducibility of fragmentation patterns across all samples using the 'balance score' and uses it to group consensus spectra into deconvoluted compound features. Finally, export the workflow result artifacts — the .mgf spectra summary file and the accompanying quantification table — and validate that both are produced and non-empty and that the deconvolved spectra/feature rows correspond to the input sample set as reported by the GNPS job summary.

## Related tools

- **MSHub** (Machine-learning engine that performs auto-deconvolution of GC-MS compound fragmentation patterns via unsupervised non-negative matrix factorization) — https://github.com/bittremieux/GNPS_GC
- **GNPS** (Platform hosting the MSHub feature-based molecular networking workflow used to run the deconvolution and export .mgf and quantification artifacts) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **pyMolNetEnhancer** (Optional downstream module for integrating chemical class and substructure information into the resulting molecular networks) — https://github.com/madeleineernst/pyMolNetEnhancer

## Evaluation signals

- Both result artifacts are produced and non-empty: the .mgf spectra summary file and the accompanying quantification table
- The deconvolved spectra/feature rows correspond to the input sample set as reported by the MSHub/GNPS job summary
- Consensus spectra are grouped into deconvolved compound features using the balance score, i.e. reproducibility of fragmentation patterns across all samples is quantified
- The job completes on the full dataset, confirming the FFT-based scalability strategy handled the NNMF scaling limitation
- Downstream molecular network can be built from the .mgf artifact and optionally annotated with chemical classes via pyMolNetEnhancer

## Limitations

- Naive NNMF deconvolution does not scale to full deposited datasets; the FFT-based strategy is required to overcome this scalability limitation
- The companion GNPS_Workflows repository carries no asserted license (NOASSERTION), so reuse/redistribution of workflow definitions may be restricted
- The bittremieux/GNPS_GC companion repository is archived

## Evidence

- [readme] Companion repository for MSHub analysis: [evidence span withheld — anchor status: not_found]
