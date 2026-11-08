---
name: metabolomics-workflow-containerization
description: Use when an LC-MS metabolomics pipeline must chain MS-DIAL peak-table
  generation to MS-FLO quality filtering reproducibly, or when reproducibility in
  LC-MS data processing is compromised by uncontainerized downloadable tools like
  MS-DIAL, MZmine, or xcms.
when_to_use_negative:
- Input is already a filtered feature/peak table — this skill covers raw .mzML/.abf
  to MS-DIAL→MS-FLO processing, not downstream statistical or pathway analysis.
- The computing environment requires a graphical user interface; Nextflow4MS-DIAL
  is currently command-line-based only.
- Data are in formats other than .mzML or .abf and cannot be converted with ProteoWizard
  msConvert or Reifycs Abf Converter.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: Nextflow
  role: Workflow engine that orchestrates MS-DIAL and MS-FLO processes, automating
    the transfer of peak tables between them and supporting Docker/Singularity containerization
    for reproducibility on any Unix-like system and multiple job schedulers
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: 'Containerized (version 4.48) first stage: comprehensive LC-focused peak detection,
    m/z and retention time identification, peak intensity calculation, and optional
    metabolite identification on .mzML data'
- name: MS-FLO
  role: 'Containerized second stage: receives the MS-DIAL peak table automatically
    and performs duplicate peak flagging, adduct and isotope flagging, peak alignment,
    and contaminant ion removal'
- name: Docker
  role: Container runtime used with the 'docker' Nextflow profile to encapsulate MS-DIAL
    and MS-FLO for reproducible local execution
- name: Singularity
  role: Container runtime used with the 'singularity' profile for high-performance
    computing environments
- name: ProteoWizard msConvert
  role: Recommended external converter for other raw data formats to .mzML before
    ingestion into the workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:16eb3b42c87f457676e467154fcc15b17ee1ab76c868b1e6620d6b583e04b62e
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
  - Singularity
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# metabolomics-workflow-containerization

## Summary

Wraps MS-DIAL and MS-FLO LC-MS metabolomics processing in a Nextflow workflow with Docker and Singularity containerization so that .mzML peak processing runs reproducibly on local machines or HPC schedulers without manual data handling between tools.

## When to use

Use when an LC-MS metabolomics pipeline must chain MS-DIAL peak-table generation to MS-FLO quality filtering reproducibly, or when reproducibility in LC-MS data processing is compromised by uncontainerized downloadable tools like MS-DIAL, MZmine, or xcms.

## When NOT to use

- Input is already a filtered feature/peak table — this skill covers raw .mzML/.abf to MS-DIAL→MS-FLO processing, not downstream statistical or pathway analysis.
- The computing environment requires a graphical user interface; Nextflow4MS-DIAL is currently command-line-based only.
- Data are in formats other than .mzML or .abf and cannot be converted with ProteoWizard msConvert or Reifycs Abf Converter.

## Inputs

- .mzML LC-MS raw data files placed in data/raw_data/
- .abf files (alternative accepted raw format)
- MS-DIAL parameter file named msdial_params.txt in data/
- MS-FLO parameter file named msflo_params.ini in data/
- MS1 metabolite library named ms1_lib.txt in data/
- MS2 metabolite library named ms2_lib.msp in data/

## Outputs

- MS-DIAL peak table (msdial file renamed to .tsv so it opens in spreadsheet software)
- MS-FLO filtered/flagged peak table (e.g. 25,414 peaks from 25,447 MS-DIAL peaks on the MTBLS733 test set)
- execution_report.html summarizing runtime and computational resource usage
- execution_timeline.html showing the execution timeline per process
- logs/execution.log recording Nextflow and workflow versions, parameters, resource allocation, container information, and process-level logs

## How to apply

Define the pipeline in Nextflow's main.nf with each step (MS-DIAL .mzML processing, then automatic transfer of the peak table to MS-FLO) as a Nextflow process so data handoffs are automated and no manual copying intervenes. Containerize both MS-DIAL (version 4.48) and MS-FLO via Docker images declared in conf/base.config for local runs, or via the singularity profile (conf/HiPerGator.config) for HPC environments like the University of Florida's HiPerGator running Red Hat Enterprise Linux 8.8. Supply the MS-DIAL parameter file (msdial_params.txt) and MS-FLO parameter file (msflo_params.ini) in data/, plus ms1_lib.txt and ms2_lib.msp metabolite libraries, then execute 'nextflow run main.nf -profile docker > logs/execution.log'. The workflow accepts .mzML (from ProteoWizard msConvert) or .abf (from Reifycs Abf Converter) raw files placed in data/raw_data/, and emits annotated CSV/TSV peak tables plus execution_report.html and execution_timeline.html for provenance.

## Related tools

- **Nextflow** (Workflow engine that orchestrates MS-DIAL and MS-FLO processes, automating the transfer of peak tables between them and supporting Docker/Singularity containerization for reproducibility on any Unix-like system and multiple job schedulers) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Containerized (version 4.48) first stage: comprehensive LC-focused peak detection, m/z and retention time identification, peak intensity calculation, and optional metabolite identification on .mzML data)
- **MS-FLO** (Containerized second stage: receives the MS-DIAL peak table automatically and performs duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal)
- **Docker** (Container runtime used with the 'docker' Nextflow profile to encapsulate MS-DIAL and MS-FLO for reproducible local execution)
- **Singularity** (Container runtime used with the 'singularity' profile for high-performance computing environments)
- **ProteoWizard msConvert** (Recommended external converter for other raw data formats to .mzML before ingestion into the workflow)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Peak count check: MS-DIAL should identify 25,447 peaks and MS-FLO output 25,414 peaks on the MTBLS733 10-sample test dataset
- Handoff integrity: confirm no manual copying occurred — Nextflow automatically transfers the MS-DIAL peak table to MS-FLO with no intervention
- Reproducibility: execute the workflow five times; each run should produce identical results
- Provenance artifacts present: execution_report.html, execution_timeline.html, and logs/execution.log record successful completion of both MS-DIAL and MS-FLO processes
- Container usage: logs/execution.log includes container information, confirming MS-DIAL and MS-FLO ran inside the Docker/Singularity image

## Limitations

- Currently, MS-FLO must be downloaded from the Internet rather than bundled with the workflow.
- Software updates (e.g. MS-DIAL) are monitored manually with no efficient maintenance plan for workflow and container.
- The workflow is command-line-only, with no graphical user interface for users with limited computational experience.
- Tested only on Intel-based macOS 13.5.1 and Linux (Red Hat Enterprise Linux 8.8 / HiPerGator); Apple silicon (M-series) systems have not been tested.
- Only 10 samples in the test dataset, and few LC-MS datasets had been processed with both MS-DIAL and MS-FLO under transparent parameter settings, limiting reproduction of steps.
- Reproducibility and scalability were not quantitatively measured.

## Evidence
