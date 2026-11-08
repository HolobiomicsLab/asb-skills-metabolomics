---
name: nextflow-execution-report-interpretation
description: Use when a Nextflow workflow (e.g., LC-MS metabolomics processing) has
  finished or failed and you must audit what ran, with which container/parameters,
  on which resources, or verify cross-platform reproducibility.
when_to_use_negative:
- When the goal is scientific provenance of the data analysis itself (parameters,
  peak tables) rather than execution metadata — the reports capture retrospective
  provenance, not MS-DIAL/MS-FLO parameter semantics
- When auditing a non-Nextflow pipeline (e.g., CWL, Snakemake), which will not emit
  these report artifacts
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Generates execution_report.html, execution_timeline.html, and the execution
    log containing version, parameter, resource, and container metadata during workflow
    execution
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow4MS-DIAL
  role: The LC-MS metabolomics workflow whose reports and logs are interpreted; deposits
    reference outputs in the 'results' folder for validation
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: SLURM
  role: Workload manager whose reported CPU/memory allocations (e.g., 20 CPUs, 200
    GB on HiPerGator) are cross-checked against the Nextflow reports
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:21edd0a9f97a7bb7ba7b21a992911eb637a16d82236d629020a360f3c9bd3b78
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  - SLURM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nextflow-execution-report-interpretation

## Summary

Interprets Nextflow retrospective provenance artifacts — execution_report.html, execution_timeline.html, and the execution log — from an LC-MS metabolomics workflow run to audit runtime, resource usage, versions, parameters, and container metadata, and to verify reproducibility claims.

## When to use

Use when a Nextflow workflow (e.g., LC-MS metabolomics processing) has finished or failed and you must audit what ran, with which container/parameters, on which resources, or verify cross-platform reproducibility.

## When NOT to use

- When the goal is scientific provenance of the data analysis itself (parameters, peak tables) rather than execution metadata — the reports capture retrospective provenance, not MS-DIAL/MS-FLO parameter semantics
- When auditing a non-Nextflow pipeline (e.g., CWL, Snakemake), which will not emit these report artifacts

## Inputs

- execution_report.html from a completed Nextflow run
- execution_timeline.html from a completed Nextflow run
- Nextflow execution log (e.g., logs/execution.log) with version, parameter, container, and resource metadata
- error.txt from a failed execution
- Reference outputs from the workflow GitHub 'results' folder for comparison

## Outputs

- Per-process runtime and CPU/memory resource usage summary
- Execution timeline per Nextflow process
- Retrospective provenance record (Nextflow version, workflow version, parameter settings, container image, resource allocation)
- Cross-platform reproducibility comparison against reference peak tables

## How to apply

After a run (e.g., 'nextflow run main.nf -profile docker' or '-profile singularity' under SLURM), open execution_report.html to read workflow runtime and computational resource usage (CPU/memory per process), and execution_timeline.html to inspect per-process timing for bottleneck analysis. Read the execution log (e.g., logs/execution.log), which records the Nextflow and workflow versions, parameter settings, resource allocation, container information, workflow progress, and process-level logs — these metadata constitute the retrospective provenance needed to compare runs across platforms (e.g., macOS 13.5.1 Intel vs. HiPerGator RHEL 8.8 with 20 CPUs / 200 GB memory). For failure diagnosis, consult error.txt, the example error log from a failed execution. To validate a reproduction, diff the collected reports and peak tables (.msdial renamed to .tsv) against reference outputs deposited in the GitHub 'results' folder; identical results across five executions demonstrated consistent reproducibility (MS-DIAL 25,447 peaks, MS-FLO 25,414 peaks).

## Related tools

- **Nextflow** (Generates execution_report.html, execution_timeline.html, and the execution log containing version, parameter, resource, and container metadata during workflow execution) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow4MS-DIAL** (The LC-MS metabolomics workflow whose reports and logs are interpreted; deposits reference outputs in the 'results' folder for validation) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **SLURM** (Workload manager whose reported CPU/memory allocations (e.g., 20 CPUs, 200 GB on HiPerGator) are cross-checked against the Nextflow reports)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log; open work/*/report/execution_report.html and execution_timeline.html
```

## Evaluation signals

- execution_report.html exists and reports nonzero workflow runtime and per-process resource usage for the MS-DIAL and MS-FLO processes
- execution log contains Nextflow version, workflow version, parameter settings, and container information (e.g., Docker or Singularity image)
- Collected reports from a local Docker run match those from a Singularity/SLURM run on HiPerGator with identical parameters
- Peak tables accompanying the reports match reference values (MS-DIAL 25,447 total / 1,031 true positives; MS-FLO 25,414 / 1,030) and repeat executions yield identical results
- On failure, error.txt identifies the failing process rather than the run failing silently

## Limitations

- The authors did not quantitatively measure scalability or reproducibility from these reports
- Only 10 test samples (Li et al. 2018, MTBLS733) were used to validate the reported runs
- Reports from the macOS runs reflect Intel-based MacOS only; Apple silicon (M-series) was not tested
- Workflow components such as MS-DIAL are updated frequently and monitored manually, so container/version metadata in older logs may not match current deployments

## Evidence

- [methods] Article names retrospective provenance artifacts: [evidence span withheld — anchor status: not_found]
