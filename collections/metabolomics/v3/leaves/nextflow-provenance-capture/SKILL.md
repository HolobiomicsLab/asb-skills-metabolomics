---
name: nextflow-provenance-capture
description: Use when a Nextflow workflow run (e.g. containerized MS-DIAL/MS-FLO processing
  of .mzML LC-MS data) must yield retrospective provenance artifacts — execution reports,
  timeline, and logs — documenting runtime, resources, and data derivation.
when_to_use_negative:
- You need prospective/FAIR-compliant provenance (e.g. workflow deposit to WorkflowHub)
  — the paper states this is not yet done and is future work.
- Input is not a Nextflow-executed workflow (e.g. a manual MS-DIAL GUI run) — there
  is no Nextflow provenance machinery to capture.
- You need quantitative scalability or reproducibility metrics — the authors note
  these were not quantitatively measured.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine that captures retrospective provenance (task reports, intermediate
    results, working-directory data files) during execution
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow4MS-DIAL
  role: The LC-MS metabolomics workflow whose runs generate the provenance artifacts
    (execution_report.html, execution_timeline.html, logs/execution.log)
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Docker
  role: Container profile used for local execution; container information is recorded
    in the provenance log
- name: Singularity
  role: Container profile for HPC environments (e.g. HiPerGator); container information
    is recorded in the provenance log
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:954aa7dec1a8222ebfecb57754b7a5d9224a0f23d8ecd76e95e02cd60ed61ac0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# nextflow-provenance-capture

## Summary

Captures retrospective provenance of a Nextflow workflow execution by collecting Nextflow's task execution reports, intermediate results, and data files stored in each task's working directory, as demonstrated by the Nextflow4MS-DIAL LC-MS metabolomics pipeline (MS-DIAL -> MS-FLO).

## When to use

Use when a Nextflow workflow run (e.g. containerized MS-DIAL/MS-FLO processing of .mzML LC-MS data) must yield retrospective provenance artifacts — execution reports, timeline, and logs — documenting runtime, resources, and data derivation.

## When NOT to use

- You need prospective/FAIR-compliant provenance (e.g. workflow deposit to WorkflowHub) — the paper states this is not yet done and is future work.
- Input is not a Nextflow-executed workflow (e.g. a manual MS-DIAL GUI run) — there is no Nextflow provenance machinery to capture.
- You need quantitative scalability or reproducibility metrics — the authors note these were not quantitatively measured.

## Inputs

- Nextflow workflow main.nf (Nextflow4MS-DIAL, requires Java 11+ and Nextflow >= 22.10.0)
- Input LC-MS data files (.mzML or .abf) in data/raw_data/
- Configuration files: msdial_params.txt, msflo_params.ini, ms1_lib.txt, ms2_lib.msp in data/
- Execution profile choice: functional_test, docker, or singularity

## Outputs

- execution_report.html — workflow runtime and computational resource usage summary
- execution_timeline.html — per-process execution timeline
- logs/execution.log — Nextflow/workflow versions, parameter settings, resource allocation, container information, workflow progress, process-level logs
- Per-task working directories containing task execution reports, intermediate results, and data files
- error.txt — example error log from a failed execution

## How to apply

Run the pipeline with a defined profile (functional_test for validation on the example MetaboLights MTBLS733 mzML dataset with msdial_params.txt and msflo_params.ini; docker for local runs; singularity for HPC) and redirect Nextflow output to logs/execution.log. Nextflow automatically captures retrospective provenance: task execution reports, intermediate results, and data files in each task's working directory. After the run, inspect execution_report.html for workflow runtime and computational resource usage, execution_timeline.html for the per-process execution timeline, and logs/execution.log for metadata including Nextflow and workflow versions, parameter settings, resource allocation, container information, workflow progress, and process-level logs. The rationale is that automated capture avoids human error from manual logging and supports reproducibility across Docker/Singularity environments.

## Related tools

- **Nextflow** (Workflow engine that captures retrospective provenance (task reports, intermediate results, working-directory data files) during execution) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow4MS-DIAL** (The LC-MS metabolomics workflow whose runs generate the provenance artifacts (execution_report.html, execution_timeline.html, logs/execution.log)) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Docker** (Container profile used for local execution; container information is recorded in the provenance log)
- **Singularity** (Container profile for HPC environments (e.g. HiPerGator); container information is recorded in the provenance log)

## Examples

```
nextflow run main.nf -profile functional_test > logs/execution.log
```

## Evaluation signals

- execution_report.html, execution_timeline.html, and logs/execution.log all exist after the run.
- logs/execution.log contains Nextflow version, workflow version, parameter settings, resource allocation, and container metadata.
- With the functional_test profile on the MTBLS733 example dataset, both containerized processes (MS-DIAL, then MS-FLO) appear in the execution timeline.
- Repeated runs (the paper executed the workflow five times) produce identical results, confirming consistent reproducibility alongside the captured provenance.
- Failed executions are distinguishable via error.txt rather than a malformed or missing execution.log.

## Limitations

- Provenance capture is tied to Nextflow's working-directory structure; it does not yet enhance FAIRness of data provenance (e.g. deposit to WorkflowHub), which the authors list as a goal.
- MS-FLO must be downloaded from the Internet rather than bundled, so container metadata alone does not guarantee full reproducibility.
- Workflow components such as MS-DIAL are updated frequently and monitored manually, with no efficient maintenance plan for the workflow and container.
- Provenance is command-line based only; there is no graphical user interface for users with limited computational experience.

## Evidence

- [results] Retrospective provenance in task working directories: [evidence span withheld — anchor status: not_found]
