---
name: msdial-peak-detection-parameter-configuration
description: 'Use when preparing an LC-HRMS metabolomics run with Nextflow4MS-DIAL:
  mzML (or .abf) files must be processed through MS-DIAL v4.48 with parameter sets
  matched to the original study (e.g. Li et al. 2018 / MTBLS733) and the resulting
  peak table handed to MS-FLO.'
when_to_use_negative:
- Your raw data is already a processed peak table and you only want downstream statistical
  or pathway analysis (the workflow's future-work section notes such tools are not
  yet incorporated).
- You need to convert raw vendor formats other than mzML or .abf — first convert with
  ProteoWizard msConvert; this skill configures MS-DIAL, not format conversion.
- You are on Apple silicon (M-series) macOS — the workflow has only been tested on
  Intel-based macOS and Red Hat Enterprise Linux 8.8 (HiPerGator).
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL
  role: Performs comprehensive LC-focused peak detection, filtration of noise and
    contaminants, m/z and retention-time identification, peak-intensity calculation,
    and optional metabolite identification using ms1_lib.txt and ms2_lib.msp.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-FLO
  role: Receives the MS-DIAL peak table automatically for duplicate peak flagging,
    adduct and isotope flagging, peak alignment, and contaminant ion removal.
- name: Nextflow
  role: 'Orchestrates the containerized MS-DIAL→MS-FLO pipeline (profiles: docker
    locally, singularity with SLURM on HiPerGator allocating 20 CPUs and 200 GB memory).'
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: ProteoWizard msConvert
  role: Used to convert other raw data formats to .mzML before MS-DIAL processing.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:9ce40e98ae9eddb1c927d47684919db230a3369644562ffb96fa6dcfd5b073e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - ProteoWizard msConvert
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# msdial-peak-detection-parameter-configuration

## Summary

Configure the MS-DIAL command-line parameters for LC-HRMS metabolomics peak detection, m/z and retention-time identification, and peak-intensity calculation so the workflow reproduces the original study's processed peak counts when applied to a provided dataset.

## When to use

Use when preparing an LC-HRMS metabolomics run with Nextflow4MS-DIAL: mzML (or .abf) files must be processed through MS-DIAL v4.48 with parameter sets matched to the original study (e.g. Li et al. 2018 / MTBLS733) and the resulting peak table handed to MS-FLO.

## When NOT to use

- Your raw data is already a processed peak table and you only want downstream statistical or pathway analysis (the workflow's future-work section notes such tools are not yet incorporated).
- You need to convert raw vendor formats other than mzML or .abf — first convert with ProteoWizard msConvert; this skill configures MS-DIAL, not format conversion.
- You are on Apple silicon (M-series) macOS — the workflow has only been tested on Intel-based macOS and Red Hat Enterprise Linux 8.8 (HiPerGator).

## Inputs

- mzML LC-MS raw data files (or .abf files from Reifycs Abf Converter) placed in data/raw_data/
- msdial_params.txt — MS-DIAL parameter configuration file (parameters aligned to the original study)
- msflo_params.ini — MS-FLO parameter configuration file
- ms1_lib.txt and ms2_lib.msp — metabolite library files (optional, for metabolite identification)
- conf/base.config with correct reference file and Docker/Singularity settings

## Outputs

- MS-DIAL peak table (~25,447 peaks for the MTBLS733 test set; .msdial output, renamed to .tsv for spreadsheet viewing)
- Downstream MS-FLO processed peak table (~25,414 peaks)
- Nextflow execution_report.html and execution_timeline.html artifacts

## How to apply

Place the mzML raw files in data/raw_data/ and drop the two configuration files into data/: msdial_params.txt for MS-DIAL and msflo_params.ini for MS-FLO, plus the metabolite library files ms1_lib.txt and ms2_lib.msp. Parameter values should be aligned to the original study whose results are being reproduced (e.g. Li et al. 2018 MTBLS733); example configurations are available in functional_test/sample_data/ of the Nextflow4MS-DIAL repository. Before processing, confirm the reference file in conf/base.config and the MS-DIAL configuration file are set correctly — using --max_cpus rather than --cpus to define CPUs available per process (e.g. 20 CPUs on HiPerGator) avoids Slurm 'Process requirement exceed available CPUs' errors. Metabolite identification is optional and only activates if ms1_lib.txt / ms2_lib.msp are supplied. Run `nextflow run main.nf -profile docker` (local) or `nextflow run main.nf -profile singularity` (HPC) to execute MS-DIAL v4.48 in a container; the produced peak table is automatically handed to MS-FLO for duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal.

## Related tools

- **MS-DIAL** (Performs comprehensive LC-focused peak detection, filtration of noise and contaminants, m/z and retention-time identification, peak-intensity calculation, and optional metabolite identification using ms1_lib.txt and ms2_lib.msp.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-FLO** (Receives the MS-DIAL peak table automatically for duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal.)
- **Nextflow** (Orchestrates the containerized MS-DIAL→MS-FLO pipeline (profiles: docker locally, singularity with SLURM on HiPerGator allocating 20 CPUs and 200 GB memory).) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **ProteoWizard msConvert** (Used to convert other raw data formats to .mzML before MS-DIAL processing.)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Five repeated executions of the workflow on the same mzML inputs produce identical MS-DIAL and MS-FLO peak counts (the MTBLS733 test set yields 25,447 total peaks from MS-DIAL and 25,414 peaks from MS-FLO with 1,030 true positives at 4.05%).
- The produced .msdial file opens in spreadsheet software after renaming to .tsv and contains the expected number of peak rows for the aligned parameters.
- execution_report.html and execution_timeline.html are generated, recording runtime, computational resource usage, Nextflow and workflow versions, parameter settings, resource allocation, and container information.
- Slurm does not report 'Process requirement exceed available CPUs' — the --max_cpus directive (not --cpus) is set correctly in conf/base.config or conf/HiPerGator.config.
- Metabolite identification output is present only when ms1_lib.txt and ms2_lib.msp were supplied; its absence is expected when the libraries are omitted.

## Limitations

- The test set contained only 10 samples (five per group), so reproducibility on larger or more heterogeneous cohorts is untested.
- Few LC-MS metabolomics datasets had been processed with both MS-DIAL and MS-FLO under transparent parameter settings, limiting the ability to reproduce steps conveniently.
- MS-FLO must be downloaded from the Internet rather than bundled with the workflow.
- Workflow components such as MS-DIAL are updated frequently; updates are currently monitored manually with no efficient maintenance plan for the workflow and container.
- The workflow is command-line only — no graphical user interface is available for users with limited computational experience.
- Not yet tested on Apple silicon (M-series) systems.
- Special characters in file names can cause processes not to execute; underscores are safe.

## Evidence
