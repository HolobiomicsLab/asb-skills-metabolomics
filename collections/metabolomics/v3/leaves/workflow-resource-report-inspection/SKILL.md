---
name: workflow-resource-report-inspection
description: Use when you must audit how a completed Nextflow run recorded runtime,
  CPU/memory resource usage, container images, parameter settings, and version metadata,
  or when reproducing provenance outputs claimed in a paper.
when_to_use_negative:
- The goal is prospective provenance or workflow design — this skill only covers retrospective
  provenance captured after execution
- The run was not launched with Nextflow's report/timeline generation, so no execution_report.html
  or execution_timeline.html was produced
- The question concerns metabolomics results themselves (e.g. the 25,447 MS-DIAL peaks
  or MS-FLO flagging output) rather than execution metadata
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine that captures retrospective provenance (execution reports,
    timelines, per-task working directories) during pipeline execution
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow4MS-DIAL
  role: The LC-MS metabolomics workflow whose MS-DIAL → MS-FLO execution produces
    the provenance artifacts being inspected
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Docker
  role: Container profile whose image identifiers appear in the execution log's container
    metadata
- name: Singularity
  role: HPC container profile whose container information is recorded in the provenance
    logs on systems like HiPerGator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:db56defca8781d37a8b7b4b69d32b797d06bff586bc2bdc9add1933589a1cb46
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  - Singularity
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# workflow-resource-report-inspection

## Summary

A skill for reconstructing the retrospective provenance of a Nextflow-executed workflow (e.g. Nextflow4MS-DIAL) by inspecting the execution_report.html, execution_timeline.html, and logs/execution.log artifacts that Nextflow generates during a run, alongside intermediate results stored in each task's working directory.

## When to use

Use when you must audit how a completed Nextflow run recorded runtime, CPU/memory resource usage, container images, parameter settings, and version metadata, or when reproducing provenance outputs claimed in a paper.

## When NOT to use

- The goal is prospective provenance or workflow design — this skill only covers retrospective provenance captured after execution
- The run was not launched with Nextflow's report/timeline generation, so no execution_report.html or execution_timeline.html was produced
- The question concerns metabolomics results themselves (e.g. the 25,447 MS-DIAL peaks or MS-FLO flagging output) rather than execution metadata

## Inputs

- Completed Nextflow run of main.nf with the docker or functional_test profile (input: .mzML or .abf LC-HRMS files from MetaboLights MTBLS733 example data)
- Nextflow-generated artifacts: execution_report.html, execution_timeline.html
- Redirected stdout log: logs/execution.log (and error.txt for failed runs)
- Per-task working directories under work/ containing intermediate results and data files

## Outputs

- Verified set of provenance artifacts: execution_report.html, execution_timeline.html, logs/execution.log
- Extracted metadata: Nextflow version, workflow version, parameter settings (msdial_params.txt / msflo_params.ini values), resource allocation, container identifiers, process-level execution logs
- Per-process execution timeline and aggregate runtime/resource-usage summary

## How to apply

Run or re-run the workflow (e.g. `nextflow run main.nf -profile docker > logs/execution.log` for the MS-DIAL → MS-FLO pipeline) so Nextflow persists its retrospective provenance, which consists of task execution reports, intermediate results, and data files stored within each task's working directory. Then inspect execution_report.html for aggregate workflow runtime and computational resource usage, and execution_timeline.html for the per-process execution timeline covering the two containerized processes (MS-DIAL followed by MS-FLO). Parse logs/execution.log for recorded metadata: Nextflow and workflow versions, parameter settings, resource allocation, container information (Docker/Singularity), workflow progress, and process-level logs. Validation follows the paper's own criterion: confirm that execution_report.html, execution_timeline.html, and logs/execution.log all exist after the run and that the log contains version, parameter, resource-allocation, and container metadata matching the provenance outputs the paper attributes to Nextflow.

## Related tools

- **Nextflow** (Workflow engine that captures retrospective provenance (execution reports, timelines, per-task working directories) during pipeline execution) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow4MS-DIAL** (The LC-MS metabolomics workflow whose MS-DIAL → MS-FLO execution produces the provenance artifacts being inspected) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Docker** (Container profile whose image identifiers appear in the execution log's container metadata)
- **Singularity** (HPC container profile whose container information is recorded in the provenance logs on systems like HiPerGator)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- All three provenance artifacts exist after the run: execution_report.html, execution_timeline.html, and logs/execution.log
- logs/execution.log contains Nextflow and workflow versions, parameter settings, resource allocation, and container information
- execution_timeline.html shows separate entries for the two containerized processes (MS-DIAL and MS-FLO)
- execution_report.html reports nonzero runtime and computational resource usage consistent with the profile used (docker for local, singularity for HPC)
- Per-task working directories under work/ hold intermediate results and data files matching the MS-DIAL → MS-FLO chain

## Limitations

- Provenance files are only produced if the run is launched so output is redirected, e.g. `> logs/execution.log`; error.txt is the example log for failed executions
- Nextflow's retrospective provenance is spread across per-task working directories, so reconstructing full data-derivation history requires inspecting each task directory
- The paper notes the workflow's FAIRness of data provenance is not yet enhanced — it has not been deposited to WorkflowHub — so machine-readable provenance standards are not guaranteed

## Evidence

- [results] Retrospective provenance in Nextflow is captured through task execution reports, intermediate results, and data files stored within each task's working directory.: [evidence span withheld — anchor status: not_found]
- [methods] The workflow uses Nextflow to orchestrate two containerized processes, MS-DIAL followed by MS-FLO.: [evidence span withheld — anchor status: not_found]
