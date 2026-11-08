---
name: metabolomics-dataset-qc
description: Use when validating that a containerized LC-MS metabolomics pipeline
  (e.g., MS-DIAL -> MS-FLO under Nextflow) produces deterministic, identical results
  across repeated executions before trusting downstream analyses.
when_to_use_negative:
- Input is already a processed peak/feature table — this skill validates raw-data
  processing determinism, not downstream statistics
- You need a quantitative scalability or reproducibility benchmark — the authors note
  'We did not quantitatively measure the scalability or reproducibility', so this
  check is binary identity, not a statistical reproducibility measure
- Processing more than the 10-sample test set for the first time — validate determinism
  on the small reference set before scaling to larger datasets
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Nextflow4MS-DIAL
  role: The workflow being executed five times and checked for run-to-run reproducibility
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine used to launch each of the five identical executions with
    the docker profile
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Containerized peak-detection stage whose peak table (25,447 peaks expected)
    is compared across runs
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-FLO
  role: Containerized filtering stage whose output peak table (25,414 peaks, 1,030
    true positives) is compared across runs
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Docker
  role: Containerization ensuring identical software environment across the five repeated
    executions
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:b452e9024bb00b340267b52f268cf4b9492ec7308e9941bd8e80f81e623af4c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# metabolomics-dataset-qc

## Summary

A run-to-run reproducibility quality check for the Nextflow4MS-DIAL LC-MS metabolomics workflow: execute the pipeline five times on the same input data and verify identical peak-table outputs. It grounds workflow QC claims in concrete checksum/diff comparisons rather than a single successful run.

## When to use

Use when validating that a containerized LC-MS metabolomics pipeline (e.g., MS-DIAL -> MS-FLO under Nextflow) produces deterministic, identical results across repeated executions before trusting downstream analyses.

## When NOT to use

- Input is already a processed peak/feature table — this skill validates raw-data processing determinism, not downstream statistics
- You need a quantitative scalability or reproducibility benchmark — the authors note 'We did not quantitatively measure the scalability or reproducibility', so this check is binary identity, not a statistical reproducibility measure
- Processing more than the 10-sample test set for the first time — validate determinism on the small reference set before scaling to larger datasets

## Inputs

- 10 raw LC-MS mzML sample files from the MetaboLights MTBLS733 test dataset (Li et al. 2018)
- MS-DIAL parameter file (msdial_params.txt) and MS-FLO parameter file (msflo_params.ini) in data/
- Docker container with MS-DIAL v4.48 and MS-FLO

## Outputs

- Five sets of peak tables (.tsv): MS-DIAL peak table and MS-FLO output per run
- run_comparison.tsv recording pairwise checksum/diff agreement across the five runs
- logs/execution.log per run

## How to apply

Place the 10 raw LC-MS mzML samples (from Li et al. 2018, available via MetaboLights MTBLS733) with msdial_params.txt and msflo_params.ini in the data/ directory, then run 'nextflow run main.nf -profile docker' five times, resetting the work directory between runs so each starts from identical inputs. Each run uses the Docker container carrying MS-DIAL version 4.48 and MS-FLO, and the log is captured to logs/execution.log. Collect the .tsv peak tables (MS-DIAL peak table and MS-FLO output) from the results directory after each run. Compare the five runs pairwise by checksum and row-level diff, recording agreement in run_comparison.tsv. The check passes only if all five runs are identical AND totals match the paper's reported values: MS-DIAL 25,447 total peaks (1,031 true positives, 4.05%) and MS-FLO 25,414 total peaks (1,030 true positives, 4.05%).

## Related tools

- **Nextflow4MS-DIAL** (The workflow being executed five times and checked for run-to-run reproducibility) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine used to launch each of the five identical executions with the docker profile) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Containerized peak-detection stage whose peak table (25,447 peaks expected) is compared across runs) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-FLO** (Containerized filtering stage whose output peak table (25,414 peaks, 1,030 true positives) is compared across runs) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Docker** (Containerization ensuring identical software environment across the five repeated executions) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- All five runs produce checksum-identical MS-DIAL and MS-FLO .tsv peak tables (row-level diff shows zero differences)
- MS-DIAL peak table totals 25,447 peaks with 1,031 true positives (4.05%)
- MS-FLO output totals 25,414 peaks with 1,030 true positives (4.05%)
- Each run's logs/execution.log records identical Nextflow and workflow versions, parameter settings, and container information
- Work directory was reset between runs so all five executions started from the same inputs

## Limitations

- The authors note 'We did not quantitatively measure the scalability or reproducibility' — this is a binary identity check, not a quantitative reproducibility metric
- The check was established on only 10 mzML samples ('The test data included only 10 samples'); determinism on larger or different datasets is not demonstrated
- Verified on macOS 13.5.1 (Intel Core i7) and HiPerGator (RHEL 8.8); not tested on Apple silicon (M-series) systems
- MS-FLO must be downloaded from the Internet rather than bundled with the workflow, which can affect environment pinning

## Evidence
