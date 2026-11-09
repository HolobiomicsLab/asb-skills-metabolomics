---
name: execution-log-interpretation
description: 'Use when a Nextflow4MS-DIAL run (MS-DIAL followed by MS-FLO on .mzML
  inputs) has completed and you must reconstruct what happened: run duration, per-process
  timelines, resource allocation, container and parameter provenance from the execution
  reports and logs.'
when_to_use_negative:
- The goal is prospective or FAIR-compliant provenance — the article notes the workflow
  is not yet deposited to WorkflowHub and data provenance FAIRness is future work
- The analysis target is the metabolomics results themselves (peak tables, the 25,447
  identified peaks) rather than execution metadata
- The run is not Nextflow-based — e.g. manually run MS-DIAL or MS-FLO produces no
  execution_report.html or execution_timeline.html to interpret
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Nextflow
  role: Workflow engine whose retrospective provenance system generates execution_report.html,
    execution_timeline.html, and the execution log
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow4MS-DIAL
  role: The LC-MS metabolomics pipeline whose runs generate the provenance artifacts
    being interpreted
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:6643029793507cf95a442d0874401ed0e412ce01ce455dd47c47dc8aaf0c1a34
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# execution-log-interpretation

## Summary

Interprets Nextflow retrospective provenance artifacts — execution_report.html, execution_timeline.html, and logs/execution.log — produced by the Nextflow4MS-DIAL LC-MS metabolomics workflow, to recover runtime, resource usage, and run metadata after MS-DIAL/MS-FLO processing of .mzML data.

## When to use

Use when a Nextflow4MS-DIAL run (MS-DIAL followed by MS-FLO on .mzML inputs) has completed and you must reconstruct what happened: run duration, per-process timelines, resource allocation, container and parameter provenance from the execution reports and logs.

## When NOT to use

- The goal is prospective or FAIR-compliant provenance — the article notes the workflow is not yet deposited to WorkflowHub and data provenance FAIRness is future work
- The analysis target is the metabolomics results themselves (peak tables, the 25,447 identified peaks) rather than execution metadata
- The run is not Nextflow-based — e.g. manually run MS-DIAL or MS-FLO produces no execution_report.html or execution_timeline.html to interpret

## Inputs

- Nextflow run console output (redirected to logs/execution.log)
- execution_report.html (Nextflow execution report)
- execution_timeline.html (Nextflow execution timeline)
- error.txt (failed-run error log, when applicable)

## Outputs

- Workflow runtime and computational resource usage summary
- Per-process execution timeline for MS-DIAL and MS-FLO
- Run metadata record: Nextflow/workflow versions, parameter settings, resource allocation, container information, workflow progress, process-level logs

## How to apply

After launching the pipeline with a profile such as docker or singularity (e.g. against the MetaboLights MTBLS733 example dataset with msdial_params.txt and msflo_params.ini), redirect console output into logs/execution.log. Then inspect three retrospective-provenance artifacts: execution_report.html for total workflow runtime and computational resource usage; execution_timeline.html for the per-process execution timeline covering the two containerized processes (MS-DIAL, then MS-FLO); and logs/execution.log for run metadata including the Nextflow and workflow versions, parameter settings, resource allocation, container information, workflow progress, and process-level logs. Validate that all three files exist post-run and that the log records version, parameter, resource-allocation, and container metadata, matching the provenance outputs the paper attributes to Nextflow. For failed runs, error.txt serves as an example error log to diagnose the failure.

## Related tools

- **Nextflow** (Workflow engine whose retrospective provenance system generates execution_report.html, execution_timeline.html, and the execution log) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow4MS-DIAL** (The LC-MS metabolomics pipeline whose runs generate the provenance artifacts being interpreted) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- execution_report.html, execution_timeline.html, and logs/execution.log all exist after the run completes
- logs/execution.log contains the Nextflow and workflow versions, parameter settings, resource allocation, and container information
- execution_timeline.html shows entries for both containerized processes (MS-DIAL followed by MS-FLO)
- Five repeated executions of the workflow produce identical results, so log metadata should be consistent across runs
- For failed runs, a corresponding error.txt documents the failure rather than an empty or missing log

## Limitations

- Retrospective provenance is captured only in Nextflow's own report and log formats; the article notes the workflow is not yet deposited to WorkflowHub, so FAIRness of data provenance is not yet enhanced
- If a process did not execute, check for special characters in file names — underscores are safe
- Slurm CPU errors (e.g. 'req: 5; avail: 3') stem from using --cpus instead of --max_cpus in the configuration

## Evidence
