---
name: nextflow-workflow-execution
description: Use when you need to execute an end-to-end untargeted LC-HRMS metabolomics
  feature-detection workflow reproducibly from raw .mzML files, with containerized
  software (MS-DIAL + MS-FLO) under Nextflow on a local machine or HPC (SLURM).
when_to_use_negative:
- Input is already a processed feature/peak table rather than raw .mzML/.abf data
  — the workflow expects vendor-converted mzML from ProteoWizard msConvert or Reifycs
  Abf Converter.
- You need a graphical user interface — the workflow is command-line-based only.
- You are on Apple silicon (M-series) macOS — only Intel-based MacOS was tested.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0154
tools:
- name: Nextflow
  role: Workflow engine that orchestrates MS-DIAL and MS-FLO as processes with containerized
    execution and retrospective provenance reporting.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Containerized (Docker/Singularity) LC-MS peak-detection engine producing the
    .msdial peak table handed to MS-FLO.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-FLO
  role: Downstream tool executed automatically after MS-DIAL for duplicate peak flagging,
    adduct/isotope flagging, peak alignment and contaminant ion removal.
- name: Docker
  role: Local container runtime profile (`-profile docker`) encapsulating MS-DIAL
    v4.48 and MS-FLO for computational reproducibility.
- name: Singularity
  role: Container runtime for the HPC profile on SLURM-executed systems such as HiPerGator.
- name: SLURM
  role: Job scheduler on HiPerGator used with the singularity profile to allocate
    resources (20 CPUs, 200 GB memory).
- name: ProteoWizard msConvert
  role: Pre-step used to convert vendor raw data formats to .mzML before workflow
    input.
- name: Reifycs Abf Converter
  role: Pre-step to create .abf files if raw data are in that vendor format.
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:9194f75f77670662ec3b5e42e2ba7184e3fda3b1fccbe8826cd5ff445631e8ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - ProteoWizard msConvert
  - Reifycs Abf Converter
  - SLURM
  - Singularity
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nextflow-workflow-execution

## Summary

Run the containerized Nextflow4MS-DIAL LC-MS metabolomics workflow (MS-DIAL v4.48 + MS-FLO in a Docker/Singularity container) on mzML input via `nextflow run main.nf -profile docker`, producing reproducible .msdial peak tables and provenance artifacts.

## When to use

Use when you need to execute an end-to-end untargeted LC-HRMS metabolomics feature-detection workflow reproducibly from raw .mzML files, with containerized software (MS-DIAL + MS-FLO) under Nextflow on a local machine or HPC (SLURM).

## When NOT to use

- Input is already a processed feature/peak table rather than raw .mzML/.abf data — the workflow expects vendor-converted mzML from ProteoWizard msConvert or Reifycs Abf Converter.
- You need a graphical user interface — the workflow is command-line-based only.
- You are on Apple silicon (M-series) macOS — only Intel-based MacOS was tested.

## Inputs

- .mzML or .abf raw LC-HRMS files (e.g. 10-sample MTBLS733 dataset, five samples per group)
- msdial_params.txt configuration file placed in data/
- msflo_params.ini configuration file placed in data/
- ms1_lib.txt MS1 metabolite library in data/
- ms2_lib.msp MS2 library in data/
- reference file configured in conf/base.config

## Outputs

- MS-DIAL peak table (.msdial, renameable to .tsv for spreadsheet viewing; e.g. 25,447 peaks)
- MS-FLO processed peak table (e.g. 25,414 peaks after alignment and contaminant removal)
- logs/execution.log with version, parameter, container and process metadata
- execution_report.html runtime/resource summary
- execution_timeline.html per-process timeline
- error.txt log from failed executions

## How to apply

Clone the GitHub repository, place ten-sample .mzML raw LC-MS data in data/raw_data/, and add msdial_params.txt, msflo_params.ini, ms1_lib.txt and ms2_lib.msp to data/ (examples in functional_test/sample_data/). Launch the pipeline with `nextflow run main.nf -profile docker` locally or the `singularity` profile with SLURM on HPC (HiPerGator: allocate 20 CPUs, 200 GB memory per process, and use --max_cpus, not --cpus, to cap per-process resources). The workflow runs MS-DIAL v4.48 for peak detection, noise/contaminant filtration, m/z + retention-time identification and intensity calculation, then automatically transfers the peak table to MS-FLO for duplicate flagging, adduct/isotope flagging, peak alignment and contaminant-ion removal — no manual copy step. Require Java ≥ 11 (developed with Java 11.0.8), Nextflow ≥ 22.10.0, and avoid special characters in file names (underscores are safe) to prevent processes being silently skipped.

## Related tools

- **Nextflow** (Workflow engine that orchestrates MS-DIAL and MS-FLO as processes with containerized execution and retrospective provenance reporting.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Containerized (Docker/Singularity) LC-MS peak-detection engine producing the .msdial peak table handed to MS-FLO.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-FLO** (Downstream tool executed automatically after MS-DIAL for duplicate peak flagging, adduct/isotope flagging, peak alignment and contaminant ion removal.)
- **Docker** (Local container runtime profile (`-profile docker`) encapsulating MS-DIAL v4.48 and MS-FLO for computational reproducibility.)
- **Singularity** (Container runtime for the HPC profile on SLURM-executed systems such as HiPerGator.)
- **SLURM** (Job scheduler on HiPerGator used with the singularity profile to allocate resources (20 CPUs, 200 GB memory).)
- **ProteoWizard msConvert** (Pre-step used to convert vendor raw data formats to .mzML before workflow input.)
- **Reifycs Abf Converter** (Pre-step to create .abf files if raw data are in that vendor format.)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Peak table row counts match the article's reported values: MS-DIAL 25,447 total peaks with 1,031 true positives (4.05%); MS-FLO 25,414 peaks with 1,030 true positives (4.05%).
- Re-run the workflow five times with identical inputs and confirm byte-identical/checksum-equal peak tables across all runs.
- logs/execution.log must record Nextflow and workflow versions, parameter settings, resource allocation, container information and per-process logs for every successful run.
- Confirm MS-FLO output exists and no manual file copying was needed between MS-DIAL and MS-FLO stages.
- execution_report.html shows runtime and computational resource usage for each process and execution_timeline.html shows the per-process timeline.

## Limitations

- MS-FLO must be downloaded from the Internet rather than bundled with the workflow container.
- Workflow components such as MS-DIAL are updated frequently and updates are monitored manually, with no efficient maintenance plan for the workflow and container.
- Not tested on Apple silicon (M-series) systems.
- Special characters in file names can cause a process to be silently skipped; use only underscores and alphanumeric characters.
- Only 10-sample test data (MTBLS733) were used, so scalability beyond that is untested.
- The workflow was not quantitatively measured for scalability or reproducibility in the article.

## Evidence
