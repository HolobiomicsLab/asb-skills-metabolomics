---
name: ssim-nrmse-registration-metric-computation
description: Use when you need to score how well a registered SIMS image matches its
  paired IMC reference, or to prove that a full affine registration outperforms simpler
  baselines (rotation-only, random-shift) using SSIM and NRMSE on paired FOVs.
when_to_use_negative:
- Images are already registered and only downstream single-cell segmentation or intensity
  extraction is needed (this skill only scores alignment, it does not register images)
- The modalities share no corresponding anatomical/chemical structures to align (e.g.
  unpaired or independently sampled sections), making SSIM/NRMSE meaningless
- Evaluating biological agreement (protein-metabolite correlation) rather than geometric
  registration quality — use correlation or joint-embedding analyses instead
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: scSpaMet
  role: Provides the registration notebook (notebook 02) and the registration comparison
    notebook (notebook 12) whose outputs are scored, plus custom affine transformation
    scripts in src; SSIM/NRMSE are computed between its registered SIMS output and
    the IMC reference
  repo: github.com/coskunlab/ScSpaMet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:c13d34b8304a273c84214c744b91c618761cf5785ebdf47e69e8caa1e6bafc34
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scSpaMet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# ssim-nrmse-registration-metric-computation

## Summary

Quantifies cross-modality image alignment quality by computing structural similarity (SSIM) and normalized root-mean-square error (NRMSE) between a registered TOF-SIMS metabolite image and its IMC protein reference, and uses the same metrics to benchmark affine registration against rotation-only and random-shift baselines across 24 FOVs.

## When to use

Use when you need to score how well a registered SIMS image matches its paired IMC reference, or to prove that a full affine registration outperforms simpler baselines (rotation-only, random-shift) using SSIM and NRMSE on paired FOVs.

## When NOT to use

- Images are already registered and only downstream single-cell segmentation or intensity extraction is needed (this skill only scores alignment, it does not register images)
- The modalities share no corresponding anatomical/chemical structures to align (e.g. unpaired or independently sampled sections), making SSIM/NRMSE meaningless
- Evaluating biological agreement (protein-metabolite correlation) rather than geometric registration quality — use correlation or joint-embedding analyses instead

## Inputs

- Affinely registered TOF-SIMS image patches (Phosphate 79 m/z channel) paired with IMC patches per FOV
- Rotation-only aligned SIMS vs IMC image pairs (baseline)
- Random-shift SIMS vs IMC image pairs (baseline)
- Raw data from https://doi.org/10.5281/zenodo.6784251 (24 FOVs)

## Outputs

- registration_metrics.csv (rows: affine, rotation-only, random shift; columns: SSIM, NRMSE)
- Per-FOV SSIM values for each registration condition
- Per-FOV NRMSE values for each registration condition

## How to apply

Load paired IMC and TOF-SIMS patches from the Zenodo deposit (10.5281/zenodo.6784251), which were cross-modality registered using Histone 3 and Intercalator IMC markers against the Phosphate 79 m/z TOF-SIMS channel. For the test condition, take the scSpaMet affinely registered SIMS image (template matching to locate the corresponding region, rotation-offset calculation, then affine transformation of SIMS onto IMC) and compute SSIM and NRMSE against the IMC reference. Recompute the identical metrics for two negative controls: rotation-only alignment (no shear/translation correction) and a random-shift registration. Aggregate all three conditions into registration_metrics.csv with rows for affine, rotation-only, and random shift and columns for SSIM and NRMSE, aggregated across all 24 FOVs. Validate that the affine condition yields the highest SSIM and lowest NRMSE of the three; if it does not, the registration or metric computation is suspect.

## Related tools

- **scSpaMet** (Provides the registration notebook (notebook 02) and the registration comparison notebook (notebook 12) whose outputs are scored, plus custom affine transformation scripts in src; SSIM/NRMSE are computed between its registered SIMS output and the IMC reference) — github.com/coskunlab/ScSpaMet

## Examples

```
from skimage.metrics import structural_similarity, normalized_root_mse; ssim = structural_similarity(sims_registered, imc_ref, data_range=imc_ref.max()-imc_ref.min()); nrmse = normalized_root_mse(imc_ref, sims_registered)
```

## Evaluation signals

- Affine registration achieves higher SSIM and lower NRMSE than both rotation-only and random-shift baselines across all 24 FOVs
- registration_metrics.csv contains exactly three rows (affine, rotation-only, random shift) and two columns (SSIM, NRMSE) with finite, non-null values
- Random-shift baseline should show the worst (lowest SSIM, highest NRMSE) scores; if it competes with affine, the metric computation or pairing is wrong
- SSIM values fall in [-1, 1] and NRMSE values are non-negative for all conditions
- Metric computation uses the same registered/aligned SIMS channel (Phosphate 79 m/z) and the same IMC markers (Histone 3, Intercalator) context for all three conditions so the comparison is apples-to-apples

## Limitations

- SSIM/NRMSE quantify geometric alignment only; good registration scores do not guarantee biological or single-cell-level protein-metabolite correspondence
- The benchmark is specific to the scSpaMet cross-modality registration (IMC Histone 3/Intercalator vs TOF-SIMS Phosphate 79 m/z) and may not transfer to modalities without a shared structural channel
- TOF-SIMS imaging yields relatively small regions from tissue samples due to the trade-off between imaging speed and spatial resolution, limiting the number of evaluable FOVs

## Evidence

- [results] Cross-modality registration markers: "a cross-modality single-cell registration pipeline was developed utilizing Histone 3 and Intercalator markers in the IMC dataset, and Phosphate 79 m/z channels in TOF-SIMS dataset"
- [results] Affine registration steps: [evidence span withheld — anchor status: not_found]
- [readme] Registration comparison notebook: "12 Comparison of registration from IMC and SIMS modalities"
- [readme] Raw paired data location: "You can find the raw data here: https://doi.org/10.5281/zenodo.6784251."
