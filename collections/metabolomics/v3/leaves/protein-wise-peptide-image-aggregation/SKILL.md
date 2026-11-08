---
name: protein-wise-peptide-image-aggregation
description: Use when you have a filtered LC-MS/MS peptide list mapped to proteins
  and per-peptide m/z ion images from an imzML MSI dataset that must be summed gene-wise
  and overlaid on registered histology.
when_to_use_negative:
- The peptide list has not yet been filtered by LC-MS/MS confirmation — aggregating
  unconfirmed peptides defeats the confidence-gaining purpose of the MSIght workflow.
- The H&E-to-MSI registration matrix has not been computed; peptide images must be
  transformed into histology space before gene-wise summation.
- The goal is a single-m/z ion image or small-molecule MSI image rather than peptide-level
  images that must be summed per protein.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MSIght
  role: 'Provides the full pipeline functions: extract_mz_image_transform (50 ppm
    peptide image extraction and registration transform), merge_all_results_gene_wise
    (protein-wise summation), and overlay_msi_he (cv2.addWeighted blending with binarized
    H&E)'
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language of the MSIght package in which the extraction, transformation,
    gene-wise aggregation, and overlay functions are written
- name: OpenCV
  role: cv2.addWeighted is used to blend each summed protein image with the binarized
    H&E image for the final overlay
- name: Jupyter
  role: Distributable notebook interface for running the MSIght analysis pipeline
    step-by-step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:4bda14668d97d8f02042e921185a284f8187bd84b80a97936f05af6cd9900de2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - OpenCV
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# protein-wise-peptide-image-aggregation

## Summary

Aggregates transformed, per-peptide MALDI-MS ion images into a single summed image per protein (e.g., smoothelin, Col6a1, fibronectin, myosin-11) and overlays the result on the binarized H&E histology image, as implemented in MSIght for Figure 5-style biomarker panels.

## When to use

Use when you have a filtered LC-MS/MS peptide list mapped to proteins and per-peptide m/z ion images from an imzML MSI dataset that must be summed gene-wise and overlaid on registered histology.

## When NOT to use

- The peptide list has not yet been filtered by LC-MS/MS confirmation — aggregating unconfirmed peptides defeats the confidence-gaining purpose of the MSIght workflow.
- The H&E-to-MSI registration matrix has not been computed; peptide images must be transformed into histology space before gene-wise summation.
- The goal is a single-m/z ion image or small-molecule MSI image rather than peptide-level images that must be summed per protein.

## Inputs

- imzML MSI dataset (preprocessed with MSIght load_and_preprocess_imzml)
- Filtered LC-MS/MS peptide list with protein (gene-wise) assignments from the upstream peptide-confirmation step
- Affine registration matrix from the register_he_msi co-registration step
- Binarized H&E histology image

## Outputs

- One summed MALDI-MS image per protein (gene-wise aggregate of transformed peptide m/z images)
- Per-protein H&E/MSI overlay panels saved as PNG files (e.g., the four Figure 5 bladder biomarker panels)

## How to apply

Load the preprocessed imzML dataset with MSIght.refactor_common_functions.load_and_preprocess_imzml and read the filtered LC-MS/MS peptide list from the upstream confirmation step. Compute each peptide's theoretical z=1 monoisotopic m/z and extract its ion image with MSIght.refactor_mz_image_extract.extract_mz_image_transform using a 50 ppm mass tolerance. Transform each peptide image into histology coordinate space using the affine registration matrix from the register_he_msi co-registration step (with apply_bspline_transform_to_msi available for non-linear refinement). Aggregate the transformed peptide images gene-wise per protein with MSIght.refactor_he_lcmsms_msi_merge.merge_all_results_gene_wise, then blend each summed protein image with the binarized H&E image via OpenCV cv2.addWeighted using MSIght.refactor_mz_image_extract.overlay_msi_he to produce the final per-protein overlay panels.

## Related tools

- **MSIght** (Provides the full pipeline functions: extract_mz_image_transform (50 ppm peptide image extraction and registration transform), merge_all_results_gene_wise (protein-wise summation), and overlay_msi_he (cv2.addWeighted blending with binarized H&E)) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language of the MSIght package in which the extraction, transformation, gene-wise aggregation, and overlay functions are written)
- **OpenCV** (cv2.addWeighted is used to blend each summed protein image with the binarized H&E image for the final overlay)
- **Jupyter** (Distributable notebook interface for running the MSIght analysis pipeline step-by-step)

## Examples

```
pip install msight
```

## Evaluation signals

- Each peptide image was extracted within the 50 ppm mass tolerance of its theoretical z=1 monoisotopic m/z.
- Transformed peptide images spatially align with the binarized H&E image boundaries after application of the affine registration matrix.
- Exactly one summed image is produced per target protein (e.g., four images for smoothelin, Col6a1, fibronectin, myosin-11).
- Four per-protein H&E/MSI overlay PNG panels are saved as the Figure 5 outputs.
- Overlay panels show protein signal localized to histological structures, consistent with the co-registered stain improving spatial localization.

## Limitations

- Accuracy depends on the quality of the upstream affine (and optional B-spline non-linear) registration; misregistration propagates into the aggregated protein image.
- Aggregation is only as confident as the filtered LC-MS/MS peptide list; unconfirmed or isobaric peptide identifications contaminate the summed protein image.
- MSIght is Python-based and assumes the user can run code, unlike no-code alternatives such as MSiReader; R-based Cardinal users must adapt.

## Evidence
