---
name: baseline-registration-ablation-analysis
description: Use when a multi-modal imaging pipeline needs quantitative proof that
  its registration step outperforms simpler baselines — e.g. affinely registered SIMS
  vs IMC patches evaluated with SSIM/NRMSE across multiple FOVs.
when_to_use_negative:
- When the goal is single-cell segmentation or downstream metabolite/protein quantification
  rather than validating the image-registration step itself
- When images come from a single modality or are already pixel-aligned, so rotation-only/random-shift
  ablation baselines are meaningless
tools:
- name: scSpaMet
  role: Provides the affine registration pipeline (template matching, rotation-offset
    calculation, affine transformation) and the registration-comparison notebook used
    to run the SSIM/NRMSE ablation
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:f1bcd5686dbeee1f3003ff6d8ca15aa73a03a32eeb884983b345f763044a1c03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# baseline-registration-ablation-analysis

## Summary

Ablation-style benchmark that scores a full affine cross-modality registration (IMC vs TOF-SIMS) against reduced baselines (rotation-only, random-shift) using SSIM and NRMSE, to demonstrate that shear/translation correction materially improves alignment.

## When to use

Use when a multi-modal imaging pipeline needs quantitative proof that its registration step outperforms simpler baselines — e.g. affinely registered SIMS vs IMC patches evaluated with SSIM/NRMSE across multiple FOVs.

## When NOT to use

- When the goal is single-cell segmentation or downstream metabolite/protein quantification rather than validating the image-registration step itself
- When images come from a single modality or are already pixel-aligned, so rotation-only/random-shift ablation baselines are meaningless

## Inputs

- Paired IMC and SIMS image patches from Zenodo deposit 10.5281/zenodo.6784251 (24 FOVs)
- IMC channels: Histone 3 and Intercalator markers (registration reference)
- TOF-SIMS channel: Phosphate 79 m/z (moving image for registration)
- Affine registration outputs from the scSpaMet pipeline (template matching + rotation offset + affine transform)

## Outputs

- registration_metrics.csv — rows: affine, rotation-only, random-shift; columns: SSIM, NRMSE per FOV
- Per-condition SSIM and NRMSE values aggregated across 24 FOVs

## How to apply

Load paired IMC and SIMS image patches from the Zenodo deposit (10.5281/zenodo.6784251), which were registered using the Histone 3 and Intercalator IMC markers and the Phosphate 79 m/z TOF-SIMS channel. Run the scSpaMet registration pipeline — template matching to locate the corresponding region, rotation-offset calculation, then affine transformation of the SIMS image onto the IMC reference — and compute SSIM and NRMSE between the affinely registered SIMS image and the IMC reference. Then recompute the same metrics for two ablation baselines: a rotation-only alignment without shear/translation correction, and a random-shift registration. Aggregate all three conditions into a single table (rows = affine, rotation-only, random shift; columns = SSIM, NRMSE), e.g. registration_metrics.csv, across all 24 FOVs. Validation: the affine condition must show higher SSIM and lower NRMSE than both baselines; scSpaMet notebook 12 ('Comparison of registration from IMC and SIMS modalities') supports this comparison.

## Related tools

- **scSpaMet** (Provides the affine registration pipeline (template matching, rotation-offset calculation, affine transformation) and the registration-comparison notebook used to run the SSIM/NRMSE ablation) — github.com/coskunlab/ScSpaMet

## Examples

```
jupyter notebook notebooks/12_Comparison_of_registration_from_IMC_and_SIMS_modalities.ipynb
```

## Evaluation signals

- Affine registration achieves higher SSIM than both rotation-only and random-shift baselines across all 24 FOVs
- Affine registration achieves lower NRMSE than both baselines across all 24 FOVs
- registration_metrics.csv contains exactly three condition rows (affine, rotation-only, random-shift) with SSIM and NRMSE columns
- Random-shift baseline yields the worst (lowest SSIM, highest NRMSE) metrics, consistent with uninformative alignment

## Limitations

- TOF-SIMS imaging yields relatively small imaging regions per tissue sample due to the trade-off between imaging speed and spatial resolution, limiting the number of FOVs available for benchmarking
- Metabolite contrast in TOF-SIMS is weaker for registration than protein contrast in IMC; alignment relies on the Phosphate 79 m/z channel only

## Evidence

- [results] a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset"
- [results] Template matching, rotation offset calculation, then affine transformation to obtain bi-modal matched images: [evidence span withheld — anchor status: not_found]
- [readme] Notebook 12 supports the registration ablation comparison: "12 Comparison of registration from IMC and SIMS modalities"
- [readme] Raw paired data source for the benchmark: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
