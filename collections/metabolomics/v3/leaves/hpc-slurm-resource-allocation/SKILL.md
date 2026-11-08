---
name: hpc-slurm-resource-allocation
description: Use when running a containerized Nextflow LC-MS metabolomics pipeline
  (MS-DIAL/MS-FLO) on an SLURM-managed HPC cluster such as HiPerGator, and local Docker
  execution is insufficient or reproducibility must be demonstrated cross-platform.
when_to_use_negative:
- When the analysis fits on a local machine — the article shows the workflow runs
  on macOS 13.5.1 (2.6 GHz 6-Core Intel Core i7, 16 GB memory) with Docker, and HPC
  allocation is only needed for larger scale or HPC-only environments.
- When the input raw files are not already in .mzML or .abf format — format conversion
  with ProteoWizard msConvert or Reifycs Abf Converter is a separate prior step.
- When the environment is not Unix-like or lacks a container runtime (Docker/Singularity);
  the workflow is not yet tested on Apple silicon (M-series).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Nextflow
  role: Workflow engine that submits processes to the SLURM scheduler via the singularity
    profile and records execution reports/timelines.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Singularity
  role: Container runtime used on the HiPerGator Linux HPC so the MS-DIAL 4.48 / MS-FLO
    image matches the local Docker execution.
  repo: https://www.sylabs.io/guides/3.0/user-guide/
- name: MS-DIAL
  role: Downstream LC-MS processing tool inside the container whose CPU/memory allocation
    is managed by this skill.
- name: MS-FLO
  role: Downstream LC-MS optimization tool inside the container; must be downloaded
    separately before containerized execution.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:b86dcf5e98429f20229697cfa4b14e3c0ba4c325b2d4368092348e4f56bf565d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# hpc-slurm-resource-allocation

## Summary

Allocating HPC resources (20 CPUs, 200 GB memory) via SLURM on HiPerGator to run the containerized Nextflow4MS-DIAL LC-MS metabolomics workflow reproducibly, mirroring local Docker results.

## When to use

Use when running a containerized Nextflow LC-MS metabolomics pipeline (MS-DIAL/MS-FLO) on an SLURM-managed HPC cluster such as HiPerGator, and local Docker execution is insufficient or reproducibility must be demonstrated cross-platform.

## When NOT to use

- When the analysis fits on a local machine — the article shows the workflow runs on macOS 13.5.1 (2.6 GHz 6-Core Intel Core i7, 16 GB memory) with Docker, and HPC allocation is only needed for larger scale or HPC-only environments.
- When the input raw files are not already in .mzML or .abf format — format conversion with ProteoWizard msConvert or Reifycs Abf Converter is a separate prior step.
- When the environment is not Unix-like or lacks a container runtime (Docker/Singularity); the workflow is not yet tested on Apple silicon (M-series).

## Inputs

- .mzML (or .abf) LC-HRMS raw data files placed in data/raw_data/
- MS-DIAL parameter file msdial_params.txt
- MS-FLO parameter file msflo_params.ini
- MS1 library ms1_lib.txt and MS2 library ms2_lib.msp
- Singularity container with MS-DIAL 4.48 and MS-FLO

## Outputs

- MS-DIAL and MS-FLO peak tables (.msdial files, renamed .tsv for spreadsheet use)
- execution_report.html with runtime and resource usage
- execution_timeline.html per-process timeline
- logs/execution.log with versions, parameters, container info, and process logs

## How to apply

Configure the HiPerGator partition in conf/HiPerGator.config and request 20 CPUs and 200 GB of memory through the SLURM workload manager. Use the `singularity` Nextflow profile so the same container shipping MS-DIAL 4.48 and MS-FLO runs identically to the local `docker` profile. Set per-process CPU limits with `--max_cpus` (not `--cpus`) to avoid the Slurm error 'Process requirement exceed available CPUs'. Execute `nextflow run main.nf -profile singularity` and capture execution_report.html, execution_timeline.html, and logs/execution.log to document runtime, resource usage, and provenance. The rationale is that software containerization on HPC guarantees computational reproducibility across platforms — five repeated executions produced identical results matching the reference peak counts (MS-DIAL 25,447 peaks/1,031 true positives; MS-FLO 25,414 peaks/1,030 true positives).

## Related tools

- **Nextflow** (Workflow engine that submits processes to the SLURM scheduler via the singularity profile and records execution reports/timelines.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Singularity** (Container runtime used on the HiPerGator Linux HPC so the MS-DIAL 4.48 / MS-FLO image matches the local Docker execution.) — https://www.sylabs.io/guides/3.0/user-guide/
- **MS-DIAL** (Downstream LC-MS processing tool inside the container whose CPU/memory allocation is managed by this skill.)
- **MS-FLO** (Downstream LC-MS optimization tool inside the container; must be downloaded separately before containerized execution.)

## Examples

```
nextflow run main.nf -profile singularity > logs/execution.log
```

## Evaluation signals

- Slurm job completes without the error 'Process requirement exceed available CPUs -- req: 5; avail: 3' after allocating 20 CPUs, confirming --max_cpus was set correctly.
- execution_report.html reports runtime and resource usage consistent with the 20-CPU / 200 GB allocation.
- Peak tables produced on HiPerGator match the reference results deposited in the GitHub 'results' folder (MS-DIAL 25,447 peaks, 1,031 true positives; MS-FLO 25,414 peaks, 1,030 true positives).
- Five repeated executions yield identical results, demonstrating cross-platform reproducibility vs. the macOS Docker run.
- logs/execution.log records the container image, parameter settings, and workflow versions needed for provenance.

## Limitations

- The reported test used only 10 samples (Li et al. 2018 dataset, MetaboLights MTBLS733); scalability was not quantitatively measured.
- MS-FLO must be downloaded from the Internet separately rather than being bundled with the workflow.
- Software updates (e.g. MS-DIAL) are monitored manually with no efficient maintenance plan for workflow and container.
- The workflow has been tested on Intel-based macOS and Red Hat Enterprise Linux 8.8; Apple silicon (M-series) systems are untested.
- Special characters in file names cause processes to not execute — use only safe characters such as underscores.

## Evidence

- [full_text] Run 'nextflow run main.nf -profile singularity' on the HiPerGator Linux supercomputer (Red Hat Enterprise 8.8), allocating 20 CPUs and 200 GB of memory through the SLURM workload manager: [evidence span withheld — anchor status: not_found]
