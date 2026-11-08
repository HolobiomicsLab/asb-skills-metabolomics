---
name: nnmf-fragmentation-factorization
description: Use when deposited GC-MS datasets contain co-eluting compounds whose
  fragmentation spectra must be auto-deconvoluted (e.g. breathomics or volatilome
  data) before repository-scale GNPS molecular networking analysis.
when_to_use_negative:
- Input is already deconvoluted (e.g. a feature table or deconvoluted .mgf from another
  tool) rather than raw GC-MS data
- Data are LC-MS/MS to be processed with the classical or feature-based molecular
  networking workflows (e.g. via MZmine and pyMolNetEnhancer) rather than GC-MS auto-deconvolution
- Only a single-sample chromatogram is available, so no cross-sample balance score
  can be computed
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MSHub
  role: Machine learning (NNMF-based) approach that performs auto-deconvolution of
    GC-MS compound fragmentation patterns on the GNPS platform
  repo: https://github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Web platform hosting the MSHub feature-based molecular networking workflow,
    used to store, process, share, and perform molecular networking of GC-MS data
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:c67353ffb25da70fa28d2d7da98f3e6a6b3c7025d9484b9709bacbc97c44ecf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GNPS_GC
  tools:
  - GNPS
  - MSHub
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nnmf-fragmentation-factorization

## Summary

This skill applies the MSHub auto-deconvolution strategy on the GNPS platform, which factorizes GC-MS compound fragmentation patterns via unsupervised non-negative matrix factorization (NNMF) and groups consensus spectra into deconvolved compound features using a 'balance score' of fragmentation-pattern reproducibility.

## When to use

Use when deposited GC-MS datasets contain co-eluting compounds whose fragmentation spectra must be auto-deconvoluted (e.g. breathomics or volatilome data) before repository-scale GNPS molecular networking analysis.

## When NOT to use

- Input is already deconvoluted (e.g. a feature table or deconvoluted .mgf from another tool) rather than raw GC-MS data
- Data are LC-MS/MS to be processed with the classical or feature-based molecular networking workflows (e.g. via MZmine and pyMolNetEnhancer) rather than GC-MS auto-deconvolution
- Only a single-sample chromatogram is available, so no cross-sample balance score can be computed

## Inputs

- Deposited GC-MS dataset from the GNPS/MassIVE repository (e.g. MSV000084033)
- Raw GC-MS chromatography-mass spectrometry sample files

## Outputs

- .mgf spectra summary file containing deconvoluted fragmentation spectra
- Quantification table of deconvolved compound features across samples
- Molecular networking artifacts suitable for repository-scale GNPS analysis

## How to apply

Retrieve the deposited GC-MS dataset (e.g. MSV000084033 from the GNPS/MassIVE repository) and launch the MSHub feature-based molecular networking workflow on the GNPS platform. MSHub performs auto-deconvolution of compound fragmentation patterns via unsupervised non-negative matrix factorization, using a Fast Fourier Transform-based strategy to overcome scalability limitations of the NNMF deconvolution on full datasets. The workflow computes a 'balance score' that quantifies the reproducibility of fragmentation patterns across all samples, and uses it to group consensus spectra into deconvolved compound features. Export the resulting .mgf spectra summary file and the accompanying quantification table, then confirm both artifacts are produced, non-empty, and that deconvolved spectra/feature rows correspond to the input sample set as reported by the MSHub/GNPS job summary.

## Related tools

- **MSHub** (Machine learning (NNMF-based) approach that performs auto-deconvolution of GC-MS compound fragmentation patterns on the GNPS platform) — https://github.com/bittremieux/GNPS_GC
- **GNPS** (Web platform hosting the MSHub feature-based molecular networking workflow, used to store, process, share, and perform molecular networking of GC-MS data) — http://github.com/CCMS-UCSD/GNPS_Workflows

## Evaluation signals

- Both output artifacts (.mgf spectra summary file and quantification table) are produced and non-empty
- Deconvolved spectra/feature rows correspond to the input sample set as reported by the MSHub/GNPS job summary
- The workflow reports the balance score quantifying reproducibility of fragmentation patterns across all samples
- Consensus spectra are grouped into deconvolved compound features consistent with the MSHub/GNPS job summary
- The NNMF deconvolution completes on the full dataset (FFT-based strategy applied without scalability failure)

## Limitations

- Unsupervised NNMF deconvolution has scalability limitations on large full datasets, requiring the Fast Fourier Transform-based strategy built into the workflow
- Deconvolution quality depends on the 'balance score' reproducibility of fragmentation patterns across samples; low sample reproducibility may affect grouping of consensus spectra

## Evidence

- [readme] Companion repository for MSHub analysis: [evidence span withheld — anchor status: not_found]
