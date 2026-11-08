---
name: snakemake-workflow-execution
description: Use when raw multi-vendor MALDI MSI (and optionally IFM) data must be
  processed through msiFlow's automated Snakemake pipelines, or when reproducing the
  paper's workflows on the Zenodo 10.5281/zenodo.11913042 example datasets.
when_to_use_negative:
- The task requires lipid annotation or interpretation — msiFlow explicitly does not
  include an annotation or interpretation step
- Only a small trial run is needed — use the small demo dataset at github.com/Immunodynamics-Engel-Lab/msiflow/tree/main/demo
  instead of the 120 GB RAM Zenodo example data
- The analysis target is generic image augmentation rather than imaging mass spectrometry
  — imgaug is a different library surfaced only incidentally
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: msiFlow
  role: Repository of seven Snakemake workflows covering pre-processing, registration,
    segmentation, and analysis of MALDI MSI and IFM data
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine executing the msiFlow Snakefiles with parallel, high-throughput
    data processing
- name: Docker
  role: Container runtime for the phispa1812/msiflow_cli image, making the workflows
    applicable without local environment setup
- name: imzy
  role: Reader/writer interface for imzML and other imaging mass spectrometry formats
    used in the msiFlow environment
  repo: https://github.com/vandeplaslab/imzy
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:e7a990c858a0f88eee12b0adaffbfa68be423bbd1c9ae3e4e4a6a3b62941d52e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  repo_url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  tools:
  - Docker
  - Snakemake
  - imzy
  - msiFlow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# snakemake-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executing the msiFlow Snakemake workflows (e.g. msi_preprocessing_flow, msi_if_registration_flow) from a cloned repository or the Docker CLI image to automate end-to-end MALDI MSI and IFM analysis on example or raw vendor data.

## When to use

Use when raw multi-vendor MALDI MSI (and optionally IFM) data must be processed through msiFlow's automated Snakemake pipelines, or when reproducing the paper's workflows on the Zenodo 10.5281/zenodo.11913042 example datasets.

## When NOT to use

- The task requires lipid annotation or interpretation — msiFlow explicitly does not include an annotation or interpretation step
- Only a small trial run is needed — use the small demo dataset at github.com/Immunodynamics-Engel-Lab/msiflow/tree/main/demo instead of the 120 GB RAM Zenodo example data
- The analysis target is generic image augmentation rather than imaging mass spectrometry — imgaug is a different library surfaced only incidentally

## Inputs

- Raw multi-vendor MALDI MSI data files (structured and named per the workflow README)
- IFM microscopy images (for msi_if_registration_flow)
- config.yaml with workflow-specific parameters (e.g. IF channel as moving image, MSI-derived UMAP tissue image as fixed image)
- msiflow_env.yaml conda environment specification or Docker image phispa1812/msiflow_cli

## Outputs

- Peak list generated from raw MALDI MSI data, with processed data in imzML format and quality-control visualisations
- Registered IFM image(s) written to the registered/ output directory
- Generated output files at the locations described in the workflow README

## How to apply

Clone github.com/Immunodynamics-Engel-Lab/msiflow and create the conda environment from msiflow_env.yaml, or pull the Docker image phispa1812/msiflow_cli for a containerised setup. Download the example data from Zenodo deposit 10.5281/zenodo.11913042 (the pre-processing dataset requires at least 120 GB RAM) and consult the workflow-specific README (e.g. msi_preprocessing_flow) to structure and name raw MALDI MSI input files and set parameters in config.yaml. Run the workflow with 'snakemake --snakefile msi_preprocessing_flow/Snakefile --cores <number-of-cores> --configfile <path-to-config>', optionally adding --resources mem_mb=<available_RAM_in_MB> to prevent out-of-memory errors; the registration flow is invoked analogously with msi_if_registration_flow/Snakefile, config.yaml, and Docker as an alternative executor. Snakemake enables parallel processing of vendor files for high-throughput runs; outputs such as the peak list (imzML plus QC visualisations) or registered images (registered/ directory) must be verified against the locations described in the workflow README.

## Related tools

- **msiFlow** (Repository of seven Snakemake workflows covering pre-processing, registration, segmentation, and analysis of MALDI MSI and IFM data) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine executing the msiFlow Snakefiles with parallel, high-throughput data processing)
- **Docker** (Container runtime for the phispa1812/msiflow_cli image, making the workflows applicable without local environment setup)
- **imzy** (Reader/writer interface for imzML and other imaging mass spectrometry formats used in the msiFlow environment) — https://github.com/vandeplaslab/imzy

## Examples

```
snakemake --snakefile msi_preprocessing_flow/Snakefile --cores 8 --configfile config.yaml --resources mem_mb=120000
```

## Evaluation signals

- The Snakemake workflow completes without error and the peak list output file from raw MALDI MSI data is generated, as stated for msi_preprocessing_flow
- Generated output files exist at the locations described in the workflow README
- For msi_if_registration_flow, the registered image file exists in the registered/ directory
- Jaccard index of the transformed urothelial mask vs the reference mask falls in 0.75–0.85, consistent with the reported average of 0.8
- The --resources mem_mb flag is set to available RAM (example pre-processing dataset needs ≥120 GB) and no out-of-memory errors occur

## Limitations

- The paper's Zenodo pre-processing example dataset requires at least 120 GB RAM, so the workflow can fail with out-of-memory errors without a --resources mem_mb limit
- Automated IFM-to-MSI registration may be less accurate than landmark-based registration or laser ablation marks
- msiFlow does not include an annotation or interpretation step; downstream lipid annotation requires external databases such as LipidMaps

## Evidence
