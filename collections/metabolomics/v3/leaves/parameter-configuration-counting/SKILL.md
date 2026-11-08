---
name: parameter-configuration-counting
description: Use when a task asks to quantify the structural complexity of a Nextflow
  workflow by enumerating its processes and user-facing parameters and comparing those
  counts against a published baseline (e.g., Metaboigniter's >20 processes / >200
  parameters).
when_to_use_negative:
- The goal is runtime reproducibility testing (verifying identical outputs across
  repeated executions) rather than static structural complexity comparison
- The workflow exposes no configuration files or parameter definitions, so no user-facing
  parameter count can be extracted
- The task requires measuring scalability, runtime, or resource usage — the source
  paper explicitly did not quantitatively measure scalability or reproducibility,
  and this skill only counts static structure
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow4MS-DIAL
  role: Subject workflow whose main.nf process definitions and conf/ parameter files
    are enumerated and counted
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine whose process definitions (main.nf) and config format (conf/*.config)
    define what is counted
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:26abf09df7053ef3ca545c64cc3b1a4f79d222ef0826986a3dbb045e5a675041
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# parameter-configuration-counting

## Summary

A structural-complexity analysis skill that parses a Nextflow workflow's process definitions and configuration files to count its major processes and user-facing parameters, then tabulates these counts against a reported comparator workflow (e.g., Metaboigniter's >20 processes and >200 parameters) to quantify workflow simplification.

## When to use

Use when a task asks to quantify the structural complexity of a Nextflow workflow by enumerating its processes and user-facing parameters and comparing those counts against a published baseline (e.g., Metaboigniter's >20 processes / >200 parameters).

## When NOT to use

- The goal is runtime reproducibility testing (verifying identical outputs across repeated executions) rather than static structural complexity comparison
- The workflow exposes no configuration files or parameter definitions, so no user-facing parameter count can be extracted
- The task requires measuring scalability, runtime, or resource usage — the source paper explicitly did not quantitatively measure scalability or reproducibility, and this skill only counts static structure

## Inputs

- Nextflow process definitions (main.nf) of the workflow under analysis
- Nextflow config files: conf/base.config and HPC-specific configs such as conf/HiPerGator.config
- Tool parameter files shipped with the workflow, e.g., msdial_params.txt and msflo_params.ini in functional_test/sample_data/
- Paper-reported comparator figures (Metaboigniter: >20 processes, >200 parameters)

## Outputs

- process_parameter_comparison.csv with one row per workflow and columns for process count and user-facing parameter count

## How to apply

Parse the Nextflow process definitions in main.nf and the conf/ files to enumerate every process, recording the process count (Nextflow4MS-DIAL's two major processes are the MS-DIAL and MS-FLO containerized steps). Next, extract user-facing parameters from conf/base.config, conf/HiPerGator.config, and the shipped tool parameter files msdial_params.txt and msflo_params.ini under functional_test/sample_data/, counting each configurable parameter. Record the comparator's paper-reported figures (Metaboigniter: over 20 processes, over 200 parameters) rather than re-deriving them, since the baseline is treated as a citation value. Assemble a CSV (process_parameter_comparison.csv) with one row per workflow and columns for process count and user-facing parameter count, then validate that the subject workflow's process count matches the paper's claim (2) and the comparator exceeds the reported thresholds.

## Related tools

- **Nextflow4MS-DIAL** (Subject workflow whose main.nf process definitions and conf/ parameter files are enumerated and counted) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine whose process definitions (main.nf) and config format (conf/*.config) define what is counted)

## Examples

```
grep -c '^process ' main.nf && grep -oE 'params\.[A-Za-z0-9_]+' conf/base.config conf/HiPerGator.config | sort -u | wc -l
```

## Evaluation signals

- process_parameter_comparison.csv exists and is valid CSV format
- The table contains explicit process-count and parameter-count fields
- Nextflow4MS-DIAL reports exactly 2 major processes, consistent with the paper's claim of a two-process design (MS-DIAL and MS-FLO)
- Metaboigniter's recorded counts exceed the reported thresholds of 20 processes and 200 parameters
- One row appears per workflow (Nextflow4MS-DIAL, Metaboigniter) so the simplification is directly comparable

## Limitations

- The Metaboigniter baseline counts (over 20 processes, over 200 parameters) are taken as paper-reported comparator values, not independently re-derived from the comparator's source code
- What counts as a 'user-facing parameter' is a curation decision spanning both engine-level config files (conf/base.config, conf/HiPerGator.config) and tool-specific parameter files (msdial_params.txt, msflo_params.ini); inconsistent definitions inflate or deflate counts
- The subject workflow was only exercised on a small test set (10 samples) with limited datasets processed under transparent parameter settings, so the counted configuration may not cover all real-world parameter surface

## Evidence
