---
name: ants-synra-deformable-registration
description: Use when MALDI MSI and IFM images are from consecutive tissue sections
  and IFM-derived cell masks must be warped into the MSI pixel frame for cell-population-level
  lipid assignment.
when_to_use_negative:
- Landmarks or features present in both modalities, or laser ablation marks, are available
  — the article notes this automated registration might be less accurate than landmark-based
  registration.
- Input is single-modality MALDI MSI with no paired IFM image; msi_if_registration_flow
  explicitly combines MSI & IFM.
- The goal is lipid annotation or interpretation — msiFlow does not include an annotation
  or interpretation step, so registration alone cannot deliver identified molecules.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: msiFlow
  role: Hosts the msi_if_registration_flow Snakemake workflow that chains IFM pre-processing
    and the ANTs-based registration, writing results to the registered/ directory
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: ANTs
  role: Supplies the SyNRA transform chain (rigid + affine + deformable) with the
    mutual-information metric used to warp IFM images into the MSI frame
- name: Snakemake
  role: Workflow engine that executes the registration steps of msi_if_registration_flow
    reproducibly and in parallel
- name: Docker
  role: msiFlow is incorporated into a Docker image so the registration workflow can
    be run via the Docker CLI image without local installation
  repo: https://hub.docker.com/r/phispa1812/msiflow_gui
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:63643039fc01c47c542d4ee322f23f33a60e670385df11604e4a8e498b89051c
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
  - ANTs
  - Docker
  - Snakemake
  - msiFlow
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ants-synra-deformable-registration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Warps immunofluorescence microscopy (IFM) images into the frame of MALDI MSI data using an ANTs SyNRA transform chain (rigid + affine + deformable) with a mutual-information metric, as automated in msiFlow's msi_if_registration_flow and quantitatively validated by a Jaccard index of ~0.8.

## When to use

Use when MALDI MSI and IFM images are from consecutive tissue sections and IFM-derived cell masks must be warped into the MSI pixel frame for cell-population-level lipid assignment.

## When NOT to use

- Landmarks or features present in both modalities, or laser ablation marks, are available — the article notes this automated registration might be less accurate than landmark-based registration.
- Input is single-modality MALDI MSI with no paired IFM image; msi_if_registration_flow explicitly combines MSI & IFM.
- The goal is lipid annotation or interpretation — msiFlow does not include an annotation or interpretation step, so registration alone cannot deliver identified molecules.

## Inputs

- MALDI MSI dataset (e.g., t-MALDI-2 lipidomic imzML data of the tissue section)
- IFM images of the consecutive tissue section, including the urothelium-marker channel
- MSI-derived UMAP tissue segmentation image used as the fixed image
- config.yaml specifying moving (IF channel) and fixed (MSI UMAP tissue image) inputs for msi_if_registration_flow

## Outputs

- Registered IFM image(s) warped into the MSI frame, written to the registered/ output directory
- Transformed urothelial mask obtained by thresholding the registered marker channel
- Jaccard index (intersection over union) quantifying registration overlap (~0.8 expected)

## How to apply

Prepare the msi_if_registration_flow config.yaml with the MSI-derived UMAP tissue image (from msi_segmentation_flow) as fixed image and the IF channel as moving image, then run the Snakemake workflow locally or via the Docker CLI image to execute IFM pre-processing followed by registration. Registration applies an ANTs SyNRA transform chain — rigid, then affine, then deformable — optimized with a mutual-information metric, which suits the lack of one-to-one intensity correspondence between lipid ion images and fluorescence stains. Warped IFM image(s) are written to the registered/ output directory. Because the modalities are acquired on consecutive tissue sections — staining washes strongly reduce the signal-to-noise of the spatial lipidome — intensity-based registration to the MSI frame is required rather than direct overlay. Validate by thresholding the registered urothelium-marker channel into a binary mask and computing the Jaccard index (intersection over union) against a reference urothelial mask; expect ~0.8 (accept 0.75-0.85).

## Related tools

- **msiFlow** (Hosts the msi_if_registration_flow Snakemake workflow that chains IFM pre-processing and the ANTs-based registration, writing results to the registered/ directory) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **ANTs** (Supplies the SyNRA transform chain (rigid + affine + deformable) with the mutual-information metric used to warp IFM images into the MSI frame)
- **Snakemake** (Workflow engine that executes the registration steps of msi_if_registration_flow reproducibly and in parallel)
- **Docker** (msiFlow is incorporated into a Docker image so the registration workflow can be run via the Docker CLI image without local installation) — https://hub.docker.com/r/phispa1812/msiflow_gui

## Examples

```
snakemake --snakefile msi_if_registration_flow/Snakefile --configfile config.yaml
```

## Evaluation signals

- Computed Jaccard index (intersection over union) between the transformed urothelial mask and the reference urothelial mask falls within 0.75-0.85 (reported average 0.8).
- Registered IFM image file(s) exist in the registered/ output directory after the workflow completes.
- snakemake --snakefile msi_if_registration_flow/Snakefile --configfile ... runs to completion without error, locally or via the Docker CLI image.
- The transform chain includes all three SyNRA stages — rigid, affine, and deformable — optimized with the mutual-information metric (verify ANTs transform outputs/logs).
- Config maps the IF channel to moving image and the MSI-derived UMAP tissue image to fixed image; swapped roles indicate misconfiguration.

## Limitations

- The article acknowledges the automated registration approach might be less accurate than the popular landmark-based registration selecting features present in both modalities, or than registration using laser ablation marks from MSI.
- Consecutive tissue sections inherently bound accuracy: staining washes strongly reduce the signal-to-noise of the spatial lipidome, and lateral lipid diffusion during sample preparation limits complete cell-specificity of assigned signals.
- Demonstrated for lipidomic MALDI MSI + IFM (UPEC urinary tract infection model); scaling to other MSI modalities such as metabolomics and proteomics is anticipated but not demonstrated.

## Evidence

- [methods] msi_if_registration_flow combines MSI & IFM including IFM pre-processing: "combines MSI & IFM including IFM pre-processing | `msi_if_registration_flow`"
- [intro] Registration enables assigning the spatial lipidome to specific cell populations: "combining IFM and MALDI MSI would enable the assignment of the spatial lipidome to specific cell populations"
- [discussion] Consecutive sections motivate registration because staining washes reduce lipidome signal-to-noise: "We deliberately used consecutive sections as the washing steps of the staining procedure of cell-specific molecules strongly reduce the signal-to-noise of the spatial lipidome."
- [discussion] Automated registration may be less accurate than landmark-based registration: "our automated registration approach might be less accurate than the popular landmark-based registration selecting features present in both modalities"
- [other] Registration applies the ANTs SyNRA chain (rigid + affine + deformable) with mutual-information metric: [evidence span withheld — anchor status: not_found]
- [other] Validation thresholds the registered urothelium-marker channel and computes Jaccard against a reference mask: [evidence span withheld — anchor status: not_found]
- [other] Expected Jaccard range is 0.75-0.85 with reported average 0.8: [evidence span withheld — anchor status: not_found]
