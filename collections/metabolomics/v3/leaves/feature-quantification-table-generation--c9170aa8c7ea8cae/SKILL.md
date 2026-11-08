---
name: feature-quantification-table-generation
description: Use when processing a deposited GC-MS dataset (e.g. MassIVE MSV000084033)
  through the GNPS MSHub workflow and you need deconvoluted fragmentation spectra
  plus a per-feature quantification table as exportable artifacts.
when_to_use_negative:
- The data are LC-MS/MS to be processed with the classical feature-based molecular
  networking workflow (e.g. MZmine-generated MGF) rather than GC-MS auto-deconvolution
- A quantification table already exists downstream of deconvolution and the task is
  only to map chemical classes or motifs onto the network (e.g. via pyMolNetEnhancer)
- The spectra are already deconvolved and the task is annotation or spectral matching
  rather than feature quantification
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MSHub
  role: Machine-learning workflow that performs auto-deconvolution of GC-MS compound
    fragmentation patterns and produces the .mgf spectra summary and quantification
    table
  repo: github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform hosting the MSHub feature-based molecular networking workflow and
    the job artifacts (job summary, .mgf, quantification table)
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:ed5793ee8497527a490345729bc3b0f78d13595fcb93ed38f5e10fab73b3ac88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# feature-quantification-table-generation

## Summary

Generates a sample-by-feature quantification table alongside the .mgf spectra summary from MSHub auto-deconvolution of GC-MS data on the GNPS platform, so deconvoluted compound features can be analyzed at repository scale.

## When to use

Use when processing a deposited GC-MS dataset (e.g. MassIVE MSV000084033) through the GNPS MSHub workflow and you need deconvoluted fragmentation spectra plus a per-feature quantification table as exportable artifacts.

## When NOT to use

- The data are LC-MS/MS to be processed with the classical feature-based molecular networking workflow (e.g. MZmine-generated MGF) rather than GC-MS auto-deconvolution
- A quantification table already exists downstream of deconvolution and the task is only to map chemical classes or motifs onto the network (e.g. via pyMolNetEnhancer)
- The spectra are already deconvolved and the task is annotation or spectral matching rather than feature quantification

## Inputs

- Deposited GC-MS dataset from the GNPS/MassIVE repository (e.g. MSV000084033)
- Per-sample GC-MS chromatogram files submitted to the MSHub workflow

## Outputs

- .mgf deconvolved spectra summary file
- Feature quantification table (deconvolved compound features across input samples)
- MSHub/GNPS job summary

## How to apply

Retrieve the deposited GC-MS dataset from the GNPS/MassIVE repository (e.g. MSV000084033) and launch the MSHub feature-based molecular networking workflow on GNPS, which auto-deconvolves compound fragmentation patterns via unsupervised non-negative matrix factorization. Because NNMF deconvolution does not scale naively to full datasets, rely on the Fast Fourier Transform-based strategy built into the workflow. The workflow's balance score quantifies the reproducibility of fragmentation patterns across all samples and is used to group consensus spectra into deconvolved compound features. Export the workflow result artifacts: the .mgf spectra summary file and the accompanying quantification table. Validate that both artifacts are produced and non-empty, and that the deconvolved feature rows correspond to the input sample set as reported by the MSHub/GNPS job summary.

## Related tools

- **MSHub** (Machine-learning workflow that performs auto-deconvolution of GC-MS compound fragmentation patterns and produces the .mgf spectra summary and quantification table) — github.com/bittremieux/GNPS_GC
- **GNPS** (Platform hosting the MSHub feature-based molecular networking workflow and the job artifacts (job summary, .mgf, quantification table)) — http://github.com/CCMS-UCSD/GNPS_Workflows

## Evaluation signals

- Both output artifacts — the .mgf spectra summary file and the accompanying quantification table — are produced and non-empty
- The deconvolved spectra/feature rows in the quantification table correspond to the input sample set as reported by the MSHub/GNPS job summary
- Consensus spectra are grouped into deconvolved compound features consistently with balance-score reproducibility across all samples
- The number of features in the quantification table matches the number of deconvolved compound entries in the .mgf summary

## Limitations

- Unsupervised non-negative matrix factorization deconvolution has scalability limitations on full datasets, requiring the Fast Fourier Transform-based strategy
- Feature grouping depends on the balance score, i.e. reproducibility of fragmentation patterns across samples; poorly reproducible fragmentation may affect deconvolved feature quality

## Evidence
