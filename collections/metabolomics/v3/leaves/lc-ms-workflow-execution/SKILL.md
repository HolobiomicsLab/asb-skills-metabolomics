---
name: lc-ms-workflow-execution
description: Use when raw LC-MS data (.mzML or .abf) must be processed reproducibly
  end-to-end — peak detection through filtering, alignment, and contaminant removal
  — on a Unix-like machine, container, or HPC/SLURM cluster.
when_to_use_negative:
- Data are in vendor formats other than .mzML/.abf — convert with ProteoWizard msConvert
  or Reifycs Abf Converter first; the workflow does not accept them directly
- The goal is statistical analysis, pathway analysis, or format conversion beyond
  the peak table — the article explicitly lists these as not yet incorporated
- A graphical interface is required — the workflow is command-line-based only
- Running on Apple silicon (M-series) — the workflow has been tested only on Intel-based
  MacOS and Linux
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine that orchestrates the MS-DIAL -> MS-FLO processes, resolves
    the chosen profile (docker/singularity/functional_test/HiPerGator), and captures
    retrospective provenance in task work directories and execution reports
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: 'First containerized process: performs peak detection (m/z, retention time,
    intensity) and optional metabolite identification on .mzML input, parameterized
    by msdial_params.txt'
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-FLO
  role: Second containerized process that automatically receives MS-DIAL peak tables
    and performs duplicate peak flagging, adduct and isotope flagging, peak alignment,
    and contaminant ion removal, parameterized by msflo_params.ini
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Docker
  role: Containerization backend for local execution via the '-profile docker' profile,
    configured in conf/base.config
- name: Singularity
  role: Containerization backend for high-performance computing execution via the
    '-profile singularity' profile (e.g., HiPerGator/SLURM), configured in conf/HiPerGator.config
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:e8c80ca25f2acb3e39a3460351f37ff7ff184042dee90f5f013458e98d8534f1
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lc-ms-workflow-execution

## Summary

Execute the Nextflow4MS-DIAL two-process pipeline (containerized MS-DIAL followed by MS-FLO) to convert raw LC-HRMS mzML/.abf files into filtered, aligned peak tables with Nextflow's retrospective provenance reports.

## When to use

Use when raw LC-MS data (.mzML or .abf) must be processed reproducibly end-to-end — peak detection through filtering, alignment, and contaminant removal — on a Unix-like machine, container, or HPC/SLURM cluster.

## When NOT to use

- Data are in vendor formats other than .mzML/.abf — convert with ProteoWizard msConvert or Reifycs Abf Converter first; the workflow does not accept them directly
- The goal is statistical analysis, pathway analysis, or format conversion beyond the peak table — the article explicitly lists these as not yet incorporated
- A graphical interface is required — the workflow is command-line-based only
- Running on Apple silicon (M-series) — the workflow has been tested only on Intel-based MacOS and Linux

## Inputs

- Raw LC-HRMS data files in .mzML or .abf format, placed in data/raw_data/
- MS-DIAL parameter file (msdial_params.txt) and MS-FLO parameter file (msflo_params.ini) in data/
- Optional MS1 (ms1_lib.txt) and MS2 (ms2_lib.msp) metabolite identification libraries
- Nextflow >=22.10.0 and Java 11+ runtime environment

## Outputs

- MS-DIAL peak table (.msdial, renameable to .tsv for spreadsheet software)
- MS-FLO filtered/aligned peak table (CSV output, EDAM:format_3752)
- Nextflow retrospective provenance: execution_report.html (runtime and resource usage), execution_timeline.html (per-process timeline), and logs/execution.log (versions, parameters, resource allocation, container information)
- Intermediate results and data files stored in each Nextflow task working directory

## How to apply

Clone github.com/Nextflow4Metabolomics/nextflow4ms-dial and place .mzML (or .abf) files in data/raw_data/, converting other vendor formats first with ProteoWizard msConvert (or Reifycs Abf Converter for .abf). Add MS-DIAL and MS-FLO parameter files named msdial_params.txt and msflo_params.ini in data/, plus ms1_lib.txt and ms2_lib.msp if metabolite identification is needed (it is optional). Run 'nextflow run main.nf -profile docker' locally or '-profile singularity' on HPC (HPC allocation is configured in conf/HiPerGator.config; use --max_cpus, not --cpus, for per-process CPU limits under SLURM). Nextflow orchestrates MS-DIAL for peak detection (m/z, retention time, intensity) then automatically transfers the peak table to MS-FLO for duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal. Decide between profiles by environment: docker for local execution, singularity for systems where Docker is unavailable; confirm the reference file in conf/base.config and msdial_params.txt before running your own data.

## Related tools

- **Nextflow** (Workflow engine that orchestrates the MS-DIAL -> MS-FLO processes, resolves the chosen profile (docker/singularity/functional_test/HiPerGator), and captures retrospective provenance in task work directories and execution reports) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (First containerized process: performs peak detection (m/z, retention time, intensity) and optional metabolite identification on .mzML input, parameterized by msdial_params.txt) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-FLO** (Second containerized process that automatically receives MS-DIAL peak tables and performs duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal, parameterized by msflo_params.ini) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Docker** (Containerization backend for local execution via the '-profile docker' profile, configured in conf/base.config)
- **Singularity** (Containerization backend for high-performance computing execution via the '-profile singularity' profile (e.g., HiPerGator/SLURM), configured in conf/HiPerGator.config)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- execution_report.html, execution_timeline.html, and logs/execution.log all exist after the run, and the log contains Nextflow/workflow versions, parameter settings, resource-allocation, and container metadata
- Against the MTBLS733 reference dataset (Li et al. 2018), MS-DIAL reports 25,447 total peaks (1,031 true positives) and MS-FLO 25,414 total peaks (1,030 true positives), matching the GitHub 'results' folder
- Repeated executions (the paper ran five) produce identical peak tables, demonstrating consistent reproducibility
- Cross-platform check: docker run on macOS 13.5.1 and singularity run on HiPerGator (RHEL 8.8, SLURM, 20 CPUs / 200 GB) yield identical peak counts
- File-name hygiene: any process unexpectedly not executed should prompt a check for special characters in file names (underscores are safe)

## Limitations

- Command-line only; no graphical user interface for users with limited computational experience
- Not yet tested on Apple silicon (M-series) systems — only Intel-based MacOS and Linux (RHEL 8.8) are validated
- MS-FLO must be downloaded from the Internet rather than bundled with the workflow; workflow/container component updates (e.g., MS-DIAL 4.48) are monitored manually with no efficient maintenance plan
- Validation used a small test set (10 samples) and reproducibility/scalability was not quantitatively measured
- A common SLURM failure ('Process requirement exceed available CPUs') arises if --cpus instead of --max_cpus is used to define per-process CPU limits

## Evidence

- [other] Retrospective provenance in Nextflow is captured through task execution reports, intermediate results, and data files stored within each task's working directory.: [evidence span withheld — anchor status: not_found]
