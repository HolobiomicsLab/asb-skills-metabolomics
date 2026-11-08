---
name: isotopic-pattern-prediction-for-lipids
description: Use when MSI lipidomics ion images contain same-class lipid species differing
  by one double bond (e.g., PC 38:3 vs PC 38:2, PE 38:3 vs PE 38:4) whose monoisotopic
  signals are inflated by the M+2 isotopologue of the more unsaturated isobar.
when_to_use_negative:
- The interference is Na/H adduct overlap (Δm/z 0.0025) rather than Type II isotopologue
  overlap — that is resolved from sodiated-to-protonated adduct ratios of standards,
  not from predicted isotopic patterns.
- The candidate isobars belong to different lipid classes or differ by more than one
  double bond — correction is restricted to same-class single-double-bond pairs identified
  from the database.
- No lipid database annotation exists for the isobaric pairs; both the pair identification
  and the predicted pattern depend on database columns such as lipid_class, adduct,
  and neutral_id.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: LipidQMap
  role: Implements the Type II isobaric correction based on predicted isotope patterns,
    applied sequentially after Na/H adduct correction; corrected results viewable
    via the Raw/Isotope corrected/Quantitative toggle
  repo: https://github.com/swinnenteam/LipidQMap
- name: Cardinal
  role: Source of the exported HDF5 MSI representation (featureData/feature_id, pixelData/pixel_index)
    that is loaded with numpy/h5py for the correction
- name: NumPy
  role: Constructs the per-pixel correction matrix and stores the corrected (n_features,
    n_pixels) float32 array with np.save
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:f8e766d9e33f6ef7d5c32f8578f86dae1be11855cd4f0d5d68caed6d7c29b329
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/swinnenteam/LipidQMap
  tools:
  - Cardinal
  - LipidQMap
  - NumPy
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# isotopic-pattern-prediction-for-lipids

## Summary

Predicts the relative isotopic intensities (M, M+1, M+2, ...) of annotated lipid species and uses the predicted pattern to mathematically resolve Type II isobaric overlap in MSI lipidomics — e.g., subtracting the M+2 isotopologue contribution of PC 38:4 from the monoisotopic ion image of PC 38:3 — as implemented in LipidQMap.

## When to use

Use when MSI lipidomics ion images contain same-class lipid species differing by one double bond (e.g., PC 38:3 vs PC 38:2, PE 38:3 vs PE 38:4) whose monoisotopic signals are inflated by the M+2 isotopologue of the more unsaturated isobar.

## When NOT to use

- The interference is Na/H adduct overlap (Δm/z 0.0025) rather than Type II isotopologue overlap — that is resolved from sodiated-to-protonated adduct ratios of standards, not from predicted isotopic patterns.
- The candidate isobars belong to different lipid classes or differ by more than one double bond — correction is restricted to same-class single-double-bond pairs identified from the database.
- No lipid database annotation exists for the isobaric pairs; both the pair identification and the predicted pattern depend on database columns such as lipid_class, adduct, and neutral_id.

## Inputs

- Centroided MSI ion images (feature-by-pixel intensity matrix) from a Cardinal-exported HDF5 file (or .imzML read by LipidQMap), keyed by featureData/feature_id and pixelData/pixel_index
- Lipid annotation database (.csv with nine required columns, or featureData with lipid_class, adduct, neutral_id, and mz float32 columns)
- Predicted relative isotopic intensities (M, M+1, M+2, ...) per annotated lipid species

## Outputs

- Corrected feature-by-pixel intensity array of shape (n_features, n_pixels), float32, e.g., corrected_img.npy
- Deconvolved ion images in which the less unsaturated species of each single-double-bond pair no longer contains the more unsaturated species' M+2 contribution

## How to apply

Load the centroided ion images and lipid annotations from the Cardinal-exported HDF5 with numpy and h5py, joining on featureData/feature_id and pixelData/pixel_index, and mine the lipid database (lipid_class, adduct, neutral_id, mz float32 columns) for same-class pairs differing by exactly one double bond whose monoisotopic m/z values overlap (e.g., PE 38:3 vs PE 38:4, PC 34:0 vs PC 34:1). For each pair, compute the predicted isotopic pattern (M, M+1, M+2, ...) of the more unsaturated species and determine the fractional contribution of its M+2 isotopologue at the adjacent m/z of the less unsaturated partner. Assemble these fractions into a correction matrix and subtract, per pixel, the predicted isotopic contribution of each isobar from the other's measured intensity — in LipidQMap the Na/H adduct and Type II corrections are applied sequentially, so resolve prevalent M+Na interference first. Store the corrected feature-by-pixel intensity array of shape (n_features, n_pixels) as float32 via np.save (e.g., corrected_img.npy). The rationale is that the measured monoisotopic intensity of the less unsaturated lipid is systematically biased upward by the unresolved M+2 isotopologue of its one-more-double-bond isobar, and the predicted isotope pattern quantifies that bias exactly. Note the correction performs optimally with protonating matrices, so verify behavior on matrix-specific acquisitions before trusting absolute values.

## Related tools

- **LipidQMap** (Implements the Type II isobaric correction based on predicted isotope patterns, applied sequentially after Na/H adduct correction; corrected results viewable via the Raw/Isotope corrected/Quantitative toggle) — https://github.com/swinnenteam/LipidQMap
- **Cardinal** (Source of the exported HDF5 MSI representation (featureData/feature_id, pixelData/pixel_index) that is loaded with numpy/h5py for the correction)
- **NumPy** (Constructs the per-pixel correction matrix and stores the corrected (n_features, n_pixels) float32 array with np.save)

## Examples

```
import numpy as np, h5py; f=h5py.File('cardinal_export.h5'); corr=np.float32(f['intensities'][:]-m2_matrix); np.save('corrected_img.npy', corr); f.close()
```

## Evaluation signals

- Corrected intensities are non-negative for all corrected isobaric pairs.
- The corrected array shape equals (n_features, n_pixels) of the source HDF5.
- Isotope contributions were subtracted only between same-class single-double-bond isobar pairs identified from the database (e.g., PC 34:0 vs PC 34:1); no cross-class corrections appear.
- Per pixel, the subtracted amount equals the predicted M+2 relative isotopic intensity of the more unsaturated species scaled by its measured monoisotopic intensity at that pixel.
- In LipidQMap, toggling between the Raw and Isotope corrected views shows the expected intensity reduction for the less unsaturated species in regions dominated by the more unsaturated isobar.

## Limitations

- Correction performs optimally when protonating matrices are used; further research and optimization are necessary to fully leverage the correction in qMSI with various MALDI matrices.
- Na/H adduct and Type II corrections are applied sequentially; datasets with prevalent M+Na adducts depend on the adduct correction being performed first.
- Subtraction is limited to same-class species differing by exactly one double bond from the database; the accuracy of the recovered abundance rests on correct assignment of the lipid identification to the measured m/z.

## Evidence

- [abstract] LipidQMap resolves Na/H adduct isobaric overlaps by leveraging sodiated-to-protonated adduct ratios of standards, and corrects Type II isobaric overlaps using predicted isotopic patterns.: [evidence span withheld — anchor status: not_found]
- [readme] README raw vs isotope-corrected view toggle: [evidence span withheld — anchor status: external_doc]
