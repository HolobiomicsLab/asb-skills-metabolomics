---
name: msiflow-workflow-configuration-and-execution
description: Use when MALDI MSI (optionally multimodal with IFM) data must be processed
  through an automated, reproducible workflow — e.g. pixel-wise statistical comparison
  of lipid intensities between infected and control groups within tissue ROIs.
when_to_use_negative:
- msiFlow does not include an annotation or interpretation step — do not expect lipid
  ID annotation (beyond preliminary mass matching, e.g. LipidMaps) from these workflows
- Scaling to other MSI modalities such as metabolomics and proteomics is anticipated
  but not demonstrated — avoid assuming out-of-the-box support
- If the analysis requires the exact landmark-based registration accuracy of laser
  ablation marks, the automated registration approach may be less accurate than desired
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_3174
tools:
- name: msiFlow
  role: Provides the Snakemake workflows (msi_preprocessing_flow, msi_if_registration_flow,
    msi_segmentation_flow, if_segmentation_flow, region_group_analysis_flow, molecular_signatures_flow)
    that are configured and executed by this skill
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine used to run each msiFlow flow from the CLI with --snakefile,
    --cores and --configfile, enabling parallel high-throughput processing
- name: Docker
  role: All workflows are packaged in a Docker image, making execution broadly applicable
    and suitable for non-programmers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:babd6302aff607aa92da42c621735c3004e5f2b0b313eb9c012601b74b4ba8d5
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
  - msiFlow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# msiflow-workflow-configuration-and-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run msiFlow's Snakemake workflows (e.g. msi_preprocessing_flow, msi_segmentation_flow, region_group_analysis_flow) to carry out end-to-end, reproducible MALDI MSI analysis from raw data import to statistical comparison of ROIs between sample groups.

## When to use

Use when MALDI MSI (optionally multimodal with IFM) data must be processed through an automated, reproducible workflow — e.g. pixel-wise statistical comparison of lipid intensities between infected and control groups within tissue ROIs.

## When NOT to use

- msiFlow does not include an annotation or interpretation step — do not expect lipid ID annotation (beyond preliminary mass matching, e.g. LipidMaps) from these workflows
- Scaling to other MSI modalities such as metabolomics and proteomics is anticipated but not demonstrated — avoid assuming out-of-the-box support
- If the analysis requires the exact landmark-based registration accuracy of laser ablation marks, the automated registration approach may be less accurate than desired

## Inputs

- Raw MALDI MSI data (vendor-neutral imzML-compatible import; t-MALDI-2 acquisitions in the UTI case study)
- Peak-pixel-intensity table produced by msi_preprocessing_flow
- Segmented/ROI-annotated pixel tables from msi_segmentation_flow
- IF microscopy images (for msi_if_registration_flow)
- YAML config file specifying samples, group labels (e.g. infected vs control) and ROI annotations

## Outputs

- Peak list from raw MALDI MSI data (msi_preprocessing_flow)
- Registered MSI+IFM data (msi_if_registration_flow)
- Segmentation/clustering results (msi_segmentation_flow)
- Per-lipid statistical comparison table (log2 fold-changes and p-values per ROI between groups) from region_group_analysis_flow
- Visualisation outputs such as volcano plots of regulated lipid species per ROI

## How to apply

Clone msiFlow (github.com/Immunodynamics-Engel-Lab/msiflow) and select the appropriate Snakemake workflow from its 7 flows: msi_preprocessing_flow for generating a peak list from raw MALDI MSI data, msi_if_registration_flow for combining MSI with IFM, msi_segmentation_flow for dimensionality reduction and clustering, or region_group_analysis_flow for statistical analysis of molecular changes between two groups across ROIs. Each flow is executed from the command line via `snakemake --snakefile <flow>/Snakefile --cores <n> --configfile <config.yaml>`, where the config file defines inputs, group metadata (e.g. infected n=4 vs control n=4), and ROI annotations. Workflows can be run with multiple cores for parallel, high-throughput processing, and a Docker image is available for non-programmers. Demo data with expected output files is provided in the repository's demo folder and on Zenodo (10.5281/zenodo.11913042) to validate the installation before applying it to real data. After execution, verify that expected output tables and plots (e.g. per-lipid log2 fold-changes, p-values, volcano plots per ROI) exist in the workflow output directory.

## Related tools

- **msiFlow** (Provides the Snakemake workflows (msi_preprocessing_flow, msi_if_registration_flow, msi_segmentation_flow, if_segmentation_flow, region_group_analysis_flow, molecular_signatures_flow) that are configured and executed by this skill) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine used to run each msiFlow flow from the CLI with --snakefile, --cores and --configfile, enabling parallel high-throughput processing)
- **Docker** (All workflows are packaged in a Docker image, making execution broadly applicable and suitable for non-programmers)

## Examples

```
snakemake --snakefile region_group_analysis_flow/Snakefile --cores 8 --configfile config.yaml
```

## Evaluation signals

- The demo dataset (github.com/Immunodynamics-Engel-Lab/msiflow/tree/main/demo) runs successfully and reproduces its expected output files
- Output tables and plots exist in the workflow output directory after the Snakemake run completes without errors
- For region_group_analysis_flow, regulation directions match expectations: TG/DG/PC species up-regulated and PE/ST species down-regulated in infected urothelium vs control (p < 0.05, |log2(FC)| > 1)
- Volcano plots per ROI highlight significantly regulated lipid species and their classes, consistent with the aggregated fold-change/p-value table

## Limitations

- msiFlow does not include an annotation or interpretation step; it is aimed at identifying interesting molecular patterns and preliminary candidates
- Achieving complete cell-specificity is limited by the use of consecutive tissue sections and lateral diffusion of lipids during MSI sample preparation
- The automated registration approach might be less accurate than landmark-based registration or laser ablation marks
- Scaling to other MSI modalities (metabolomics, proteomics) is anticipated but not demonstrated

## Evidence
