---
name: containerized-pipeline-configuration
description: Use when running an LC-HRMS metabolomics pipeline (.mzML/.abf inputs)
  under Nextflow where MS-DIAL and MS-FLO must execute in containers on a local machine
  or HPC/Slurm environment, or when diagnosing container profile, resource-limit,
  or configuration-file setup issues.
when_to_use_negative:
- When a graphical interface or point-and-click processing is needed — Nextflow4MS-DIAL
  is command-line-based only
- When raw vendor files must be kept in their native format without conversion — inputs
  must first be converted to .mzML (ProteoWizard msConvert) or .abf (Reifycs Abf Converter)
- When the goal is only statistical or pathway analysis of an existing peak table
  — this skill configures upstream LC-MS feature processing, not downstream analysis
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0769
tools:
- name: Nextflow
  role: Workflow engine that orchestrates the two containerized processes (MS-DIAL
    then MS-FLO), supports Docker/Singularity profiles, multiple schedulers, and emits
    execution_report.html, execution_timeline.html, and logs
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Docker
  role: Container runtime used by the `docker` profile for local execution, configured
    in conf/base.config
- name: Singularity
  role: Container runtime used by the `singularity` profile for high-performance computing
    environments (e.g. HiPerGator via conf/HiPerGator.config)
- name: MS-DIAL
  role: First containerized process; performs peak detection, alignment, and optional
    metabolite identification on .mzML/.abf data using msdial_params.txt
- name: MS-FLO
  role: Second containerized process; performs duplicate peak flagging, adduct/isotope
    flagging, peak alignment, and contaminant ion removal on peak tables transferred
    from MS-DIAL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:74f2a414617648b6ee05d755cc9d795dd975177358fd1a86f36545d83cac3281
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# containerized-pipeline-configuration

## Summary

Configure and run a containerized Nextflow workflow (Nextflow4MS-DIAL) so that MS-DIAL and MS-FLO execute inside Docker or Singularity containers with explicit profile selection, parameter files, and resource allocation, ensuring computational reproducibility of LC-MS metabolomics processing.

## When to use

Use when running an LC-HRMS metabolomics pipeline (.mzML/.abf inputs) under Nextflow where MS-DIAL and MS-FLO must execute in containers on a local machine or HPC/Slurm environment, or when diagnosing container profile, resource-limit, or configuration-file setup issues.

## When NOT to use

- When a graphical interface or point-and-click processing is needed — Nextflow4MS-DIAL is command-line-based only
- When raw vendor files must be kept in their native format without conversion — inputs must first be converted to .mzML (ProteoWizard msConvert) or .abf (Reifycs Abf Converter)
- When the goal is only statistical or pathway analysis of an existing peak table — this skill configures upstream LC-MS feature processing, not downstream analysis

## Inputs

- Nextflow4MS-DIAL repository (github.com/Nextflow4Metabolomics/nextflow4ms-dial) with main.nf and conf/base.config
- Raw LC-MS data files in .mzML or .abf format placed in data/raw_data/
- MS-DIAL parameter file named msdial_params.txt in data/
- MS-FLO parameter file named msflo_params.ini in data/
- Metabolite library files ms1_lib.txt and ms2_lib.msp (optional, for metabolite identification)
- Java 11+ and Nextflow (>=22.10.0) installation; Docker or Singularity runtime

## Outputs

- results/ folder containing MS-DIAL and MS-FLO outputs (peak tables, with .msdial extensions changed to .tsv)
- execution_report.html summarizing workflow runtime and computational resource usage
- execution_timeline.html showing per-process execution timeline
- logs/execution.log with Nextflow/workflow versions, parameter settings, resource allocation, container information, and process-level logs

## How to apply

Clone the nextflow4ms-dial repository and select the execution profile in the `nextflow run main.nf -profile ...` command: `functional_test` for a quick validation run, `docker` for local execution, and `singularity` for HPC systems (e.g. HiPerGator with the conf/HiPerGator.config for Slurm/Singularity settings). Place the user's mzML or abf raw files in data/raw_data/, the MS-DIAL and MS-FLO parameter files as data/msdial_params.txt and data/msflo_params.ini, and the metabolite libraries as data/ms1_lib.txt and data/ms2_lib.msp; verify the reference file in conf/base.config and the Docker configuration before launching. Nextflow then orchestrates the two containerized processes, MS-DIAL followed by MS-FLO, automatically transferring peak tables between them. When Slurm reports 'Process requirement exceed available CPUs', use --max_cpus (not --cpus) in the config to cap per-process CPU allocation. Avoid special characters in file names (underscores are safe), because they can silently prevent processes from executing.

## Related tools

- **Nextflow** (Workflow engine that orchestrates the two containerized processes (MS-DIAL then MS-FLO), supports Docker/Singularity profiles, multiple schedulers, and emits execution_report.html, execution_timeline.html, and logs) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Docker** (Container runtime used by the `docker` profile for local execution, configured in conf/base.config)
- **Singularity** (Container runtime used by the `singularity` profile for high-performance computing environments (e.g. HiPerGator via conf/HiPerGator.config))
- **MS-DIAL** (First containerized process; performs peak detection, alignment, and optional metabolite identification on .mzML/.abf data using msdial_params.txt)
- **MS-FLO** (Second containerized process; performs duplicate peak flagging, adduct/isotope flagging, peak alignment, and contaminant ion removal on peak tables transferred from MS-DIAL)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- execution_report.html, execution_timeline.html, and logs/execution.log all exist after a successful run
- logs/execution.log contains Nextflow and workflow versions, parameter settings, resource allocation, container information, and process-level logs
- Repeated executions (e.g. the paper's five runs on MTBLS733) produce identical results, demonstrating consistent reproducibility
- MS-DIAL identifies the expected number of features on the reference dataset (25,447 peaks on MTBLS733's 10 samples)
- No process is silently skipped — file names contain no special characters (underscores only)

## Limitations

- MS-FLO must be downloaded from the Internet rather than bundled with the workflow
- Workflow components such as MS-DIAL are updated frequently; updates are monitored manually with no efficient maintenance plan for workflow and containers
- Not yet tested on Apple silicon (M-series) systems — only Intel-based macOS and Red Hat Enterprise Linux 8.8 (HiPerGator) are validated
- Slurm resource errors can occur if --cpus is used instead of --max_cpus to define per-process CPU availability

## Evidence

- [intro] Nextflow, a widely adopted workflow development tool, facilitates reproducible workflows by supporting software containers like Docker and Singularity.: "Nextflow, a widely adopted workflow development tool, facilitates reproducible workflows by supporting software containers like Docker and Singularity."
- [abstract] The workflow supports software containerization, ensuring computational reproducibility and enabling collaborative research. Nextflow4MS-DIAL is compatible with any Unix-like system and supports: "The workflow supports software containerization, ensuring computational reproducibility and enabling collaborative research. Nextflow4MS-DIAL is compatible with any Unix-like system and supports"
- [readme] nextflow run main.nf -profile functional_test > logs/execution.log: "nextflow run main.nf -profile functional_test > logs/execution.log"
- [readme] Use the `docker` profile for local execution and the `singularity` profile for high-performance computing environments: "Use the `docker` profile for local execution and the `singularity` profile for high-performance computing environments"
- [readme] Use `--max_cpus`, not `--cpus`, in the configuration file to define the CPUs available to each process.: "Use `--max_cpus`, not `--cpus`, in the configuration file to define the CPUs available to each process."
- [readme] It includes metadata such as the Nextflow and workflow versions, parameter settings, resource allocation, container information, workflow progress, and process-level logs.: "It includes metadata such as the Nextflow and workflow versions, parameter settings, resource allocation, container information, workflow progress, and process-level logs."
- [methods] Let Nextflow orchestrate the two containerized processes, MS-DIAL followed by MS-FLO, via the Docker profile.: [evidence span withheld — anchor status: not_found]
