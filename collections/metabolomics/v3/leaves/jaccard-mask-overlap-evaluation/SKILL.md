---
name: jaccard-mask-overlap-evaluation
description: Use when an IFM image has been warped into the MALDI MSI frame (e.g.
  via ANTs SyNRA in msiFlow's msi_if_registration_flow) and you need a quantitative
  overlap metric to validate registration accuracy.
when_to_use_negative:
- Registration relies on landmark-based features or laser ablation marks present in
  both modalities — this mask-overlap metric is specific to msiFlow's automated consecutive-section
  registration and may underestimate accuracy of feature-based methods
- The MSI dataset lacks a segmentable, morphologically distinct structure (e.g. urothelium)
  that can be segmented in both modalities, so no comparable masks exist
- Goal is lipid annotation or molecular interpretation rather than registration validation
  — msiFlow explicitly does not include an annotation or interpretation step
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0610
tools:
- name: msiFlow
  role: Provides msi_if_registration_flow, which pre-processes IFM, warps it into
    the MSI frame via ANTs SyNRA, and writes registered images used to derive the
    evaluated mask
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Snakemake
  role: Executes the registration workflow end-to-end so the registered output and
    masks are reproducibly generated
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
- name: Docker
  role: Containerized execution environment for running the msi_if_registration_flow
    workflow without local installation
  repo: https://github.com/Immunodynamics-Engel-Lab/msiflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-55306-7
    title: 'msiFlow: automated workflows for reproducible and scalable multimodal
      mass spectrometry imaging and microscopy data analysis'
  source_content_hash: sha256:03b790807eb6815a683736428ba209786113a3091c029d3513cb7762738071a4
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Jaccard mask overlap evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantitatively validates automated IFM-to-MALDI MSI image co-registration by computing the Jaccard index (intersection over union) between a transformed urothelial mask from registered IFM and a reference MSI-derived urothelial mask; msiFlow's registration of consecutive tissue sections was reported to achieve an average Jaccard of 0.8.

## When to use

Use when an IFM image has been warped into the MALDI MSI frame (e.g. via ANTs SyNRA in msiFlow's msi_if_registration_flow) and you need a quantitative overlap metric to validate registration accuracy.

## When NOT to use

- Registration relies on landmark-based features or laser ablation marks present in both modalities — this mask-overlap metric is specific to msiFlow's automated consecutive-section registration and may underestimate accuracy of feature-based methods
- The MSI dataset lacks a segmentable, morphologically distinct structure (e.g. urothelium) that can be segmented in both modalities, so no comparable masks exist
- Goal is lipid annotation or molecular interpretation rather than registration validation — msiFlow explicitly does not include an annotation or interpretation step

## Inputs

- Registered (transformed) IFM image with a urothelium-marker channel, from the registered/ output directory of msi_if_registration_flow
- Reference urothelial mask derived from the MSI segmentation (MSI-derived UMAP tissue image)
- Multimodal example data (MALDI MSI plus IFM images) from Zenodo record 10.5281/zenodo.11913042

## Outputs

- Jaccard index (intersection over union) quantifying overlap between the transformed IFM urothelial mask and the reference MSI urothelial mask
- Binary transformed urothelial mask obtained by thresholding the registered urothelium-marker channel

## How to apply

After running msi_if_registration_flow, which applies an ANTs SyNRA transform chain (rigid + affine + deformable) with a mutual-information metric to warp IFM images (moving image) onto the MSI-derived UMAP tissue image (fixed image), read the registered image from the registered/ output directory. Threshold the registered urothelium-marker channel to obtain the transformed urothelial mask, then compute the Jaccard index (intersection over union) between this mask and the reference urothelial mask. Accept the registration as validated when the Jaccard index falls in the 0.75-0.85 range, consistent with the reported average of 0.8; also verify the registered image file exists. Because IFM and MSI are acquired on consecutive tissue sections (staining washes degrade the lipidome signal), mask-overlap validation is the appropriate check rather than pixel-level identity.

## Related tools

- **msiFlow** (Provides msi_if_registration_flow, which pre-processes IFM, warps it into the MSI frame via ANTs SyNRA, and writes registered images used to derive the evaluated mask) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Snakemake** (Executes the registration workflow end-to-end so the registered output and masks are reproducibly generated) — https://github.com/Immunodynamics-Engel-Lab/msiflow
- **Docker** (Containerized execution environment for running the msi_if_registration_flow workflow without local installation) — https://github.com/Immunodynamics-Engel-Lab/msiflow

## Examples

```
snakemake --snakefile msi_if_registration_flow/Snakefile --configfile config.yaml
```

## Evaluation signals

- Computed Jaccard index falls within the expected 0.75-0.85 range, consistent with the reported average of 0.8
- Registered image file exists in the registered/ output directory after the Snakemake run completes
- Transformed urothelial mask is obtained by a reproducible threshold on the registered urothelium-marker channel (not manual drawing)
- Jaccard is computed strictly as intersection over union between the transformed IFM mask and the MSI-derived reference mask
- The Snakemake workflow (msi_if_registration_flow) runs to completion without errors, locally or via the Docker CLI image

## Limitations

- masks come from consecutive tissue sections, and lateral diffusion of lipids during MSI sample preparation limits complete cell-specificity, so perfect overlap is not expected
- The automated registration approach validated by this metric might be less accurate than landmark-based registration or laser ablation marks
- Reported performance (average Jaccard 0.8) was demonstrated on the urothelial mask of UPEC-infected bladder t-MALDI-2 data; extension to other tissues or MSI modalities (metabolomics, proteomics) is anticipated but not demonstrated

## Evidence

- [results] The msiFlow registration of IFM to MALDI MSI data was validated by the Jaccard index for the overlap between the transformed urothelial mask from IFM and MSI, yielding an average Jaccard of 0.8.: [evidence span withheld — anchor status: not_found]
