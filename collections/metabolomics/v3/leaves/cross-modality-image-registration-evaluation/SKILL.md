---
name: cross-modality-image-registration-evaluation
description: Use when paired multi-modal images (e.g., TOF-SIMS metabolite and IMC
  protein channels) need alignment and you must quantitatively verify that a registration
  method beats simpler baselines.
when_to_use_negative:
- Images are already co-registered or from a single modality, so no cross-modality
  alignment evaluation is needed
- The task is single-cell segmentation or downstream clustering rather than judging
  image alignment quality
- The modalities lack shared structural anchors (e.g., no IMC marker and SIMS channel
  with corresponding tissue structure) needed for template matching
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3384
- http://edamontology.org/topic_3520
tools:
- name: scSpaMet
  role: Provides the registration pipeline scripts (template matching, rotation offset
    calculation, affine transformation) and the notebook for comparing registration
    from IMC and SIMS modalities
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:f60bf02012a146521f342565505b4b0417cd902c7b560e84b6ab92e28d543f57
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3384
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-modality-image-registration-evaluation

## Summary

Evaluates cross-modality registration of TOF-SIMS and IMC images by comparing a full affine pipeline (template matching, rotation offset, affine transformation) against rotation-only and random-shift baselines using SSIM and NRMSE over multiple FOVs.

## When to use

Use when paired multi-modal images (e.g., TOF-SIMS metabolite and IMC protein channels) need alignment and you must quantitatively verify that a registration method beats simpler baselines.

## When NOT to use

- Images are already co-registered or from a single modality, so no cross-modality alignment evaluation is needed
- The task is single-cell segmentation or downstream clustering rather than judging image alignment quality
- The modalities lack shared structural anchors (e.g., no IMC marker and SIMS channel with corresponding tissue structure) needed for template matching

## Inputs

- Paired IMC and SIMS image patches from Zenodo deposit 10.5281/zenodo.6784251
- IMC registration markers: Histone 3 and Intercalator channels
- TOF-SIMS registration channel: Phosphate 79 m/z
- 24 fields of view (FOVs)

## Outputs

- registration_metrics.csv with rows for affine, rotation-only, and random-shift and columns for SSIM and NRMSE
- Affinely registered SIMS images matched to IMC references

## How to apply

Load paired IMC and SIMS image patches from the Zenodo deposit (10.5281/zenodo.6784251), which were registered using the Histone 3 and Intercalator IMC markers and the Phosphate 79 m/z TOF-SIMS channel. Apply the scSpaMet pipeline: template matching to locate the corresponding region, rotation-offset calculation between the aligned and cropped images, then affine transformation of the SIMS image onto the IMC reference. Compute SSIM and NRMSE between the affinely registered SIMS image and the IMC reference across the 24 FOVs, then recompute both metrics for a rotation-only alignment (no shear/translation correction) and a random-shift baseline. Aggregate all three conditions into a table with rows for affine, rotation-only, and random shift and columns for SSIM and NRMSE (registration_metrics.csv). Rationale: a correct cross-modality registration should outperform naive alignments, so metric superiority over both baselines is the acceptance criterion.

## Related tools

- **scSpaMet** (Provides the registration pipeline scripts (template matching, rotation offset calculation, affine transformation) and the notebook for comparing registration from IMC and SIMS modalities) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter notebook notebooks/12_Comparison_of_registration_from_IMC_and_SIMS.ipynb  # compute SSIM/NRMSE for affine vs. rotation-only vs. random-shift registration
```

## Evaluation signals

- Affine registration achieves higher SSIM and lower NRMSE than both rotation-only and random-shift baselines across all 24 FOVs
- The output registration_metrics.csv contains exactly three conditions (affine, rotation-only, random shift) x two metrics (SSIM, NRMSE)
- Random-shift baseline should perform worst; rotation-only should be intermediate, confirming each added transform component contributes
- Registered SIMS image visually overlays the IMC reference built from Histone 3/Intercalator markers when inspected side by side

## Limitations

- Rotation-only alignment omits shear/translation correction, so it systematically underperforms and is only a sanity-check baseline
- Relatively small imaging regions from tissue samples due to the trade-off between imaging speed and spatial resolution in the TOF-SIMS limit the number of evaluable FOVs
- Registration depends on the availability of corresponding structural channels (Histone 3/Intercalator in IMC vs. Phosphate 79 m/z in TOF-SIMS)

## Evidence

- [results] Cross-modality registration using Histone 3 and Intercalator markers in IMC and Phosphate 79 m/z channel in TOF-SIMS: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset"
- [results] Registration pipeline steps: [evidence span withheld — anchor status: not_found]
- [readme] Registration notebook in README: "12 Comparison of registration from IMC and SIMS modalities"
- [readme] Affine transformation source code: ""src" folder contains customs scripts used:
- affine transformation"
- [intro] Raw data availability: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
