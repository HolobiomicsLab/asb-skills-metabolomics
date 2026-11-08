---
name: affine-transform-application-to-msi-images
description: Use when you have imzML MALDI-MSI data plus an H&E image and an already-computed
  co-registration matrix (e.g. from MSIght register_he_msi), and need per-peptide
  or per-protein ion images spatially transformed into histology coordinates for overlay.
when_to_use_negative:
- No co-registration matrix has been computed yet — the H&E and MSI must first be
  aligned (MSIght linear/non-linear registration workflow) before per-peptide images
  can be transformed
- Peptide identifications have not been filtered — the transform is applied to peptide
  images derived from the filtered LC-MS/MS list, not to untargeted peak picking on
  the raw MSI data
- Data are not peptide-level MALDI-MSI with an H&E stain, e.g. metabolite imaging
  without histology co-registration, since the whole point is protein image overlay
  on binarized H&E
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3382
tools:
- name: MSIght
  role: Provides extract_mz_image_transform (50 ppm peptide m/z image extraction +
    affine/B-spline transform), apply_bspline_transform_to_msi, merge_all_results_gene_wise,
    and overlay_msi_he (cv2.addWeighted)
  repo: https://github.com/lingjunli-research/MSIght
- name: OpenCV
  role: cv2.addWeighted blends the summed protein image with the binarized H&E image
    in overlay_msi_he
- name: Python
  role: Implementation language of the MSIght package in which the transform and overlay
    functions are written
- name: Jupyter
  role: Distributable Jupyter notebook walks through the analysis pipeline step-by-step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:79bb1ed2e5cd1c361214c02c761e6d435cc72d8df6eb53aa2a9b7da4d48e511b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - OpenCV
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# affine-transform-application-to-msi-images

## Summary

Applies the affine (and optionally B-spline non-linear) registration matrix computed during H&E/MSI co-registration to per-peptide MALDI-MS ion images, so that peptide m/z images extracted from imzML data land in histology coordinate space and can be summed per protein and overlaid on H&E.

## When to use

Use when you have imzML MALDI-MSI data plus an H&E image and an already-computed co-registration matrix (e.g. from MSIght register_he_msi), and need per-peptide or per-protein ion images spatially transformed into histology coordinates for overlay.

## When NOT to use

- No co-registration matrix has been computed yet — the H&E and MSI must first be aligned (MSIght linear/non-linear registration workflow) before per-peptide images can be transformed
- Peptide identifications have not been filtered — the transform is applied to peptide images derived from the filtered LC-MS/MS list, not to untargeted peak picking on the raw MSI data
- Data are not peptide-level MALDI-MSI with an H&E stain, e.g. metabolite imaging without histology co-registration, since the whole point is protein image overlay on binarized H&E

## Inputs

- Preprocessed imzML MALDI-MSI dataset
- Filtered LC-MS/MS peptide identification list (from the upstream LC-MS/MS confirmation step)
- Affine registration matrix from H&E/MSI co-registration (register_he_msi output)
- Binarized, preprocessed H&E histology image

## Outputs

- Per-peptide ion images transformed into histology coordinate space
- One summed, gene-wise aggregated protein image per target protein (e.g. smoothelin, Col6a1, fibronectin, myosin-11)
- H&E/MSI overlay panels saved as PNG files

## How to apply

Load the preprocessed imzML dataset and read the filtered LC-MS/MS peptide list, then compute each filtered peptide's theoretical z=1 monoisotopic m/z. Extract per-peptide ion images from the MSI data with MSIght.refactor_mz_image_extract.extract_mz_image_transform using a 50 ppm mass threshold around each theoretical m/z. Apply the affine registration matrix output by the co-registration step (register_he_msi) to each peptide image to move it into histology coordinate space; if residual misalignment remains, refine non-linearly with apply_bspline_transform_to_msi. Aggregate the transformed peptide images gene-wise per protein with MSIght.refactor_he_lcmsms_msi_merge.merge_all_results_gene_wise to yield one summed image per protein, then blend each summed protein image with the binarized H&E image via MSIght.refactor_mz_image_extract.overlay_msi_he (OpenCV cv2.addWeighted) and save the panels as PNG. The rationale is that histological stains are coregistered to MSI datasets to improve spatial localization and resolution of identified peptides, so the transform must be applied to every peptide image before protein-level summation.

## Related tools

- **MSIght** (Provides extract_mz_image_transform (50 ppm peptide m/z image extraction + affine/B-spline transform), apply_bspline_transform_to_msi, merge_all_results_gene_wise, and overlay_msi_he (cv2.addWeighted)) — https://github.com/lingjunli-research/MSIght
- **OpenCV** (cv2.addWeighted blends the summed protein image with the binarized H&E image in overlay_msi_he)
- **Python** (Implementation language of the MSIght package in which the transform and overlay functions are written)
- **Jupyter** (Distributable Jupyter notebook walks through the analysis pipeline step-by-step)

## Examples

```
from MSIght.refactor_mz_image_extract import extract_mz_image_transform, overlay_msi_he; img = extract_mz_image_transform(...); overlay_msi_he(...)
```

## Evaluation signals

- Extracted peptide ion images must fall within the 50 ppm mass threshold of the theoretical z=1 monoisotopic m/z of the filtered peptide
- After applying the affine matrix, peptide image features should align with corresponding histological structures in the binarized H&E; residual misalignment is the trigger for apply_bspline_transform_to_msi refinement
- Gene-wise aggregation must produce exactly one summed image per target protein (e.g. four panels for smoothelin, Col6a1, fibronectin, myosin-11)
- Overlay output must be a valid blended image from cv2.addWeighted and must be saved as PNG per Figure 5 panel
- Spatial localization of the summed protein image should match the histology-derived regions expected for the bladder biomarker (co-registration is intended to improve spatial localization and resolution of identified peptides)

## Limitations

- The affine transform alone may be insufficient; non-linear refinement (B-spline) is needed when tissue deformation is non-linear, which MSIght exposes as a separate optional function
- Registration quality is bounded by the upstream H&E pre-processing and linear/non-linear registration steps; misregistration propagates into every peptide and protein image
- MSIght is written for H&E + MSI + LC-MS/MS integration; other stain types or modalities are outside the demonstrated scope

## Evidence

- [abstract] Histological stains of tissue sections are coregistered to their respective MSI data sets to improve spatial localization and resolution of identified peptides.: "Histological stains of tissue sections are coregistered to their respective MSI data sets to improve spatial localization and resolution of identified peptides."
- [supplementary] Figure S7: Linear and non-linear registration workflow: "Figure S7: Linear and non-linear registration workflow"
- [readme] MSIght is a program produced by the Lingjun Li Lab at the University of Wisconsin-Madison for automated integration of H&E, MSI, and LC-MS/MS datasets: "MSIght is a program produced by the Lingjun Li Lab at the University of Wisconsin-Madison for automated integration of H&E, MSI, and LC-MS/MS datasets"
- [supplementary] MSI pre-processing workflow: "Figure S1: MSI pre-processing workflow"
