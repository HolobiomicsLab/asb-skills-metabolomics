---
name: metabolomics-benchmark-comparison
description: Use when reproducing or benchmarking peak-detection/quality-filtering
  outputs from competing LC-MS metabolomics tools on the same public dataset, and
  when validating that repeated executions produce identical counts.
when_to_use_negative:
- The dataset has already been processed to a curated feature table and no new tool
  comparison is needed.
- The benchmark aims to compare machine-learning classifiers rather than peak-processing
  outputs.
- The input is a peak table or a statistical result, not raw .mzML LC-HRMS data.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0121
tools:
- name: Nextflow4MS-DIAL
  role: Containerized Nextflow workflow that runs MS-DIAL and MS-FLO sequentially
    on .mzML LC-HRMS data, producing reproducible peak tables for benchmark comparison.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Performs the initial LC-focused peak detection, m/z and retention time identification,
    and peak-intensity calculation before transfer to MS-FLO.
- name: MS-FLO
  role: Refines the MS-DIAL peak table through duplicate peak flagging, adduct and
    isotope flagging, peak alignment, and contaminant ion removal to yield the benchmarked
    quality-filtered output.
- name: Nextflow
  role: Orchestrates the containerized workflow runs; supports the Docker and Singularity
    profiles that make repeated executions identical.
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:b825d3bc405e2c1dadad25238d79f1aaf85158c2c7c21e1d65112e778d652e77
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# metabolomics-benchmark-comparison

## Summary

Reproduce and compare tool-specific peak-processing outputs on the same LC-HRMS dataset — as in Nextflow4MS-DIAL's five-run validation on MetaboLights MTBLS733, where MS-DIAL and MS-FLO peak counts and true-positive fractions were compared against reported values.

## When to use

Use when reproducing or benchmarking peak-detection/quality-filtering outputs from competing LC-MS metabolomics tools on the same public dataset, and when validating that repeated executions produce identical counts.

## When NOT to use

- The dataset has already been processed to a curated feature table and no new tool comparison is needed.
- The benchmark aims to compare machine-learning classifiers rather than peak-processing outputs.
- The input is a peak table or a statistical result, not raw .mzML LC-HRMS data.

## Inputs

- 10-sample LC-HRMS dataset in .mzML format (MetaboLights MTBLS733 from Li et al.)
- MS-DIAL parameter file (msdial_params.txt) and MS-FLO parameter file (msflo_params.ini)
- MS1 library (ms1_lib.txt) and MS2 library (ms2_lib.msp) for metabolite identification

## Outputs

- MS-DIAL output peak table (.msdial/.tsv) with total peak count and true-positive count
- MS-FLO output peak table with total peak count and true-positive count
- Summary CSV (e.g. peak_counts_summary.csv) containing total peaks and true-positive percentages per tool

## How to apply

Run the containerized Nextflow4MS-DIAL workflow (Nextflow with the Docker profile, containing MS-DIAL version 4.48 and MS-FLO) on a public benchmark dataset such as the 10-sample MTBLS733 dataset from Li et al. Load the MS-DIAL output peak table (with .msdial files renamed to .tsv) and the MS-FLO output peak table — generated after duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal — into pandas, counting total peaks and true-positive peaks against a reference feature set. Repeat the full workflow multiple times (the paper ran it five times) to check for consistency, then write the totals and percentages into a summary CSV. Judging correctness: the computed values must match article-reported values exactly — MS-DIAL 25,447 total / 1,031 true positive (4.05%) and MS-FLO 25,414 total / 1,030 true positive (4.05%) — consistent with the paper's finding that five repeated executions produced identical results.

## Related tools

- **Nextflow4MS-DIAL** (Containerized Nextflow workflow that runs MS-DIAL and MS-FLO sequentially on .mzML LC-HRMS data, producing reproducible peak tables for benchmark comparison.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Performs the initial LC-focused peak detection, m/z and retention time identification, and peak-intensity calculation before transfer to MS-FLO.)
- **MS-FLO** (Refines the MS-DIAL peak table through duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal to yield the benchmarked quality-filtered output.)
- **Nextflow** (Orchestrates the containerized workflow runs; supports the Docker and Singularity profiles that make repeated executions identical.) — https://www.nextflow.io/

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Total peaks counted in the MS-DIAL output must equal 25,447 on the MTBLS733 test set.
- Total peaks counted in the MS-FLO output must equal 25,414 on the MTBLS733 test set.
- True-positive peaks for MS-DIAL must equal 1,031 (4.05% of total) and for MS-FLO must equal 1,030 (4.05%).
- Five independent executions of the workflow must produce identical peak counts and true-positive percentages.
- Output summary CSV must contain the four counts and two percentages, matching the article-reported values exactly.

## Limitations

- MS-FLO must be downloaded from the Internet rather than bundled with the workflow.
- Few LC-MS metabolomics data sets had been processed with both MS-DIAL and MS-FLO under transparent parameter settings, limiting reproduction of steps.
- MS-DIAL is updated frequently; workflow components are monitored manually with no efficient maintenance plan for the workflow and container.
- The workflow is command-line only; no graphical user interface exists for users with limited computational experience.
- The workflow has not been tested on Apple silicon (M-series) systems, only Intel-based MacOS and Red Hat Enterprise Linux 8.8 HPC.

## Evidence
