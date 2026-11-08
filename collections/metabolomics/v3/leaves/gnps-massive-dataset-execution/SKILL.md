---
name: gnps-massive-dataset-execution
description: Use when a GC-MS dataset deposited on GNPS/MassIVE (an MSV accession)
  must be processed into deconvoluted compound fragmentation spectra via the MSHub/GNPS
  workflow.
when_to_use_negative:
- The input data are LC-MS/MS feature-based molecular networking data rather than
  EI GC-MS data requiring spectral deconvolution
- The spectra are already deconvoluted (an .mgf consensus spectra file and feature
  quantification table already exist for the dataset)
- The dataset is not deposited on the GNPS/MassIVE repository and cannot be launched
  as a GNPS ProteoSAFe workflow job
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSHub
  role: Machine learning engine that performs auto-deconvolution of GC-MS fragmentation
    patterns via unsupervised non-negative matrix factorization within the GNPS workflow
- name: GNPS
  role: Platform on which the MSHub feature-based molecular networking workflow is
    launched against the deposited MassIVE dataset and from which result artifacts
    are exported
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: GNPS_GC
  role: Companion repository to the MSHub auto-deconvolution manuscript documenting
    the analysis
  repo: https://github.com/bittremieux/GNPS_GC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:f2ab03397e0d9035dbe368c7c27db5f007fd828a52f7286c94e4daf7cf815583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - GNPS_GC
  - MSHub
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gnps-massive-dataset-execution

## Summary

Executes the MSHub feature-based molecular networking workflow on the GNPS platform to auto-deconvolve a deposited GC-MS dataset (e.g. MSV000084033 from MassIVE) into deconvoluted fragmentation spectra and a quantification table suitable for repository-scale analysis.

## When to use

Use when a GC-MS dataset deposited on GNPS/MassIVE (an MSV accession) must be processed into deconvoluted compound fragmentation spectra via the MSHub/GNPS workflow.

## When NOT to use

- The input data are LC-MS/MS feature-based molecular networking data rather than EI GC-MS data requiring spectral deconvolution
- The spectra are already deconvoluted (an .mgf consensus spectra file and feature quantification table already exist for the dataset)
- The dataset is not deposited on the GNPS/MassIVE repository and cannot be launched as a GNPS ProteoSAFe workflow job

## Inputs

- Deposited GC-MS dataset accession from the GNPS/MassIVE repository (e.g. MSV000084033)
- Raw GC-MS sample files (EI mass spectra) referenced by the MassIVE deposit

## Outputs

- Deconvoluted spectra summary file (.mgf) containing consensus fragmentation spectra
- Quantification table of deconvolved compound features across samples
- GNPS/MSHub job summary documenting the executed workflow

## How to apply

Retrieve the deposited GC-MS dataset (e.g. MSV000084033) from the GNPS/MassIVE repository and launch the MSHub feature-based molecular networking workflow on GNPS. The workflow performs auto-deconvolution of compound fragmentation patterns via unsupervised non-negative matrix factorization (NNMF), and uses a Fast Fourier Transform-based strategy to overcome NNMF scalability limitations on full repository-scale datasets. It computes a 'balance score' that quantifies the reproducibility of fragmentation patterns across all samples, which is used to group consensus spectra into deconvolved compound features. Export the workflow result artifacts — the .mgf spectra summary file and the accompanying quantification table — and validate that both artifacts are produced and non-empty and that deconvolved spectra/feature rows correspond to the input sample set as reported by the MSHub/GNPS job summary.

## Related tools

- **MSHub** (Machine learning engine that performs auto-deconvolution of GC-MS fragmentation patterns via unsupervised non-negative matrix factorization within the GNPS workflow)
- **GNPS** (Platform on which the MSHub feature-based molecular networking workflow is launched against the deposited MassIVE dataset and from which result artifacts are exported) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **GNPS_GC** (Companion repository to the MSHub auto-deconvolution manuscript documenting the analysis) — https://github.com/bittremieux/GNPS_GC

## Evaluation signals

- Both result artifacts — the .mgf spectra summary file and the quantification table — are produced and non-empty
- Deconvolved spectra/feature rows correspond to the input sample set as reported by the MSHub/GNPS job summary
- Consensus spectra are grouped into deconvolved compound features using the balance score quantifying fragmentation-pattern reproducibility across all samples
- The job summary confirms the MSHub feature-based molecular networking workflow (NNMF auto-deconvolution with FFT-based scalability strategy) ran to completion on the retrieved MSV accession

## Limitations

- Unsupervised NNMF deconvolution has scalability limitations on full repository-scale datasets; the workflow must rely on the built-in Fast Fourier Transform-based strategy to remain tractable
- The workflow is tied to the GNPS/MassIVE platform and its web-based job system; it cannot be applied to local datasets outside GNPS without adaptation

## Evidence
