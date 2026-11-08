---
name: nextflow-pipeline-configuration
description: Use when you have raw .mzML or .abf LC-MS metabolomics files, need to
  run the MS-DIAL → MS-FLO workflow, and must set up container profiles, parameter
  files, and resource allocations for local or HPC execution.
when_to_use_negative:
- When you already have a processed feature/peak table and only need downstream statistical
  analysis, rather than LC-MS raw-file processing
- When the data are in a format other than .mzML or .abf (e.g. proprietary vendor
  formats not yet converted), before conversion via ProteoWizard msConvert or Reifycs
  Abf Converter
- When operating on Apple silicon (M-series) systems, which the workflow has not yet
  been tested on
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: Nextflow4MS-DIAL
  role: The workflow whose configuration files and profiles are set up to run MS-DIAL
    and MS-FLO reproducibly.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Orchestration engine that reads the profile, parameter, and configuration
    files to launch the pipeline.
  repo: https://nf-co.re/usage/installation
- name: Docker
  role: Container engine used via the docker profile for local execution on macOS
    or Linux.
  repo: https://docs.docker.com/engine/installation/
- name: Singularity
  role: Container engine used via the singularity profile on high-performance computing
    systems such as HiPerGator.
  repo: https://www.sylabs.io/guides/3.0/user-guide/
- name: MS-DIAL
  role: The LC-MS processing tool whose parameters are supplied in msdial_params.txt.
- name: MS-FLO
  role: The downstream optimization tool whose parameters are supplied in msflo_params.ini.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:f854b6afa29fbacbfdd02fff87483f671ce8c4a8f23b54d54d3ca839a38d5ac9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  - Singularity
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# nextflow-pipeline-configuration

## Summary

Configure the Nextflow4MS-DIAL pipeline by supplying MS-DIAL and MS-FLO parameter files, MS1/MS2 library files, container profiles (Docker or Singularity), and hardware resource settings so LC-MS metabolomics data can be processed reproducibly.

## When to use

Use when you have raw .mzML or .abf LC-MS metabolomics files, need to run the MS-DIAL → MS-FLO workflow, and must set up container profiles, parameter files, and resource allocations for local or HPC execution.

## When NOT to use

- When you already have a processed feature/peak table and only need downstream statistical analysis, rather than LC-MS raw-file processing
- When the data are in a format other than .mzML or .abf (e.g. proprietary vendor formats not yet converted), before conversion via ProteoWizard msConvert or Reifycs Abf Converter
- When operating on Apple silicon (M-series) systems, which the workflow has not yet been tested on

## Inputs

- Raw LC-MS data files in .mzML or .abf format placed in data/raw_data/
- MS-DIAL parameter file (msdial_params.txt)
- MS-FLO parameter file (msflo_params.ini)
- MS1 library file (ms1_lib.txt)
- MS2 library file (ms2_lib.msp)
- Nextflow configuration profile (docker or singularity)
- Reference file defined in conf/base.config

## Outputs

- MS-DIAL peak table (.msdial, renamed to .tsv for spreadsheet use)
- MS-FLO processed peak table
- execution_report.html summarizing workflow runtime and computational resource usage
- execution_timeline.html showing the execution timeline for each process
- logs/execution.log containing metadata such as Nextflow and workflow versions, parameter settings, resource allocation, and container information

## How to apply

Clone the Nextflow4MS-DIAL repository and place raw .mzML or .abf files into data/raw_data/. Add MS-DIAL and MS-FLO configuration files named msdial_params.txt and msflo_params.ini to the data/ folder, and MS1/MS2 library files named ms1_lib.txt and ms2_lib.msp. Confirm that the reference file in conf/base.config matches the MS-DIAL configuration before processing your own data. Choose a profile: use `-profile docker` for local execution (e.g. macOS 13.5.1 with 16 GB memory) or `-profile singularity` for high-performance computing (e.g. HiPerGator running Red Hat Enterprise Linux 8.8, where 20 CPUs and 200 GB memory are allocated through SLURM). Use `--max_cpus`, not `--cpus`, to define the CPUs available to each process, and direct execution logs to logs/execution.log for later review of runtime and resource usage.

## Related tools

- **Nextflow4MS-DIAL** (The workflow whose configuration files and profiles are set up to run MS-DIAL and MS-FLO reproducibly.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Orchestration engine that reads the profile, parameter, and configuration files to launch the pipeline.) — https://nf-co.re/usage/installation
- **Docker** (Container engine used via the docker profile for local execution on macOS or Linux.) — https://docs.docker.com/engine/installation/
- **Singularity** (Container engine used via the singularity profile on high-performance computing systems such as HiPerGator.) — https://www.sylabs.io/guides/3.0/user-guide/
- **MS-DIAL** (The LC-MS processing tool whose parameters are supplied in msdial_params.txt.)
- **MS-FLO** (The downstream optimization tool whose parameters are supplied in msflo_params.ini.)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- The execution completes without the Slurm error 'Process requirement exceed available CPUs -- req: 5; avail: 3', indicating --max_cpus was used correctly
- logs/execution.log records the expected Nextflow version, workflow version, parameter settings, resource allocation, and container information
- MS-DIAL and MS-FLO peak tables match the reference values reported in the GitHub results folder (MS-DIAL 25,447 peaks / 1,031 true positives; MS-FLO 25,414 peaks / 1,030 true positives)
- Repeated executions on the same platform produce identical peak counts and true-positive counts
- execution_report.html and execution_timeline.html are generated and reflect the configured CPU and memory allocations

## Limitations

- MS-FLO must be downloaded from the Internet rather than bundled with the workflow
- Software updates such as MS-DIAL are monitored manually, with no efficient maintenance plan for the workflow and container
- The workflow is command-line based and lacks a graphical user interface for users with limited computational experience
- Only Apple silicon (M-series) systems remain untested, although Intel-based MacOS has been tested successfully
- Special characters in file names cause processes to be skipped; only underscores are safe

## Evidence
