---
name: slurm-hpc-job-submission
description: Use when LC-HRMS metabolomics data (mzML or abf) is too large for a local
  machine — e.g., more than a handful of samples — and execution should be moved to
  a SLURM-managed HPC cluster with the Nextflow4MS-DIAL singularity profile.
when_to_use_negative:
- The dataset is small enough for local execution — use the docker profile on a local
  machine (e.g., macOS 13.5.1, 2.6 GHz 6-Core Intel Core i7, 16 GB RAM) instead of
  reserving an HPC allocation.
- The cluster does not use SLURM or Singularity — the README only documents the singularity
  profile for HPC and Docker for local; a non-SLURM scheduler is untested.
- The user expects a graphical user interface — the workflow is command-line based
  only, and the article notes a GUI is a future goal.
- Input data are not mzML or abf — raw vendor formats must first be converted with
  ProteoWizard msConvert or Reifycs Abf Converter before this skill applies.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Nextflow
  role: Workflow engine that submits each process (MS-DIAL, MS-FLO) as a SLURM job
    using the singularity profile and HiPerGator config; supports containerized execution
    and the SLURM executor.
  repo: https://www.nextflow.io/
- name: Singularity
  role: Container runtime used on the HPC to package MS-DIAL (v4.48) and MS-FLO so
    the pipeline is reproducible across cluster nodes.
  repo: https://www.sylabs.io/guides/3.0/user-guide/
- name: SLURM
  role: Job scheduler on HiPerGator that receives per-process CPU/memory requirements
    from the Nextflow config; `--max_cpus` must be used instead of `--cpus` to avoid
    'exceed available CPUs' errors.
- name: Nextflow4MS-DIAL
  role: The workflow repository being submitted; contains main.nf, conf/HiPerGator.config,
    conf/base.config, and data/ layout consumed by the SLURM submission.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:3e95b99639e346c7e533fe4a1e982c99884f074af209cc0d3ea57d2afc4b57ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  - SLURM
  - Singularity
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# slurm-hpc-job-submission

## Summary

Submit and run the Nextflow4MS-DIAL LC-HRMS metabolomics workflow on a SLURM-managed high-performance computing cluster (e.g., UF HiPerGator) using the singularity profile, with per-process CPU/memory limits set via --max_cpus and --max_memory so peak detection scales beyond local machines.

## When to use

Use when LC-HRMS metabolomics data (mzML or abf) is too large for a local machine — e.g., more than a handful of samples — and execution should be moved to a SLURM-managed HPC cluster with the Nextflow4MS-DIAL singularity profile.

## When NOT to use

- The dataset is small enough for local execution — use the docker profile on a local machine (e.g., macOS 13.5.1, 2.6 GHz 6-Core Intel Core i7, 16 GB RAM) instead of reserving an HPC allocation.
- The cluster does not use SLURM or Singularity — the README only documents the singularity profile for HPC and Docker for local; a non-SLURM scheduler is untested.
- The user expects a graphical user interface — the workflow is command-line based only, and the article notes a GUI is a future goal.
- Input data are not mzML or abf — raw vendor formats must first be converted with ProteoWizard msConvert or Reifycs Abf Converter before this skill applies.

## Inputs

- mzML or abf LC-HRMS metabolomics raw data files placed in data/raw_data/
- MS-DIAL parameter file named msdial_params.txt in the data/ folder
- MS-FLO parameter file named msflo_params.ini in the data/ folder
- MS1 library named ms1_lib.txt and MS2 library named ms2_lib.msp in the data/ folder
- conf/HiPerGator.config for SLURM/Singularity resource allocation (e.g. 20 CPUs, 200 GB memory)
- Java 11+, Nextflow (≥22.10.0), and Singularity installed on the cluster

## Outputs

- MS-DIAL peak table and MS-FLO processed peak table in results/ (.msdial files, may be renamed .tsv for spreadsheet viewing)
- logs/execution.log containing Nextflow version, workflow version, parameter settings, resource allocation, container info, and process-level logs
- execution_report.html summarizing workflow runtime and computational resource usage
- execution_timeline.html showing the execution timeline for each process

## How to apply

Install Java 11+, Nextflow, and Singularity on the cluster; clone Nextflow4Metabolomics/nextflow4ms-dial and place raw mzML/abf files in data/raw_data/ together with msdial_params.txt, msflo_params.ini, ms1_lib.txt, and ms2_lib.msp in the data/ folder. Launch the workflow with `nextflow run main.nf -profile singularity > logs/execution.log`, which pulls the MS-DIAL v4.48 container and chains MS-DIAL peak detection → MS-FLO duplicate/adduct/isotope flagging, alignment, and contaminant removal. HPC resource settings live in conf/HiPerGator.config (Docker/Singularity config in conf/base.config). If SLURM errors with `Process requirement exceed available CPUs -- req: 5; avail: 3` despite allocating 20 CPUs, use `--max_cpus` (not `--cpus`) to cap per-process CPU requests — the README notes this is required because the workflow internally requests up to 5 CPUs per process while the partition may expose only 3 per node. Avoid special characters (underscores are safe) in filenames, as they can prevent processes from executing; redirect stdout to logs/execution.log so version, parameter, container, and process-level provenance is captured.

## Related tools

- **Nextflow** (Workflow engine that submits each process (MS-DIAL, MS-FLO) as a SLURM job using the singularity profile and HiPerGator config; supports containerized execution and the SLURM executor.) — https://www.nextflow.io/
- **Singularity** (Container runtime used on the HPC to package MS-DIAL (v4.48) and MS-FLO so the pipeline is reproducible across cluster nodes.) — https://www.sylabs.io/guides/3.0/user-guide/
- **SLURM** (Job scheduler on HiPerGator that receives per-process CPU/memory requirements from the Nextflow config; `--max_cpus` must be used instead of `--cpus` to avoid 'exceed available CPUs' errors.)
- **Nextflow4MS-DIAL** (The workflow repository being submitted; contains main.nf, conf/HiPerGator.config, conf/base.config, and data/ layout consumed by the SLURM submission.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
nextflow run main.nf -profile singularity > logs/execution.log
```

## Evaluation signals

- SLURM allocates the expected resources (e.g., 20 CPUs / 200 GB on HiPerGator) and the workflow completes without `Process requirement exceed available CPUs` errors.
- logs/execution.log records the Nextflow version, workflow version, parameter settings, resource allocation, and container information, matching the expected provenance fields.
- Results match the reference run on the MTBLS733 test dataset: MS-DIAL identifies ~25,447 peaks and MS-FLO yields ~25,414 peaks, with identical results across repeated executions.
- execution_report.html and execution_timeline.html are generated and reflect per-process runtime and resource usage.
- No processes are silently skipped — filenames in data/raw_data/ contain no special characters (underscores are safe), per the README FAQ.

## Limitations

- MS-FLO must be downloaded from the Internet rather than bundled with the workflow, adding a network dependency during container build/execution.
- Workflow components such as MS-DIAL are updated frequently, and updates are currently monitored manually — the container may drift from the latest tool version.
- Reproducibility and scalability were not quantitatively measured in the article; identical results were shown for five executions but only on a 10-sample MTBLS733 test set.
- Not tested on Apple silicon (M-series) systems; the singularity/HPC path is validated only on Red Hat Enterprise Linux 8.8 (HiPerGator) and macOS 13.5.1 (Intel).
- SLURM per-process CPU limits must be tuned with --max_cpus; a naive 20-CPU allocation can exceed the 3 CPUs available to a single process on a node.

## Evidence
