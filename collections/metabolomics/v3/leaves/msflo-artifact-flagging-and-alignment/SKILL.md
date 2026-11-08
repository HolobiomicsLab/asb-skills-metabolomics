---
name: msflo-artifact-flagging-and-alignment
description: Use when you have a raw MS-DIAL peak table (from .mzML or .abf LC-HRMS
  metabolomics files) and need to flag duplicates/adducts/isotopes, align peaks, and
  remove contaminant ions before statistical analysis.
when_to_use_negative:
- You already have a curated, aligned feature table and only need statistical analysis—the
  skill is for artifact flagging/alignment, not downstream statistics.
- Data are not in .mzML or .abf format and cannot be converted (e.g., use ProteoWizard
  msConvert or Reifycs Abf Converter first).
- You want interactive visual inspection of peaks rather than automated, reproducible
  batch processing—this workflow is command-line only with no GUI.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MS-FLO
  role: Command-line tool that receives the MS-DIAL peak table and performs duplicate/adduct/isotope
    flagging, peak alignment, and contaminant ion removal
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow4MS-DIAL
  role: Nextflow workflow that containerizes and orchestrates the MS-DIAL→MS-FLO step
    chain, with Docker/Singularity profiles
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Upstream peak detection and m/z / retention-time identification; produces
    the peak table handed to MS-FLO
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:3b2c304d33d552ff9c140c75045033941e51831bc94758e80034e65f1297734e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow4MS-DIAL
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# MS-FLO artifact flagging and peak alignment

## Summary

A downstream post-processing stage of the Nextflow4MS-DIAL pipeline in which the MS-DIAL-generated peak table is passed to the command-line MS-FLO to flag duplicate peaks, adducts, and isotopes, align peaks, and remove contaminant ions, improving data quality and accuracy for LC-MS metabolomics feature tables.

## When to use

Use when you have a raw MS-DIAL peak table (from .mzML or .abf LC-HRMS metabolomics files) and need to flag duplicates/adducts/isotopes, align peaks, and remove contaminant ions before statistical analysis.

## When NOT to use

- You already have a curated, aligned feature table and only need statistical analysis—the skill is for artifact flagging/alignment, not downstream statistics.
- Data are not in .mzML or .abf format and cannot be converted (e.g., use ProteoWizard msConvert or Reifycs Abf Converter first).
- You want interactive visual inspection of peaks rather than automated, reproducible batch processing—this workflow is command-line only with no GUI.

## Inputs

- .mzML or .abf LC-HRMS raw metabolomics files (placed in `data/raw_data/`)
- MS-DIAL peak table automatically transferred from the MS-DIAL process
- `msflo_params.ini` MS-FLO configuration file
- `msdial_params.txt` MS-DIAL configuration file
- optional `ms1_lib.txt` and `ms2_lib.msp` metabolite library files

## Outputs

- MS-FLO processed peak table (~25,414 peaks for the MTBLS733 test set)
- flagged duplicate/adduct/isotope annotations and aligned peak intensities
- Nextflow retrospective provenance artifacts (`execution_report.html`, `execution_timeline.html`, `logs/execution.log`)

## How to apply

In Nextflow4MS-DIAL, once MS-DIAL v4.48 finishes peak detection, filtering of noise/contaminants, and m/z/retention-time identification on .mzML input, the resulting peak table is automatically transferred to MS-FLO—no manual hand-off is needed. MS-FLO runs from the command line with its parameters set in a `data/msflo_params.ini` file (examples ship in `functional_test/sample_data/`) and applies four refinement steps: duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal. Metabolite identification is optional and requires `data/ms1_lib.txt` and `data/ms2_lib.msp` library files if used. The workflow was parameterized to match the Li et al. 2018 protocol (MetaboLights MTBLS733) and validated on 10 samples (five per group); the containerized profile is run with `nextflow run main.nf -profile docker` locally or `-profile singularity` with SLURM on HiPerGator (20 CPUs, 200 GB memory).

## Related tools

- **MS-FLO** (Command-line tool that receives the MS-DIAL peak table and performs duplicate/adduct/isotope flagging, peak alignment, and contaminant ion removal) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow4MS-DIAL** (Nextflow workflow that containerizes and orchestrates the MS-DIAL→MS-FLO step chain, with Docker/Singularity profiles) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Upstream peak detection and m/z / retention-time identification; produces the peak table handed to MS-FLO) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Repeated executions of the workflow produce identical results (five runs gave the same peak counts).
- For the MTBLS733 Li et al. test set, MS-FLO should report ~25,414 peaks with 1,030 true positives (≈4.05%), vs. MS-DIAL's ~25,447 peaks / 1,031 true positives.
- Peak counts are rounded to integers and are comparable to published example results in the repo's `results` folder.
- Provenance artifacts exist and reflect the run: `execution_report.html`, `execution_timeline.html`, and `logs/execution.log` contain Nextflow/workflow versions, parameter settings, and container info.
- Files named `msflo_params.ini`, `ms1_lib.txt`, and `ms2_lib.msp` are present in the `data/` folder and correctly referenced.

## Limitations

- MS-FLO must be downloaded from the Internet; it is not bundled inside the workflow container.
- The published validation used only a small 10-sample dataset; performance on much larger cohorts was not quantitatively assessed.
- Workflow components like MS-DIAL are updated frequently but monitored manually—container versions may lag upstream releases.
- Provenance FAIRness (e.g., WorkflowHub deposition) is planned but not yet implemented.
- Command-line only: users without computational experience may struggle; Apple silicon (M-series) Macs are untested.
- Output `.msdial` files should be renamed to `.tsv` for spreadsheet viewing; file names must avoid special characters or a process may silently be skipped.

## Evidence
