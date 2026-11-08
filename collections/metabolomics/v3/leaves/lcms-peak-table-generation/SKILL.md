---
name: lcms-peak-table-generation
description: Use when starting from raw .mzML (or .abf) LC-HRMS metabolomics files
  and you need a reproducible, containerized peak-detection/alignment pipeline that
  yields a filtered MS-DIAL plus MS-FLO peak table (.tsv).
when_to_use_negative:
- Input is already a feature/peak table from a prior peak-detection run rather than
  raw .mzML or .abf spectra
- Raw vendor files that have not been converted (e.g. with ProteoWizard msConvert)
  to .mzML, or that cannot be converted to .abf
- You need a graphical interface — the workflow is command-line only, with no GUI
  for users with limited computational experience
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: Nextflow4MS-DIAL
  role: Nextflow workflow orchestrating containerized MS-DIAL -> MS-FLO peak-table
    generation from .mzML inputs
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Performs LC-focused peak detection, m/z and retention-time identification,
    and peak intensity calculation inside the Docker/Singularity container (v4.48)
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-FLO
  role: Receives the MS-DIAL peak table automatically and applies duplicate/adduct/isotope
    flagging, peak alignment, and contaminant ion removal
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine enabling containerized, reproducible execution across local
    and HPC (e.g. Slurm) environments
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:ea29747898059183c4d6bfc7104d7f29e1ecb3bd64734f30b7bf4835289ce414
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lcms-peak-table-generation

## Summary

Generate feature/peak tables from untargeted LC-MS metabolomics raw data by running containerized MS-DIAL (peak detection, alignment, m/z and retention-time assignment) followed by MS-FLO quality filtering (duplicate, adduct/isotope flagging, contaminant ion removal), as implemented in the Nextflow4MS-DIAL workflow.

## When to use

Use when starting from raw .mzML (or .abf) LC-HRMS metabolomics files and you need a reproducible, containerized peak-detection/alignment pipeline that yields a filtered MS-DIAL plus MS-FLO peak table (.tsv).

## When NOT to use

- Input is already a feature/peak table from a prior peak-detection run rather than raw .mzML or .abf spectra
- Raw vendor files that have not been converted (e.g. with ProteoWizard msConvert) to .mzML, or that cannot be converted to .abf
- You need a graphical interface — the workflow is command-line only, with no GUI for users with limited computational experience

## Inputs

- Raw LC-MS data files in .mzML (or .abf) format placed in data/raw_data/
- MS-DIAL parameter file data/msdial_params.txt
- MS-FLO parameter file data/msflo_params.ini
- Optional MS1 metabolite library data/ms1_lib.txt and MS2 library data/ms2_lib.msp

## Outputs

- MS-DIAL peak table (.msdial, distributed as .tsv) with m/z, retention time, and peak intensities per feature
- MS-FLO filtered peak table (.tsv) with duplicates, adducts/isotopes, and contaminant ions flagged/removed
- Execution logs (logs/execution.log, execution_report.html, execution_timeline.html)

## How to apply

Place converted .mzML raw files in data/raw_data/ and the parameter files msdial_params.txt and msflo_params.ini (plus optional ms1_lib.txt and ms2_lib.msp for metabolite identification) in data/. Run the workflow via Nextflow with the docker profile locally (or singularity on HPC); the Docker container carries MS-DIAL version 4.48 and MS-FLO, and the pipeline automatically transfers the MS-DIAL peak table to MS-FLO for duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal. Confirm conf/base.config references the correct reference file and container settings before running. Collect the resulting .msdial/.tsv peak tables from the results/ directory; a successful MTBLS733-sized run should yield ~25,447 total peaks from MS-DIAL and ~25,414 total peaks with ~1,030 true positives (4.05%) from MS-FLO.

## Related tools

- **Nextflow4MS-DIAL** (Nextflow workflow orchestrating containerized MS-DIAL -> MS-FLO peak-table generation from .mzML inputs) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Performs LC-focused peak detection, m/z and retention-time identification, and peak intensity calculation inside the Docker/Singularity container (v4.48)) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-FLO** (Receives the MS-DIAL peak table automatically and applies duplicate/adduct/isotope flagging, peak alignment, and contaminant ion removal) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine enabling containerized, reproducible execution across local and HPC (e.g. Slurm) environments) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Peak totals match published MTBLS733 reference values: MS-DIAL 25,447 total peaks; MS-FLO 25,414 total peaks with 1,030 true positives (4.05%)
- Output peak tables open as spreadsheet-readable .tsv files with per-sample intensity columns matching the 10 input samples
- logs/execution.log records successful completion, container info (MS-DIAL 4.48), and process-level logs; execution_report.html shows nonzero resource usage for every process
- Repeated execution (five runs, reset work directory) reproduces identical peak tables, as the paper's reproducibility check demonstrated
- No process silently skipped — file names avoid special characters (underscores are safe) per the README FAQ

## Limitations

- MS-FLO must be downloaded from the Internet rather than bundled with the workflow
- Workflow is command-line only; no GUI is available for users with limited computational experience
- Not yet tested on Apple silicon (M-series) systems; tested on Intel macOS and Red Hat Enterprise Linux 8.8 (HiPerGator)
- Workflow components such as MS-DIAL are updated frequently and updates are monitored manually, with no automated maintenance plan for workflow and container
- Validation used only a 10-sample test set; scalability and reproducibility were not quantitatively measured

## Evidence
