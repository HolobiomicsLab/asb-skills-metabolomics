---
name: imzml-data-handling
description: Use when raw multi-vendor MALDI MSI files (e.g. Bruker/Waters vendor
  formats) need to be imported, pre-processed and exported to the open imzML standard,
  or when reproducible peak lists must be generated from raw imaging MS data.
when_to_use_negative:
- Data is already a processed peak list or centroid table — msi_preprocessing_flow
  is designed to start from raw vendor MALDI MSI files.
- The task is downstream analysis (ROI statistics, ML molecular signatures, or IFM/MALDI
  registration) — use msiFlow's other workflows (e.g. region_group_analysis_flow,
  molecular_signatures_flow, msi_if_registration_flow) instead.
- No annotation or interpretation step is needed — msiFlow explicitly does not include
  an annotation or interpretation step.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: msiFlow
  role: Automated Snakemake workflows (msi_preprocessing_flow) that import raw vendor
    MALDI MSI data, pre-process it, and write the open-standard imzML output.
  repo: github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine that automates and parallelises the pre-processing steps for
    high-throughput analyses.
- name: Docker
  role: Container image (phispa1812/msiflow_cli) providing a platform-independent
    way to deploy the workflow for non-programmers.
- name: imzy
  role: Python reader/writer interface to imzML and other imaging mass spectrometry
    formats (imzML, Bruker TSF/TDF/NeoFlex, Waters MassLynx) for inspecting and reading
    the produced data.
  repo: https://github.com/vandeplaslab/imzy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:a93b7040c1b18d8de0febff11f5e7506de63a7455a94f5b7c695c753bfd76489
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Immunodynamics-Engel-Lab/msiflow
  tools:
  - Docker
  - Snakemake
  - imzy
  - msiFlow
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# imzml-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handling raw vendor MALDI MSI data through the open-standard imzML format, using msiFlow's msi_preprocessing_flow to convert multi-vendor raw files into imzML plus a peak list and QC visualisations.

## When to use

Use when raw multi-vendor MALDI MSI files (e.g. Bruker/Waters vendor formats) need to be imported, pre-processed and exported to the open imzML standard, or when reproducible peak lists must be generated from raw imaging MS data.

## When NOT to use

- Data is already a processed peak list or centroid table — msi_preprocessing_flow is designed to start from raw vendor MALDI MSI files.
- The task is downstream analysis (ROI statistics, ML molecular signatures, or IFM/MALDI registration) — use msiFlow's other workflows (e.g. region_group_analysis_flow, molecular_signatures_flow, msi_if_registration_flow) instead.
- No annotation or interpretation step is needed — msiFlow explicitly does not include an annotation or interpretation step.

## Inputs

- raw vendor MALDI MSI data files (multi-vendor, e.g. Bruker/Waters raw formats)
- config.yaml configuration file for msi_preprocessing_flow
- example dataset from Zenodo deposit 10.5281/zenodo.11913042

## Outputs

- processed imaging data in imzML format (.imzML/.ibd)
- peak list generated from raw MALDI MSI data
- quality control visualisations

## How to apply

Download the example dataset from Zenodo (10.5281/zenodo.11913042) and note the memory budget — the paper's pre-processing dataset requires at least 120 GB RAM. Set up msiFlow via Docker (docker pull phispa1812/msiflow_cli) or by cloning the repository and creating the msiflow_env.yaml conda environment. Structure and name the raw MALDI MSI input files and set configuration parameters in config.yaml following the README in the msi_preprocessing_flow directory. Execute the workflow with Snakemake (--snakefile msi_preprocessing_flow/Snakefile --configfile <config>), optionally adding --resources mem_mb=<available_RAM_in_MB> to avoid out-of-memory errors. The workflow processes raw vendor files in parallel and performs peak picking (with a spatial-coherence-based peak filtering step), alignment, matrix pixel removal, normalisation, spectral-similarity outlier detection and de-isotoping, writing processed data as imzML.

## Related tools

- **msiFlow** (Automated Snakemake workflows (msi_preprocessing_flow) that import raw vendor MALDI MSI data, pre-process it, and write the open-standard imzML output.) — github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine that automates and parallelises the pre-processing steps for high-throughput analyses.)
- **Docker** (Container image (phispa1812/msiflow_cli) providing a platform-independent way to deploy the workflow for non-programmers.)
- **imzy** (Python reader/writer interface to imzML and other imaging mass spectrometry formats (imzML, Bruker TSF/TDF/NeoFlex, Waters MassLynx) for inspecting and reading the produced data.) — https://github.com/vandeplaslab/imzy

## Examples

```
snakemake --snakefile msi_preprocessing_flow/Snakefile --cores <number-of-cores> --configfile <path-to-config> --resources mem_mb=<available_RAM_in_MB>
```

## Evaluation signals

- The Snakemake workflow completes without out-of-memory errors (optionally mitigated with --resources mem_mb).
- A peak list output file is generated from the raw MALDI MSI data — the paper's qualitative acceptance criterion for this step.
- Processed data are present in the open-standard imzML format and located at the paths described in the workflow README.
- Quality control visualisations are generated alongside the imzML output.
- Intermediate pre-processing steps (peak picking with spatial coherence filtering, alignment, matrix pixel removal, normalisation, outlier detection, de-isotoping) each produce their expected outputs.

## Limitations

- The paper's example pre-processing dataset requires at least 120 GB RAM; limited-memory machines may fail without the mem_mb resource limiter.
- No quantitative acceptance threshold is reported for validating the pre-processing output — validation is the qualitative check that the workflow completes and generates a peak list.
- msiFlow does not include an annotation or interpretation step.
- Bruker TDF ion mobility data can be read in imzy, but imzML export writes spectra collapsed to the m/z axis.

## Evidence

- [readme] imzy reads imzML and vendor formats: [evidence span withheld — anchor status: external_doc]
