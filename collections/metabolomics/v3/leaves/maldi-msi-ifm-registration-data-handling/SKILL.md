---
name: maldi-msi-ifm-registration-data-handling
description: Use when raw MALDI MSI and IFM images of consecutive tissue sections
  must be spatially aligned to assign lipid spectra to marker-defined cell populations,
  e.g. before extracting Ly6G+ pixel spectra for downstream UMAP/HDBSCAN heterogeneity
  clustering.
when_to_use_negative:
- The MSI and IFM already share the same coordinate system or you have reliable landmark-based
  registration features or laser ablation marks — msiFlow's automated architecture-based
  registration may be less accurate than landmark-based approaches in that setting
- You need lipid annotation/interpretation — msiFlow explicitly does not include an
  annotation or interpretation step, only pattern identification and preliminary candidates
- Input is already a registered, segmented feature table ready for clustering — use
  the segmentation or molecular_heterogeneity_flow workflows instead of re-registering
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3391
tools:
- name: msiFlow
  role: End-to-end open-source workflow suite whose `msi_if_registration_flow` performs
    automated registration of IFM to high-resolution MALDI MSI and IFM pre-processing
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine automating the registration and pre-processing steps and enabling
    parallel, high-throughput data processing
- name: Docker
  role: Container image packaging all msiFlow workflows so registration runs reproducibly
    across platforms and for non-programmers
- name: imzy
  role: Python reader/writer interface for imzML and other imaging mass spectrometry
    formats used for data handling around msiFlow
  repo: https://github.com/vandeplaslab/imzy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:1eaceb76a8a3110dfd880eab0341a59857c8af91b19acdc426913539f5eda45f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# maldi-msi-ifm-registration-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated registration of immunofluorescence microscopy (IFM) images to high-resolution MALDI MSI data and handling of the registered multimodal outputs via msiFlow's `msi_if_registration_flow`, enabling pixel-level mapping of spatial lipidomic signatures to specific cell populations (e.g. Ly6G+ neutrophils) in infected bladder tissue.

## When to use

Use when raw MALDI MSI and IFM images of consecutive tissue sections must be spatially aligned to assign lipid spectra to marker-defined cell populations, e.g. before extracting Ly6G+ pixel spectra for downstream UMAP/HDBSCAN heterogeneity clustering.

## When NOT to use

- The MSI and IFM already share the same coordinate system or you have reliable landmark-based registration features or laser ablation marks — msiFlow's automated architecture-based registration may be less accurate than landmark-based approaches in that setting
- You need lipid annotation/interpretation — msiFlow explicitly does not include an annotation or interpretation step, only pattern identification and preliminary candidates
- Input is already a registered, segmented feature table ready for clustering — use the segmentation or molecular_heterogeneity_flow workflows instead of re-registering

## Inputs

- Raw MALDI MSI data (e.g. t-MALDI-2 imzML or vendor format from UPEC-infected bladder sections)
- Immunofluorescence microscopy (IFM) image of a consecutive tissue section, e.g. UPEC_12.tif stained for Ly6G
- Demo data chain files from github.com/Immunodynamics-Engel-Lab/msiflow/tree/main/demo or Zenodo 10.5281/zenodo.11913042

## Outputs

- Registered MSI-IFM data pair, e.g. registered/msi_if_registration/registered/UPEC_12.tif
- IFM segmentation mask from `if_segmentation_flow`, e.g. if_segmentation/segmented/UPEC_12.tif
- Cell-population-restricted pixel spectra (e.g. Ly6G+ pixels) for downstream UMAP/HDBSCAN heterogeneity clustering

## How to apply

Run msiFlow's `msi_if_registration_flow` (a Snakemake workflow packaged in a Docker image), which combines MSI and IFM data including IFM pre-processing. Because cell-specific stains on the MSI section drastically reduce lipid signal-to-noise, msiFlow deliberately registers IFM from consecutive tissue sections to the MALDI MSI measurement using an automated approach that relies on tissue architecture and borders. Inputs are raw MALDI MSI data (e.g. t-MALDI-2 measurements) and the IFM image (e.g. UPEC_12.tif); the workflow pre-processes IFM, registers it to the MSI pixel grid, and the registered output (e.g. registered/UPEC_12.tif) can then be paired with a thresholding-segmented mask from `if_segmentation_flow` (segmented/UPEC_12.tif) to extract Ly6G+ pixel spectra for downstream `molecular_heterogeneity_flow` (UMAP + HDBSCAN) analyses. Judge success by checking that the registered IFM overlays the MSI tissue section with aligned tissue boundaries and that downstream cluster-to-tissue assignment recovers the expected compartment localisations.

## Related tools

- **msiFlow** (End-to-end open-source workflow suite whose `msi_if_registration_flow` performs automated registration of IFM to high-resolution MALDI MSI and IFM pre-processing) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine automating the registration and pre-processing steps and enabling parallel, high-throughput data processing)
- **Docker** (Container image packaging all msiFlow workflows so registration runs reproducibly across platforms and for non-programmers)
- **imzy** (Python reader/writer interface for imzML and other imaging mass spectrometry formats used for data handling around msiFlow) — https://github.com/vandeplaslab/imzy

## Examples

```
git clone https://github.com/Immunodynamics-Engel-Lab/msiflow.git && snakemake --use-conda --cores 1 -s msi_if_registration_flow
```

## Evaluation signals

- Registered IFM tissue boundaries co-localise with the MSI tissue architecture (tissue borders align after automated registration)
- The paired segmentation mask (segmented/UPEC_12.tif) cleanly restricts pixels to Ly6G-positive regions before spectral extraction
- Downstream HDBSCAN clustering of registered, Ly6G+-restricted pixel spectra recovers ~3 clusters with reported tissue localisations (~83% urothelium, ~88% lamina propria, ~53/46% lamina propria/muscle split)
- The expected output files from the demo data chain (e.g. registered/UPEC_12.tif and segmented/UPEC_12.tif) are produced by the workflow without manual intervention

## Limitations

- Automated registration relies on tissue architecture and borders and may be less accurate than landmark-based registration selecting features present in both modalities or laser ablation marks
- Cell-specificity is incomplete because IFM comes from consecutive tissue sections and lateral diffusion of lipids occurs during MSI sample preparation
- Washing steps of cell-specific stains strongly reduce lipid signal-to-noise, which is why consecutive sections are used rather than staining the measured section

## Evidence

- [discussion] Automated registration of IFM to high-resolution MALDI MSI using consecutive tissue sections: [evidence span withheld — anchor status: not_found]
