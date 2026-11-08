---
name: ly6g-ifm-mask-based-roi-extraction
description: Use when registered MALDI MSI and immunofluorescence microscopy (IFM)
  data are available with a segmented cell-type marker mask (e.g. Ly6G for neutrophils)
  and the goal is to isolate that cell population's pixel spectra for ROI-restricted
  downstream analysis.
when_to_use_negative:
- When no IFM segmentation mask or registered IFM–MSI pair exists — msiFlow's registration
  relies on consecutive sections, so an unregistered marker image cannot define reliable
  ROIs on the MSI pixel grid
- When the analysis goal is whole-tissue molecular segmentation rather than a specific
  cell population — msi_segmentation_flow (clustering of MSI data directly) is the
  appropriate workflow instead
- When you need lipid annotation or biological interpretation of the identified ions
  — the paper states msiFlow 'does not include an annotation or interpretation step'
tools:
- name: msiFlow
  role: End-to-end Snakemake framework providing if_segmentation_flow (thresholding-based
    IFM marker segmentation), msi_if_registration_flow (MSI–IFM registration), and
    molecular_heterogeneity_flow (UMAP-based heterogeneity analysis) used to build
    and exploit the Ly6G mask
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine automating and parallelising the segmentation, registration,
    and mask-based extraction steps
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Container image packaging all msiFlow workflows for platform-independent,
    reproducible execution of the mask-based ROI extraction
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:2619b5fc9270277d2c7e99b1b3e2d2b6b4bf3028ed16dcc0bea69202f049e7b4
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# ly6g-ifm-mask-based-roi-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts MALDI MSI pixel spectra falling inside Ly6G-positive regions defined by a thresholding-segmented IFM mask, so that molecular heterogeneity analysis (UMAP + HDBSCAN) can be restricted to neutrophil-associated pixels in UPEC-infected bladder tissue.

## When to use

Use when registered MALDI MSI and immunofluorescence microscopy (IFM) data are available with a segmented cell-type marker mask (e.g. Ly6G for neutrophils) and the goal is to isolate that cell population's pixel spectra for ROI-restricted downstream analysis.

## When NOT to use

- When no IFM segmentation mask or registered IFM–MSI pair exists — msiFlow's registration relies on consecutive sections, so an unregistered marker image cannot define reliable ROIs on the MSI pixel grid
- When the analysis goal is whole-tissue molecular segmentation rather than a specific cell population — msi_segmentation_flow (clustering of MSI data directly) is the appropriate workflow instead
- When you need lipid annotation or biological interpretation of the identified ions — the paper states msiFlow 'does not include an annotation or interpretation step'

## Inputs

- Registered MALDI MSI data (e.g. registered/msi_if_registration/registered/UPEC_12.tif from the msiFlow demo dataset)
- Thresholding-segmented Ly6G IFM mask (e.g. if_segmentation/segmented/UPEC_12.tif from if_segmentation_flow)
- IFM microscopy image and MSI data registered by msi_if_registration_flow (consecutive tissue sections)

## Outputs

- Subset of MSI pixel spectra restricted to Ly6G-positive (neutrophil) regions
- HDBSCAN cluster assignments for Ly6G+ pixels (three clusters expected)
- cluster_tissue_composition.csv: per-cluster percentage composition across urothelium, lamina propria, and muscle

## How to apply

Load the registered MALDI MSI data and the thresholding-segmented Ly6G IFM mask for the same section from the msiFlow demo chain (registered/msi_if_registration/registered/UPEC_12.tif and if_segmentation/segmented/UPEC_12.tif). Use the binary Ly6G mask to select only MSI pixels overlapping Ly6G-positive regions, leveraging msiFlow's if_segmentation_flow (thresholding-based IFM marker segmentation) and msi_if_registration_flow outputs. This pairing matters because IFM and MSI are acquired on consecutive tissue sections, so registration error bounds how faithfully the mask maps to the MSI pixel grid. Pass the extracted Ly6G+ pixel spectra into the molecular_heterogeneity_flow (UMAP dimensionality reduction followed by HDBSCAN) to resolve molecular sub-populations without pre-specifying cluster number. Finally, assign each resulting cluster to tissue compartments (urothelium, lamina propria, muscle) and tabulate per-cluster tissue composition. All steps run inside the msiFlow Snakemake/Docker framework for reproducibility.

## Related tools

- **msiFlow** (End-to-end Snakemake framework providing if_segmentation_flow (thresholding-based IFM marker segmentation), msi_if_registration_flow (MSI–IFM registration), and molecular_heterogeneity_flow (UMAP-based heterogeneity analysis) used to build and exploit the Ly6G mask) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine automating and parallelising the segmentation, registration, and mask-based extraction steps) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Container image packaging all msiFlow workflows for platform-independent, reproducible execution of the mask-based ROI extraction) — https://github.com/Immunodynamics-Engel-Lab/msiflow

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --use-conda --cores 1 if_segmentation_flow
```

## Evaluation signals

- Mask-restricted UMAP+HDBSCAN analysis of Ly6G+ pixels recovers approximately three molecular clusters rather than one or many
- Cluster 1 localises mainly to the lamina propria (~88% of cluster pixels) and cluster 2 splits between lamina propria (~52.7%) and muscle (~46.4%)
- The per-cluster tissue composition table cluster_tissue_composition.csv is produced with urothelium/lamina propria/muscle compartments and cluster percentages summing to ~100% per cluster
- Extraction uses pixels only inside the segmented Ly6G mask from if_segmentation_flow applied to the registered UPEC_12.tif pair, confirming correct file-chain wiring (registered → segmented)

## Limitations

- Cell-specificity is limited because IFM and MSI are acquired on consecutive tissue sections and lipids laterally diffuse during MSI sample preparation
- The automated registration approach may be less accurate than landmark-based registration or laser ablation marks, propagating mask-to-MSI misalignment into the ROI extraction
- msiFlow does not include an annotation or interpretation step, so lipids underlying the clusters must be annotated separately (e.g. via mass matching to LipidMaps)

## Evidence
