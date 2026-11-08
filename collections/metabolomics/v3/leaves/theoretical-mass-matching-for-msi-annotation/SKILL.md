---
name: theoretical-mass-matching-for-msi-annotation
description: Use when filtered LC-MS/MS peptide identifications and a co-registered
  imzML/H&E pair are available and per-protein spatial ion images must be annotated
  with LC-MS/MS-level confidence (e.g., bladder biomarkers smoothelin, Col6a1, fibronectin,
  myosin-11).
when_to_use_negative:
- Peptide identifications have not yet been filtered by the LC-MS/MS confirmation
  step — the 50 ppm theoretical mass matching depends on that curated peptide list
- The H&E and MSI datasets have not been co-registered (no affine registration matrix
  available), since peptide images cannot be placed in histology coordinate space
- The goal is untargeted MSI peak discovery or segmentation (e.g., MSIght.refactor_segment.cluster_msi)
  rather than LC-MS/MS-confirmed protein imaging
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MSIght
  role: Platform performing theoretical m/z calculation, per-peptide ion-image extraction
    (extract_mz_image_transform), gene-wise summation (merge_all_results_gene_wise),
    and H&E overlay (overlay_msi_he)
  repo: https://github.com/lingjunli-research/MSIght
- name: OpenCV
  role: cv2.addWeighted blends the summed protein image with the binarized H&E image
    to produce the overlay panels
- name: Python
  role: Implementation language for the modular extraction, transformation, aggregation,
    and overlay pipeline
- name: Jupyter
  role: Distributable notebook interface explaining the analysis pipeline step-by-step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:b6fe5c3c26ee5dda426c2fba3e572f53107bcbe5bdd11a26490be1b8df1b7bcc
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# theoretical-mass-matching-for-msi-annotation

## Summary

Generates confident, per-protein MALDI-MS images by computing theoretical z=1 monoisotopic m/z values for LC-MS/MS-confirmed peptides, extracting each peptide's m/z image from the imzML dataset (50 ppm threshold), and summing the transformed images gene-wise onto a co-registered, binarized H&E overlay.

## When to use

Use when filtered LC-MS/MS peptide identifications and a co-registered imzML/H&E pair are available and per-protein spatial ion images must be annotated with LC-MS/MS-level confidence (e.g., bladder biomarkers smoothelin, Col6a1, fibronectin, myosin-11).

## When NOT to use

- Peptide identifications have not yet been filtered by the LC-MS/MS confirmation step — the 50 ppm theoretical mass matching depends on that curated peptide list
- The H&E and MSI datasets have not been co-registered (no affine registration matrix available), since peptide images cannot be placed in histology coordinate space
- The goal is untargeted MSI peak discovery or segmentation (e.g., MSIght.refactor_segment.cluster_msi) rather than LC-MS/MS-confirmed protein imaging

## Inputs

- Preprocessed imzML MALDI-MSI dataset (via MSIght.refactor_common_functions.load_and_preprocess_imzml)
- Filtered LC-MS/MS peptide identification list from the upstream confirmation step (task_005 output)
- Affine registration matrix from the co-registration step (register_he_msi output)
- Binarized H&E histology image

## Outputs

- Per-peptide m/z ion images transformed into histology coordinate space
- Gene-wise summed per-protein MSI images (one per biomarker, e.g., smoothelin, Col6a1, fibronectin, myosin-11)
- H&E/MSI blended overlay panels saved as PNG files

## How to apply

Load the preprocessed imzML dataset with MSIght.refactor_common_functions.load_and_preprocess_imzml and read the filtered peptide list from the upstream LC-MS/MS confirmation step. Compute each peptide's theoretical z=1 monoisotopic m/z and extract its ion image via MSIght.refactor_mz_image_extract.extract_mz_image_transform using a 50 ppm mass tolerance. Transform each peptide image into histology coordinate space with the affine registration matrix from register_he_msi (B-spline non-linear refinement available via apply_bspline_transform_to_msi). Sum transformed peptide images per protein with MSIght.refactor_he_lcmsms_msi_merge.merge_all_results_gene_wise, then blend each summed protein image with the binarized H&E image using OpenCV cv2.addWeighted through MSIght.refactor_mz_image_extract.overlay_msi_he, saving per-protein overlay PNGs.

## Related tools

- **MSIght** (Platform performing theoretical m/z calculation, per-peptide ion-image extraction (extract_mz_image_transform), gene-wise summation (merge_all_results_gene_wise), and H&E overlay (overlay_msi_he)) — https://github.com/lingjunli-research/MSIght
- **OpenCV** (cv2.addWeighted blends the summed protein image with the binarized H&E image to produce the overlay panels)
- **Python** (Implementation language for the modular extraction, transformation, aggregation, and overlay pipeline)
- **Jupyter** (Distributable notebook interface explaining the analysis pipeline step-by-step)

## Examples

```
from MSIght.refactor_mz_image_extract import extract_mz_image_transform, overlay_msi_he; img = extract_mz_image_transform(imzml_data, peptide_mz, ppm=50, reg_matrix=matrix); overlay_msi_he(img, he_binary, out_png)
```

## Evaluation signals

- Each per-peptide image is extracted with a 50 ppm mass threshold around the theoretical z=1 monoisotopic m/z — verify reported m/z deltas fall within ±50 ppm
- Peptide images must be spatially transformed with the affine registration matrix (optionally B-spline refined) before aggregation; check output dimensions match the H&E pixel grid
- The gene-wise aggregation must yield exactly one summed image per protein (e.g., four images for smoothelin, Col6a1, fibronectin, myosin-11)
- Overlay PNGs should show the summed protein signal visibly registered to tissue structures in the binarized H&E image
- Protein coverage and image quality can be assessed as in the supplementary protein coverage figures (e.g., Figure S9 for protein Q70IV5)

## Limitations

- Confidence of annotation is bounded by the upstream untargeted LC-MS/MS peptide confirmations; MALDI-MSI alone cannot discriminate isobaric and isomeric species
- Linear registration may be insufficient — a non-linear B-spline refinement step (apply_bspline_transform_to_msi) is available for cases where the affine matrix alone misaligns tissue
- The 50 ppm mass threshold is fixed in this workflow; narrower or wider tolerances are not validated here

## Evidence
