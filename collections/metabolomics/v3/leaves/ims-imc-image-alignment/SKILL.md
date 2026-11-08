---
name: ims-imc-image-alignment
description: Use when paired TOF-SIMS (metabolite) and IMC (protein) images of the
  same tissue FOV need geometric alignment before single-cell segmentation or joint
  protein-metabolite analysis.
when_to_use_negative:
- Input is a single modality (IMC-only or SIMS-only) with no paired channel of the
  same tissue region — template matching has no correspondence to find.
- The image pair lacks the registration markers (Histone 3/Intercalator in IMC and
  the Phosphate 79 m/z channel in TOF-SIMS), so the cross-modality template match
  cannot be computed.
- The goal is downstream single-cell segmentation, VAE joint embedding, or trajectory
  analysis on already-registered data — use the later scSpaMet notebooks (03–08) instead.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scSpaMet
  role: 'Provides the registration pipeline: template matching, rotation-offset calculation,
    and affine transformation scripts (src) plus registration and registration-comparison
    notebooks (02 and 12).'
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:ef480ad7f6c76e9b8bb2b28930abe7fb8cf9da4f3891bba313f049aa57c0881e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# IMS-IMC image alignment

## Summary

Cross-modality registration of TOF-SIMS metabolite images to IMC protein images using template matching, rotation-offset calculation, and an affine transformation, as implemented in the scSpaMet pipeline so that single-cell protein and metabolite signals land in a shared coordinate frame.

## When to use

Use when paired TOF-SIMS (metabolite) and IMC (protein) images of the same tissue FOV need geometric alignment before single-cell segmentation or joint protein-metabolite analysis.

## When NOT to use

- Input is a single modality (IMC-only or SIMS-only) with no paired channel of the same tissue region — template matching has no correspondence to find.
- The image pair lacks the registration markers (Histone 3/Intercalator in IMC and the Phosphate 79 m/z channel in TOF-SIMS), so the cross-modality template match cannot be computed.
- The goal is downstream single-cell segmentation, VAE joint embedding, or trajectory analysis on already-registered data — use the later scSpaMet notebooks (03–08) instead.

## Inputs

- Paired IMC and SIMS image patches per FOV from the Zenodo deposit (10.5281/zenodo.6784251)
- IMC Histone 3 and Intercalator marker channels (registration reference)
- TOF-SIMS Phosphate 79 m/z channel (moving image to be registered)

## Outputs

- Affinely registered SIMS image aligned to the IMC reference (bi-modal matched images)
- Rotation offset and affine transformation parameters per imaging region
- registration_metrics.csv with SSIM and NRMSE rows for affine, rotation-only, and random-shift conditions

## How to apply

Load the paired IMC and SIMS patches (raw data from Zenodo 10.5281/zenodo.6784251) and run the scSpaMet registration pipeline (notebook 02, notebook 12 for comparison, and the affine transformation scripts in src). First, use the template matching algorithm to locate the corresponding region using Histone 3 and Intercalator markers in the IMC dataset and the Phosphate 79 m/z channel in the TOF-SIMS dataset; crop both images to that region. Next, calculate the rotation offset between the two aligned and cropped images. Finally, apply the full affine transformation of the SIMS image onto the IMC image to correct residual shear/translation. To validate, compute SSIM and NRMSE between the affinely registered SIMS image and the IMC reference, and benchmark against rotation-only (no shear/translation correction) and random-shift baselines: the affine result should achieve higher SSIM and lower NRMSE across all FOVs (e.g., 24 FOVs in the benchmark task), aggregated into registration_metrics.csv.

## Related tools

- **scSpaMet** (Provides the registration pipeline: template matching, rotation-offset calculation, and affine transformation scripts (src) plus registration and registration-comparison notebooks (02 and 12).) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter notebook "notebooks/02 Registration of IMC and SIMS images for different imaging regions"
```

## Evaluation signals

- Affine condition attains higher SSIM and lower NRMSE than both rotation-only and random-shift baselines across all FOVs (24 in the benchmark).
- registration_metrics.csv contains exactly three conditions (affine, rotation-only, random-shift) as rows with SSIM and NRMSE columns.
- The pipeline order is respected: template matching (region located) → rotation-offset calculation on cropped images → affine transformation.
- Registered SIMS Phosphate 79 m/z signal spatially overlaps the IMC Histone 3/Intercalator reference channels after transformation.
- Notebook 12 (comparison of registration from IMC and SIMS modalities) reproduces the metric comparison without errors.

## Limitations

- Registration depends on specific cross-modality markers (Histone 3 and Intercalator in IMC; Phosphate 79 m/z in TOF-SIMS); images lacking these channels cannot be aligned by this pipeline.
- Rotation-only alignment omits shear/translation correction and measurably underperforms the full affine transform, so partial alignment steps alone are insufficient.
- Registration is performed per imaging region/FOV; a single global transform across regions is not supported by the workflow.

## Evidence

- [results] a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset"
- [results] Template matching, rotation offset calculation, then affine transformation to obtain bi-modal matched images: [evidence span withheld — anchor status: not_found]
- [readme] 02 Registration of IMC and SIMS images for different imaging regions: "02 Registration of IMC and SIMS images for different imaging regions"
- [readme] You can find the raw data here: https://doi.org/10.5281/zenodo.6784251.: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
- [readme] 12 Comparison of registration from IMC and SIMS modalities: "12 Comparison of registration from IMC and SIMS modalities"
