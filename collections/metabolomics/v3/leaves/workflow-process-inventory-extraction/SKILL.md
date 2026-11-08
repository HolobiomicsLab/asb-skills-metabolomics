---
name: workflow-process-inventory-extraction
description: Use when you must quantify the structural complexity of a Nextflow workflow
  — enumerating processes from main.nf and user-facing parameters from conf/*.config
  and tool parameter files — to compare against another workflow's reported counts.
when_to_use_negative:
- The goal is to verify that repeated executions produce identical outputs — that
  is a runtime reproducibility test, not a structural inventory
- The repository does not ship Nextflow process definitions (main.nf) or conf/ config
  files, so no authoritative process/parameter surface can be parsed
- The comparator workflow's counts cannot be traced to a published report and must
  be independently re-derived instead of being recorded as baseline values
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Nextflow4MS-DIAL
  role: Target workflow whose main.nf and conf/ files are parsed to enumerate its
    two major processes (MS-DIAL, MS-FLO) and user-facing parameters
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine whose process and configuration syntax (main.nf, conf/base.config,
    conf/HiPerGator.config) defines the units being counted
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: First major process in the target workflow; its msdial_params.txt parameter
    file supplies the user-facing parameter count
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:66df14cc4cd7598f11c132ebe4a4edec49028cc7c960da7fe97adc58e747e982
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# workflow-process-inventory-extraction

## Summary

A static repository-analysis skill that inventories the processes and user-facing parameters of a Nextflow workflow by parsing its main.nf and conf/ configuration files, then tabulating these counts against a literature-reported comparator (e.g., Metaboigniter's 20+ processes and 200+ parameters) to quantify workflow simplification.

## When to use

Use when you must quantify the structural complexity of a Nextflow workflow — enumerating processes from main.nf and user-facing parameters from conf/*.config and tool parameter files — to compare against another workflow's reported counts.

## When NOT to use

- The goal is to verify that repeated executions produce identical outputs — that is a runtime reproducibility test, not a structural inventory
- The repository does not ship Nextflow process definitions (main.nf) or conf/ config files, so no authoritative process/parameter surface can be parsed
- The comparator workflow's counts cannot be traced to a published report and must be independently re-derived instead of being recorded as baseline values

## Inputs

- Nextflow process definitions (main.nf) from the workflow repository
- Nextflow configuration files (conf/base.config for Docker/local, conf/HiPerGator.config for HPC/Singularity)
- Example MS-DIAL and MS-FLO parameter files (msdial_params.txt, msflo_params.ini in functional_test/sample_data/)
- Paper-reported comparator counts (e.g., Metaboigniter: >20 processes, >200 parameters)

## Outputs

- CSV table (e.g., process_parameter_comparison.csv) with one row per workflow and columns for process count and user-facing parameter count
- Highlighted comparison showing the simplification achieved by the target workflow versus the baseline

## How to apply

Parse the Nextflow process definitions in main.nf and the conf/ files of the repository to enumerate every process, recording the process count (for nextflow4ms-dial the paper reports only two major processes, MS-DIAL and MS-FLO). Extract user-facing parameters from conf/base.config, conf/HiPerGator.config, and the example MS-DIAL and MS-FLO parameter files (msdial_params.txt and msflo_params.ini shipped in functional_test/sample_data/), counting each configurable parameter. Record the comparator's paper-reported figures (Metaboigniter: over 20 processes, over 200 parameters) as baseline values rather than re-deriving them. Assemble a CSV with one row per workflow and columns for process count and user-facing parameter count, and validate that the parsed counts match the paper's claims before publishing the table.

## Related tools

- **Nextflow4MS-DIAL** (Target workflow whose main.nf and conf/ files are parsed to enumerate its two major processes (MS-DIAL, MS-FLO) and user-facing parameters) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine whose process and configuration syntax (main.nf, conf/base.config, conf/HiPerGator.config) defines the units being counted) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (First major process in the target workflow; its msdial_params.txt parameter file supplies the user-facing parameter count)

## Examples

```
nextflow run main.nf -profile functional_test > logs/execution.log
```

## Evaluation signals

- Output CSV exists, parses cleanly, and contains process-count and parameter-count fields
- Nextflow4MS-DIAL row reports exactly 2 major processes (MS-DIAL and MS-FLO), consistent with the paper's claim
- Metaboigniter row exceeds the reported 20-process and 200-parameter thresholds
- Every counted parameter can be traced to a specific config file path (conf/base.config, conf/HiPerGator.config, msdial_params.txt, msflo_params.ini)
- Parsed counts from the live repository match the paper-reported figures at the time of comparison

## Limitations

- Counts reflect the repository state at analysis time; components such as MS-DIAL are updated frequently and the authors monitor updates manually, so the parameter surface may drift
- MS-FLO must be downloaded from the Internet rather than bundled with the workflow, so its parameter inventory depends on an external artifact
- Comparator figures (Metaboigniter's 20+ processes, 200+ parameters) are paper-reported values, not independently verified counts
- The paper notes it did not quantitatively measure scalability or reproducibility, so the table reflects structural complexity only, not runtime behavior

## Evidence
