---
name: multimodal-msi-ifm-image-registration
description: Use when consecutive tissue sections have been measured by MALDI MSI
  and stained/IFM-imaged, and the IFM cell-type masks (e.g., urothelium markers) must
  be warped into the MSI pixel frame to assign lipids to cell populations.
when_to_use_negative:
- Tissue sections where landmarks present in both modalities (e.g., laser ablation
  marks) exist — landmark-based registration is reported to be more accurate
- Data requiring lipid annotation or biological interpretation — msiFlow explicitly
  does not include an annotation or interpretation step
- Single-modality data with no paired IFM or MSI image to register against
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_0121
tools:
- name: msiFlow
  role: Provides the msi_if_registration_flow Snakemake workflow combining MSI and
    IFM including IFM pre-processing
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine that automates and parallelizes the registration pipeline
    steps
- name: Docker
  role: Container image (msiflow GUI/CLI) that makes the registration workflows runnable
    by non-programmers across platforms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:afb3e56369fb07d871a9e28043af0e12e299c9f850ae7d445433e1e81f2512f8
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multimodal-msi-ifm-image-registration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated co-registration of immunofluorescence microscopy (IFM) images to MALDI mass spectrometry imaging (MSI) data using msiFlow's msi_if_registration_flow Snakemake workflow, with ANTs SyNRA (rigid + affine + deformable) transforms driven by a mutual-information metric, validated quantitatively via the Jaccard index (~0.8).

## When to use

Use when consecutive tissue sections have been measured by MALDI MSI and stained/IFM-imaged, and the IFM cell-type masks (e.g., urothelium markers) must be warped into the MSI pixel frame to assign lipids to cell populations.

## When NOT to use

- Tissue sections where landmarks present in both modalities (e.g., laser ablation marks) exist — landmark-based registration is reported to be more accurate
- Data requiring lipid annotation or biological interpretation — msiFlow explicitly does not include an annotation or interpretation step
- Single-modality data with no paired IFM or MSI image to register against

## Inputs

- Raw MALDI MSI data (imzML/vendor formats) from one tissue section
- IFM images (e.g., urothelium-marker channels) from a consecutive tissue section
- MSI-derived UMAP tissue/segmentation image used as the fixed image
- msi_if_registration_flow config.yaml specifying moving/fixed images and channels

## Outputs

- Registered IFM images warped into the MSI frame (written to the registered/ directory)
- Transformed urothelial (cell-type) mask aligned to MSI pixels
- Jaccard index quantifying IFM-to-MSI mask overlap

## How to apply

Download the multimodal example data (MALDI MSI plus IFM images) from Zenodo record 10.5281/zenodo.11913042 and prepare the msi_if_registration_flow config.yaml, setting the IF channel as the moving image and the MSI-derived UMAP tissue image as the fixed image. Run the Snakemake workflow (locally or via the msiFlow Docker image), which performs IFM pre-processing and then applies an ANTs SyNRA transform chain — rigid, affine, and deformable — optimized with the mutual-information metric because the modalities have no shared landmarks. The registered IFM images are written to the registered/ output directory. Threshold the registered urothelium-marker channel to obtain the transformed urothelial mask, then compute the Jaccard index (intersection over union) against the reference urothelial mask; an average Jaccard of 0.8 (acceptable range 0.75–0.85) indicates correct registration. Consecutive sections are used deliberately because staining washes strongly reduce the signal-to-noise of the spatial lipidome, so registration — not shared modality — bridges the sections.

## Related tools

- **msiFlow** (Provides the msi_if_registration_flow Snakemake workflow combining MSI and IFM including IFM pre-processing) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine that automates and parallelizes the registration pipeline steps)
- **Docker** (Container image (msiflow GUI/CLI) that makes the registration workflows runnable by non-programmers across platforms)

## Examples

```
snakemake --snakefile msi_if_registration_flow/Snakefile --configfile msi_if_registration_flow/config.yaml
```

## Evaluation signals

- Computed Jaccard index between the transformed urothelial mask and the reference urothelial mask falls in the 0.75–0.85 range, consistent with the reported average of 0.8
- A registered image file exists in the registered/ output directory after the workflow completes
- The Snakemake DAG completes without errors using the config.yaml with IF channel as moving image and MSI-derived UMAP image as fixed image
- Warped IFM urothelium signal visually coincides with the MSI tissue borders (the UMAP-based tissue architecture used as fixed image)
- The ANTs transform chain (rigid + affine + SyN deformable) converges under the mutual-information metric without NaN transforms

## Limitations

- Automated registration may be less accurate than landmark-based registration or laser ablation marks because consecutive sections are used
- Complete cell-specificity is limited by use of consecutive tissue sections and lateral diffusion of lipids during MSI sample preparation
- Validation was demonstrated on the urinary-tract-infection (UPEC-infected bladder) lipidomic use case; scaling to other MSI modalities (metabolomics, proteomics) is anticipated but not demonstrated

## Evidence
