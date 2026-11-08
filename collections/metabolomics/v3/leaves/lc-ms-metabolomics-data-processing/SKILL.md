---
name: lc-ms-metabolomics-data-processing
description: Use when raw LC-MS/MS metabolomics data in .mzML (or .abf) format must
  be processed reproducibly via MS-DIAL and MS-FLO, and peak tables with retrospective
  provenance reports are needed.
when_to_use_negative:
- The data is already a processed feature/peak table — the skill operates on raw .mzML/.abf
  chromatograms and will not add value to post-MS-FLO files.
- 'You need a graphical interface or statistical/pathway analysis: Nextflow4MS-DIAL
  is command-line-only and stops after peak table generation; downstream tools must
  be run separately.'
- You cannot use containerization (Docker/Singularity) and need guaranteed bit-level
  reproducibility across machines — the article notes reproducibility depends on the
  containerized profiles.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Nextflow
  role: Orchestrates the MS-DIAL → MS-FLO pipeline, enabling containerization and
    reproducible execution across local, Docker, and HPC/SLURM environments.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: 'Performs comprehensive LC-focused data processing: peak detection, filtration
    of noise and contaminants, m/z and retention time identification, and peak intensity
    calculation.'
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-FLO
  role: 'Post-MS-DIAL optimization: duplicate peak flagging, adduct and isotope flagging,
    peak alignment, and contaminant ion removal.'
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Docker
  role: Container runtime for local execution (docker profile) ensuring computational
    reproducibility.
  repo: https://docs.docker.com/engine/installation/
- name: Singularity
  role: Container runtime for high-performance computing environments (singularity
    profile) to run MS-DIAL and MS-FLO reproducibly.
  repo: https://www.sylabs.io/guides/3.0/user-guide/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:1f313ec2c26ff44609d92f9cff245049825975be16e141d8792bdbeed0dbd161
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# lc-ms-metabolomics-data-processing

## Summary

Run the containerized Nextflow4MS-DIAL workflow to convert raw LC-HRMS .mzML samples into filtered, aligned, and contaminant-removed peak tables, validated on the MetaboLights MTBLS733 dataset.

## When to use

Use when raw LC-MS/MS metabolomics data in .mzML (or .abf) format must be processed reproducibly via MS-DIAL and MS-FLO, and peak tables with retrospective provenance reports are needed.

## When NOT to use

- The data is already a processed feature/peak table — the skill operates on raw .mzML/.abf chromatograms and will not add value to post-MS-FLO files.
- You need a graphical interface or statistical/pathway analysis: Nextflow4MS-DIAL is command-line-only and stops after peak table generation; downstream tools must be run separately.
- You cannot use containerization (Docker/Singularity) and need guaranteed bit-level reproducibility across machines — the article notes reproducibility depends on the containerized profiles.

## Inputs

- Raw LC-MS data files (.mzML or .abf) placed in data/raw_data/
- MS-DIAL configuration file named msdial_params.txt
- MS-FLO configuration file named msflo_params.ini
- MS1 metabolite library file named ms1_lib.txt
- MS2 metabolite library file named ms2_lib.msp
- Reference file declared in conf/base.config

## Outputs

- MS-DIAL peak table (e.g. 25,447 peaks on the MTBLS733 test data; .msdial, renamed to .tsv)
- MS-FLO processed peak table (e.g. 25,414 peaks, 1,030 true positives at 4.05%)
- Nextflow retrospective provenance artifacts: execution_report.html and execution_timeline.html
- Execution log with Nextflow/workflow version, parameter settings, resource allocation, container information, and process-level logs (logs/execution.log)

## How to apply

Clone Nextflow4Metabolomics/nextflow4ms-dial, place raw .mzML files in data/raw_data/ along with msdial_params.txt, msflo_params.ini, ms1_lib.txt, and ms2_lib.msp in data/, and confirm the reference file in conf/base.config. Run `nextflow run main.nf -profile docker` locally or with the `singularity` profile on HiPerGator (allocating 20 CPUs and 200 GB memory) so that MS-DIAL v4.48 performs comprehensive LC-focused peak detection, filtration of noise and contaminants, m/z and retention time identification, and intensity calculation; the workflow then automatically transfers the MS-DIAL peak table to MS-FLO for duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal. Collect the MS-DIAL peak table (MTBLS733 benchmark: 25,447 peaks), the MS-FLO table (25,414 peaks, 1,030 true positives at 4.05%), rename produced .msdial files to .tsv for spreadsheet inspection, and gather the Nextflow retrospective provenance artifacts execution_report.html and execution_timeline.html plus logs/execution.log containing version, parameter, and container metadata. Validate reproducibility by re-executing the workflow and confirming identical peak counts across runs.

## Related tools

- **Nextflow** (Orchestrates the MS-DIAL → MS-FLO pipeline, enabling containerization and reproducible execution across local, Docker, and HPC/SLURM environments.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Performs comprehensive LC-focused data processing: peak detection, filtration of noise and contaminants, m/z and retention time identification, and peak intensity calculation.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-FLO** (Post-MS-DIAL optimization: duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Docker** (Container runtime for local execution (docker profile) ensuring computational reproducibility.) — https://docs.docker.com/engine/installation/
- **Singularity** (Container runtime for high-performance computing environments (singularity profile) to run MS-DIAL and MS-FLO reproducibly.) — https://www.sylabs.io/guides/3.0/user-guide/

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Re-run the workflow and confirm identical results across executions — the paper reports five identical runs on MTBLS733.
- Compare MS-DIAL peak count against the benchmark 25,447 peaks (total) with 1,031 true positives (4.05%).
- Check MS-FLO output converges to 25,414 peaks with 1,030 true positives (4.05%) after flagging and alignment.
- Verify execution_report.html and execution_timeline.html are produced and that logs/execution.log contains Nextflow/workflow version, parameter settings, and container metadata.
- Confirm .msdial outputs have been renamed to .tsv and open in spreadsheet software without errors.

## Limitations

- Tested only on 10 samples from MTBLS733; scalability to much larger cohorts is not quantitatively demonstrated.
- MS-FLO must be downloaded from the Internet rather than bundled with the workflow.
- MS-DIAL updates are monitored manually, with no automated maintenance plan for workflow and containers.
- Not yet tested on Apple silicon (M-series) systems; only Intel macOS and Linux (HiPerGator, RHEL 8.8) validated.
- The workflow is command-line only — no graphical user interface for users with limited computational experience.
- Only .mzML and .abf inputs are accepted; other vendor formats must be converted beforehand with ProteoWizard msConvert or Reifycs Abf Converter.
- Avoid special characters in file names (underscores are safe) or processes may be skipped with no execution.

## Evidence
