---
name: nextflow-pipeline-structure-analysis
description: Use when you need to quantify a Nextflow workflow's structural complexity
  — enumerate every process in main.nf and count user-facing parameters in conf/*.config
  to compare pipelines' complexity.
when_to_use_negative:
- The goal is to execute the pipeline and reproduce results (e.g. run nextflow run
  main.nf -profile docker) rather than characterize its structure.
- The workflow under analysis is not a Nextflow workflow, so process enumeration from
  main.nf and conf/*.config does not apply.
- You need runtime metrics (CPU, memory, wall time) rather than static structural
  counts — use the execution_report.html and execution_timeline.html logs instead.
tools:
- name: Nextflow4MS-DIAL
  role: The workflow whose main.nf and conf/ files are parsed to enumerate its two
    major processes (MS-DIAL, MS-FLO) and count user-facing parameters
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine whose DSL process blocks and config files (base.config, HiPerGator.config)
    define the structure being analyzed
  repo: https://www.nextflow.io/
- name: MS-DIAL
  role: First major process of the workflow; its parameters in msdial_params.txt are
    counted as user-facing configuration
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:5ac5af74d5c571eeda8e1283831f72229819a6bca758b13ebf6e6b705f6f2d80
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# nextflow-pipeline-structure-analysis

## Summary

Enumerates the processes and counts the user-facing configuration parameters of a Nextflow workflow by parsing its main.nf and conf/ files, so structural complexity can be quantified and compared across pipelines (here, Nextflow4MS-DIAL's two-process MS-DIAL→MS-FLO design versus Metaboigniter's 20+ processes and 200+ parameters).

## When to use

Use when you need to quantify a Nextflow workflow's structural complexity — enumerate every process in main.nf and count user-facing parameters in conf/*.config to compare pipelines' complexity.

## When NOT to use

- The goal is to execute the pipeline and reproduce results (e.g. run nextflow run main.nf -profile docker) rather than characterize its structure.
- The workflow under analysis is not a Nextflow workflow, so process enumeration from main.nf and conf/*.config does not apply.
- You need runtime metrics (CPU, memory, wall time) rather than static structural counts — use the execution_report.html and execution_timeline.html logs instead.

## Inputs

- Nextflow4MS-DIAL repository main.nf (Nextflow process definitions)
- conf/base.config (Docker configuration)
- conf/HiPerGator.config (HPC/Singularity configuration)
- functional_test/sample_data/msdial_params.txt (example MS-DIAL parameter file)
- functional_test/sample_data/msflo_params.ini (example MS-FLO parameter file)
- Paper-reported comparator counts for Metaboigniter (>20 processes, >200 parameters)

## Outputs

- process_parameter_comparison.csv with one row per workflow and columns for process count and user-facing parameter count

## How to apply

Start by parsing the Nextflow process definitions in main.nf (and the conf/ directory) of the nextflow4ms-dial repository, recording each process block — the paper reports the workflow contains only two major processes, MS-DIAL and MS-FLO. Next, extract user-facing parameters from conf/base.config and conf/HiPerGator.config, plus the example MS-DIAL and MS-FLO parameter files (msdial_params.txt, msflo_params.ini) shipped in functional_test/sample_data/, counting every configurable parameter. Record the paper-reported comparator values for Metaboigniter (over 20 processes and over 200 parameters). Assemble a CSV table with one row per workflow (Nextflow4MS-DIAL, Metaboigniter) and columns for process count and user-facing parameter count, highlighting the simplification Nextflow4MS-DIAL achieves. The rationale is that a lean two-process design with consolidated parameter files (msdial_params.txt, msflo_params.ini) reduces configuration burden while retaining containerized reproducibility via Docker/Singularity profiles.

## Related tools

- **Nextflow4MS-DIAL** (The workflow whose main.nf and conf/ files are parsed to enumerate its two major processes (MS-DIAL, MS-FLO) and count user-facing parameters) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine whose DSL process blocks and config files (base.config, HiPerGator.config) define the structure being analyzed) — https://www.nextflow.io/
- **MS-DIAL** (First major process of the workflow; its parameters in msdial_params.txt are counted as user-facing configuration)

## Examples

```
grep -c '^process ' main.nf; grep -oE 'params\.[A-Za-z0-9_]+' conf/base.config conf/HiPerGator.config | sort -u
```

## Evaluation signals

- process_parameter_comparison.csv exists, is valid CSV, and contains process-count and parameter-count fields for each workflow row.
- Nextflow4MS-DIAL is reported with exactly 2 major processes, consistent with the paper's claim that the design is MS-DIAL -> MS-FLO.
- Metaboigniter row exceeds the paper's reported thresholds of 20 processes and 200 parameters.
- Parameter counts are traceable to the shipped files: conf/base.config, conf/HiPerGator.config, msdial_params.txt, msflo_params.ini.
- The workflow being analyzed is the same one validated to produce identical results across five executions under identical parameters and container.

## Limitations

- The test set was small — only 10 samples — and the authors did not quantitatively measure scalability or reproducibility.
- MS-FLO must be downloaded from the Internet rather than bundled, and workflow components such as MS-DIAL are updated frequently with only manual monitoring, so structure counts can drift from the analyzed snapshot.
- Only two execution profiles are documented (conf/base.config for Docker, conf/HiPerGator.config for HPC/Singularity); other cluster configurations may define additional parameters.

## Evidence
