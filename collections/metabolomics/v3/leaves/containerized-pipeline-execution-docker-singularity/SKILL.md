---
name: containerized-pipeline-execution-docker-singularity
description: Use when processing raw LC-HRMS metabolomics files (.mzML or .abf) with
  MS-DIAL and MS-FLO and you need containerized, reproducible execution locally or
  on Slurm-based HPC.
when_to_use_negative:
- The input is already a processed peak/feature table and only downstream statistical
  or pathway analysis is needed — the workflow covers MS-DIAL → MS-FLO processing
  of raw data only
- Your computing environment is Windows or Apple silicon (M-series) — the workflow
  has only been tested on Intel-based macOS and Red Hat Enterprise Linux 8.8 (HiPerGator)
- You cannot use containers (no Docker or Singularity available) or the environment
  is a web-based tool such as MetaboAnalyst or Workflow4Metabolomics — the skill is
  specifically about local/containerized execution of downloadable tools
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0091
tools:
- name: Nextflow4MS-DIAL
  role: The reproducible Nextflow workflow that wraps MS-DIAL and MS-FLO in containers
    for LC-MS metabolomics data processing
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine that runs main.nf with the docker or singularity profile and
    records execution provenance
- name: MS-DIAL
  role: Containerized tool performing peak detection, noise/contaminant filtration,
    m/z and retention time identification, and peak intensity calculation on .mzML
    data
- name: MS-FLO
  role: Containerized tool receiving MS-DIAL peak tables for duplicate peak flagging,
    adduct and isotope flagging, peak alignment and contaminant ion removal
- name: Docker
  role: Container runtime used for local execution via the docker profile
- name: Singularity
  role: Container runtime used for high-performance computing (HiPerGator) via the
    singularity profile
- name: ProteoWizard msConvert
  role: Used beforehand to convert other raw data formats to .mzML accepted by the
    workflow
  repo: https://proteowizard.sourceforge.io/download.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:e65fdddb0e44966e10869379ffda6391fdc2df5fe04f4a5c8cdf6c8b1dbce985
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
  - ProteoWizard msConvert
  - Singularity
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# containerized-pipeline-execution-docker-singularity

## Summary

Run the Nextflow4MS-DIAL LC-MS metabolomics workflow inside Docker or Singularity containers so MS-DIAL v4.48 and MS-FLO execute identically on a local machine or an HPC cluster, yielding reproducible peak tables and execution reports.

## When to use

Use when processing raw LC-HRMS metabolomics files (.mzML or .abf) with MS-DIAL and MS-FLO and you need containerized, reproducible execution locally or on Slurm-based HPC.

## When NOT to use

- The input is already a processed peak/feature table and only downstream statistical or pathway analysis is needed — the workflow covers MS-DIAL → MS-FLO processing of raw data only
- Your computing environment is Windows or Apple silicon (M-series) — the workflow has only been tested on Intel-based macOS and Red Hat Enterprise Linux 8.8 (HiPerGator)
- You cannot use containers (no Docker or Singularity available) or the environment is a web-based tool such as MetaboAnalyst or Workflow4Metabolomics — the skill is specifically about local/containerized execution of downloadable tools

## Inputs

- Raw LC-MS data files in .mzML or .abf format placed in data/raw_data/
- MS-DIAL parameter file msdial_params.txt
- MS-FLO parameter file msflo_params.ini
- MS1 library file ms1_lib.txt
- MS2 library file ms2_lib.msp
- Container runtime: Docker (local) or Singularity (HPC)
- Nextflow workflow main.nf from the Nextflow4MS-DIAL repository

## Outputs

- MS-DIAL peak table (.msdial, renameable to .tsv for spreadsheets)
- MS-FLO processed peak table after duplicate/adduct/isotope flagging, alignment and contaminant ion removal
- Nextflow execution_report.html with runtime and resource usage
- Nextflow execution_timeline.html with per-process timeline
- logs/execution.log with version, parameter, container and process metadata

## How to apply

Install Java 11 or later, Nextflow (≥22.10.0), and either Docker (local) or Singularity (HPC). Clone the Nextflow4MS-DIAL repository, place .mzML/.abf files in data/raw_data/ and the msdial_params.txt, msflo_params.ini, ms1_lib.txt and ms2_lib.msp configuration/library files in data/. For local execution run 'nextflow run main.nf -profile docker'; on the University of Florida HiPerGator Slurm cluster use the singularity profile with 'nextflow run main.nf -profile singularity', allocating 20 CPUs and 200 GB memory and setting per-process CPU limits with --max_cpus (not --cpus) to avoid the 'Process requirement exceed available CPUs' Slurm error. Redirect stdout to logs/execution.log and afterwards collect execution_report.html, execution_timeline.html and the log as retrospective provenance. The rationale is that containerization pins MS-DIAL (v4.48) and MS-FLO binaries so that five repeated runs produced identical peak counts.

## Related tools

- **Nextflow4MS-DIAL** (The reproducible Nextflow workflow that wraps MS-DIAL and MS-FLO in containers for LC-MS metabolomics data processing) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine that runs main.nf with the docker or singularity profile and records execution provenance)
- **MS-DIAL** (Containerized tool performing peak detection, noise/contaminant filtration, m/z and retention time identification, and peak intensity calculation on .mzML data)
- **MS-FLO** (Containerized tool receiving MS-DIAL peak tables for duplicate peak flagging, adduct and isotope flagging, peak alignment and contaminant ion removal)
- **Docker** (Container runtime used for local execution via the docker profile)
- **Singularity** (Container runtime used for high-performance computing (HiPerGator) via the singularity profile)
- **ProteoWizard msConvert** (Used beforehand to convert other raw data formats to .mzML accepted by the workflow) — https://proteowizard.sourceforge.io/download.html

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Five repeated executions produce identical results (identical peak tables and counts)
- MS-DIAL output contains ~25,447 total peaks; MS-FLO output contains ~25,414 peaks with 1,030 true positives (4.05%) on the MTBLS733 test data
- logs/execution.log records the Nextflow and workflow versions, parameter settings, resource allocation and container information
- execution_report.html and execution_timeline.html are generated and summarize runtime and per-process resource usage
- Produced .msdial files, when renamed to .tsv, open in spreadsheet software with coherent peak columns

## Limitations

- MS-FLO must be downloaded from the Internet rather than bundled with the workflow
- Software components such as MS-DIAL are updated frequently and updates are monitored manually, with no efficient maintenance plan for the workflow and container
- Scalability and reproducibility were not quantitatively measured beyond the five-run demonstration
- The workflow is command-line-based; no graphical user interface exists yet
- Not yet tested on Apple silicon (M-series) systems
- Special characters in file names cause processes not to execute; use underscores
- SLurm CPU allocation errors can occur if --cpus is used instead of --max_cpus in the configuration

## Evidence

- [abstract] The workflow supports software containerization, ensuring computational reproducibility and enabling collaborative research.: "The workflow supports software containerization, ensuring computational reproducibility and enabling collaborative research."
