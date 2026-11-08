---
name: mutual-information-multimodal-matching
description: Use when consecutive tissue sections were measured by MALDI MSI and IFM
  and the cell-marker masks from microscopy must be warped into the MSI pixel frame
  without manual landmarks.
when_to_use_negative:
- Landmarks or laser ablation marks shared between the two modalities are available
  — the article notes the automated approach might be less accurate than landmark-based
  registration, so a landmark method should be preferred
- The modalities do not share recognizable tissue architecture and borders, which
  the automated matrix/tissue detection relies on
- The goal is lipid annotation or interpretation — the article states msiFlow does
  not include an annotation or interpretation step
tools:
- name: msiFlow
  role: Provides the msi_if_registration_flow Snakemake workflow that combines MSI
    and IFM including IFM pre-processing and runs the image registration
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Workflow engine executing the registration steps in an automated, reproducible
    manner
- name: Docker
  role: Container image enabling local execution of the registration workflow via
    the CLI without local dependency installation
  repo: https://hub.docker.com/r/phispa1812/msiflow_gui
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:1c04e9c4a526ec30d7292f2ff8cce34324b675a0d6d464b723244357c49db818
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# mutual-information-multimodal-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated co-registration of immunofluorescence microscopy (IFM) images onto high-resolution MALDI MSI data using msiFlow's msi_if_registration_flow, which chains ANTs SyNRA transforms (rigid + affine + deformable) with a mutual-information metric, validated quantitatively by a Jaccard index of ~0.8 between registered and reference urothelial masks.

## When to use

Use when consecutive tissue sections were measured by MALDI MSI and IFM and the cell-marker masks from microscopy must be warped into the MSI pixel frame without manual landmarks.

## When NOT to use

- Landmarks or laser ablation marks shared between the two modalities are available — the article notes the automated approach might be less accurate than landmark-based registration, so a landmark method should be preferred
- The modalities do not share recognizable tissue architecture and borders, which the automated matrix/tissue detection relies on
- The goal is lipid annotation or interpretation — the article states msiFlow does not include an annotation or interpretation step

## Inputs

- Raw MALDI MSI data (e.g. imzML) from one tissue section, with a UMAP-based tissue segmentation image as fixed image
- IFM image(s) of a consecutive tissue section (moving image), including a urothelium-marker channel
- config.yaml for msi_if_registration_flow specifying moving/fixed images and IF channels
- Example multimodal data from Zenodo record 10.5281/zenodo.11913042

## Outputs

- Registered IFM image(s) resampled into the MSI frame, written to the registered/ directory
- Transformed urothelial mask derived by thresholding the registered urothelium-marker channel
- Quantitative Jaccard index of mask overlap (expected ~0.8 on the example data)

## How to apply

Prepare the msi_if_registration_flow config.yaml, setting the IF channel as the moving image and the MSI-derived UMAP tissue image (from msi_segmentation_flow) as the fixed image. Execute the Snakemake workflow, locally or via the msiFlow Docker image, so that IFM pre-processing is followed by registration. The workflow applies an ANTs SyNRA transform chain (rigid, affine, then deformable) optimized with a mutual-information metric, which is appropriate because the modalities share tissue architecture and borders but have different intensity distributions. The registered IFM images are written to the registered/ output directory. Validate by thresholding the registered urothelium-marker channel to obtain the transformed urothelial mask and computing the Jaccard index (intersection over union) against the reference urothelial mask from MSI; accept results in the 0.75–0.85 range, consistent with the reported average of 0.8.

## Related tools

- **msiFlow** (Provides the msi_if_registration_flow Snakemake workflow that combines MSI and IFM including IFM pre-processing and runs the image registration) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Workflow engine executing the registration steps in an automated, reproducible manner)
- **Docker** (Container image enabling local execution of the registration workflow via the CLI without local dependency installation) — https://hub.docker.com/r/phispa1812/msiflow_gui

## Examples

```
snakemake --snakefile msi_if_registration_flow/Snakefile --configfile msi_if_registration_flow/config.yaml
```

## Evaluation signals

- Jaccard index between the transformed IFM urothelial mask and the reference MSI urothelial mask falls in the 0.75–0.85 range (reported average 0.8)
- Registered image file exists in the registered/ output directory after the workflow completes
- The workflow runs end-to-end from the Snakemake CLI (or Docker image) without errors using the config.yaml
- The moving/fixed image roles are correctly assigned: IF channel as moving, MSI-derived UMAP tissue image as fixed
- The transform chain completes all three stages (rigid, affine, deformable) using the mutual-information metric

## Limitations

- Registration uses consecutive tissue sections rather than the same section, so complete cell-specificity is limited due to section-to-section differences and lateral diffusion of lipids during MSI sample preparation
- The automated registration approach might be less accurate than landmark-based registration or laser ablation marks
- m/z annotation is out of scope: msiFlow does not include an annotation or interpretation step, only identifying interesting molecular patterns and preliminary candidates

## Evidence
