---
name: affine-transformation-of-bimodal-images
description: Use when paired IMC and TOF-SIMS images of the same tissue FOV must be
  spatially aligned before single-cell segmentation or protein-metabolite correlation,
  and simple rotation-only or random-shift alignment is insufficient.
when_to_use_negative:
- Input images are already co-registered single-cell feature/intensity tables rather
  than raw image pairs
- Only one modality is available (no paired IMC reference to warp the SIMS image onto)
- The task is downstream analysis (VAE joint embedding, segmentation, trajectory analysis)
  on already aligned images
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3520
tools:
- name: scSpaMet
  role: 'Provides the registration pipeline: template matching, rotation-offset calculation,
    and custom affine transformation scripts (src folder)'
  repo: https://github.com/coskunlab/scSpaMet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:ae5b2c6ddd4a28ab3129ba3965cbdec28d7588b4cf77bebd5d9f0ce9e9211055
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coskunlab/scSpaMet
  tools:
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# affine-transformation-of-bimodal-images

## Summary

Cross-modality registration of TOF-SIMS metabolite images onto IMC protein images using template matching, rotation-offset calculation, and a final affine transformation, as implemented in the scSpaMet pipeline to enable joint single-cell protein-metabolite profiling.

## When to use

Use when paired IMC and TOF-SIMS images of the same tissue FOV must be spatially aligned before single-cell segmentation or protein-metabolite correlation, and simple rotation-only or random-shift alignment is insufficient.

## When NOT to use

- Input images are already co-registered single-cell feature/intensity tables rather than raw image pairs
- Only one modality is available (no paired IMC reference to warp the SIMS image onto)
- The task is downstream analysis (VAE joint embedding, segmentation, trajectory analysis) on already aligned images

## Inputs

- Paired IMC protein image patches (Histone 3, Intercalator markers) per FOV
- TOF-SIMS metabolite image patches (Phosphate 79 m/z channel) per FOV
- Raw data from Zenodo doi:10.5281/zenodo.6784251

## Outputs

- Affinely registered SIMS image aligned to the IMC reference per FOV
- Registration quality metrics (SSIM, NRMSE) comparing affine, rotation-only, and random-shift alignments (e.g., registration_metrics.csv)

## How to apply

Load paired IMC and SIMS image patches (e.g., from Zenodo deposit 10.5281/zenodo.6784251), using the Histone 3 and Intercalator IMC markers and the Phosphate 79 m/z TOF-SIMS channel as the registration anchors. Apply template matching to locate the corresponding region between the two modalities, compute the rotation offset between the aligned and cropped images, then apply the affine transformation (implemented in the scSpaMet src scripts) to warp the SIMS image onto the IMC reference. Benchmark against rotation-only and random-shift baselines using SSIM and NRMSE across FOVs; correct application should yield higher SSIM and lower NRMSE than both baselines. Persist aggregated metrics (e.g., registration_metrics.csv) and proceed to single-cell segmentation on the co-registered images.

## Related tools

- **scSpaMet** (Provides the registration pipeline: template matching, rotation-offset calculation, and custom affine transformation scripts (src folder)) — https://github.com/coskunlab/scSpaMet

## Examples

```
jupyter notebook notebooks/02_Registration_of_IMC_and_SIMS_images.ipynb  # template matching -> rotation offset -> affine transform of SIMS onto IMC
```

## Evaluation signals

- Affine registration achieves higher SSIM and lower NRMSE than rotation-only and random-shift baselines across the 24 FOVs
- Template matching correctly locates the corresponding region between IMC (Histone 3/Intercalator) and SIMS (Phosphate 79 m/z) images before rotation estimation
- Registered SIMS image overlays tissue structures visible in the IMC reference, enabling consistent single-cell segmentation masks on both modalities
- Metrics for all three conditions (affine, rotation-only, random shift) are aggregated into one table with rows per condition and columns for SSIM and NRMSE

## Limitations

- Registration depends on a shared anchor channel (Phosphate 79 m/z in TOF-SIMS; Histone 3/Intercalator in IMC); modalities lacking such markers cannot use this pipeline
- TOF-SIMS imaging regions are relatively small due to the trade-off between imaging speed and spatial resolution, limiting FOV overlap for registration
- Oct-embedded frozen tissues preserve metabolites better but preserve tissue structures less well, complicating registration; the article notes the need to incorporate cryo-TOF-SIMS imaging

## Evidence

- [results] Cross-modality single-cell registration using Histone 3 and Intercalator markers in IMC and Phosphate 79 m/z channel in TOF-SIMS: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset"
- [results] Pipeline steps: template matching, rotation offset, affine transformation: [evidence span withheld — anchor status: not_found]
- [readme] README lists registration notebooks and affine transformation source scripts: ""src" folder contains customs scripts used:
- affine transformation"
- [readme] README notebook for registration comparison: "- 12 Comparison of registration from IMC and SIMS modalities"
- [readme] Raw data location: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
