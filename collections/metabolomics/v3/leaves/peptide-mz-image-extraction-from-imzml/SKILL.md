---
name: peptide-mz-image-extraction-from-imzml
description: Use when you have an imzML MALDI-MSI dataset plus an LC-MS/MS-confirmed
  peptide list and need per-protein spatial ion images co-registered to H&E histology.
when_to_use_negative:
- No H&E-to-MSI registration matrix is available — images cannot be placed in histology
  coordinate space
- Peptide identifications have not been filtered/confirmed by LC-MS/MS, so the putative
  m/z list is unvetted
- Goal is unsupervised segmentation rather than targeted protein imaging (MSIght.refactor_segment.cluster_msi
  is the appropriate path)
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3653
- http://edamontology.org/topic_3473
tools:
- name: MSIght
  role: Python package providing load_and_preprocess_imzml, extract_mz_image_transform,
    merge_all_results_gene_wise, and overlay_msi_he for peptide image extraction,
    transformation, aggregation, and H&E blending
  repo: https://github.com/lingjunli-research/MSIght
- name: pyimzML
  role: Reads the imzML file and extracts each peptide's m/z image at a 50 ppm mass
    threshold
- name: OpenCV
  role: cv2.addWeighted blends each summed protein image with the binarized H&E image
    for the overlay panels
- name: Jupyter
  role: Notebook application of the MSIght analysis pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:0699cbdff21d64286bce72d78439140861e77c3de81f70e15a45321a7cda0ca8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - OpenCV
  - Python
  - pyimzML
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3653
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# peptide-mz-image-extraction-from-imzml

## Summary

Extracts per-peptide ion images from imzML mass spectrometry imaging data (50 ppm mass threshold), transforms them into histology coordinate space using the affine co-registration matrix, sums them per protein, and blends the result onto the binarized H&E image with OpenCV addWeighted to produce registered protein MALDI-MS overlays.

## When to use

Use when you have an imzML MALDI-MSI dataset plus an LC-MS/MS-confirmed peptide list and need per-protein spatial ion images co-registered to H&E histology.

## When NOT to use

- No H&E-to-MSI registration matrix is available — images cannot be placed in histology coordinate space
- Peptide identifications have not been filtered/confirmed by LC-MS/MS, so the putative m/z list is unvetted
- Goal is unsupervised segmentation rather than targeted protein imaging (MSIght.refactor_segment.cluster_msi is the appropriate path)

## Inputs

- imzML MSI dataset (preprocessed)
- Filtered LC-MS/MS peptide identification list (z=1 monoisotopic m/z computed per peptide)
- Affine registration matrix from the H&E-to-MSI co-registration step (register_he_msi output)
- Binarized, preprocessed H&E histology image

## Outputs

- Per-peptide m/z ion images transformed into histology coordinate space
- Summed per-protein m/z images (gene-wise aggregation, e.g. smoothelin, Col6a1, fibronectin, myosin-11)
- Per-protein H&E/MSI overlay panels saved as PNG files

## How to apply

Load and preprocess the imzML dataset with MSIght.refactor_common_functions.load_and_preprocess_imzml and read the filtered LC-MS/MS peptide list. Compute the theoretical z=1 monoisotopic m/z for each filtered peptide and extract per-peptide ion images with MSIght.refactor_mz_image_extract.extract_mz_image_transform using a 50 ppm mass threshold. Transform each extracted image into histology coordinate space with the affine registration matrix from register_he_msi (extract_mz_image_transform; apply_bspline_transform_to_msi is available for non-linear refinement). Aggregate transformed peptide images gene-wise per protein with MSIght.refactor_he_lcmsms_msi_merge.merge_all_results_gene_wise, then blend each summed protein image with the binarized H&E image via cv2.addWeighted in MSIght.refactor_mz_image_extract.overlay_msi_he and save the per-protein overlay panels as PNGs.

## Related tools

- **MSIght** (Python package providing load_and_preprocess_imzml, extract_mz_image_transform, merge_all_results_gene_wise, and overlay_msi_he for peptide image extraction, transformation, aggregation, and H&E blending) — https://github.com/lingjunli-research/MSIght
- **pyimzML** (Reads the imzML file and extracts each peptide's m/z image at a 50 ppm mass threshold)
- **OpenCV** (cv2.addWeighted blends each summed protein image with the binarized H&E image for the overlay panels)
- **Jupyter** (Notebook application of the MSIght analysis pipeline)

## Examples

```
pip install msight; from MSIght.refactor_mz_image_extract import extract_mz_image_transform
```

## Evaluation signals

- Each extracted peptide ion image matches its theoretical z=1 monoisotopic m/z within the 50 ppm threshold
- Transformed peptide images share the pixel grid/dimensions of the binarized H&E image (registration applied correctly)
- One summed image is produced per target protein after gene-wise aggregation (e.g. exactly four panels for smoothelin, Col6a1, fibronectin, myosin-11)
- Overlay PNGs are saved and show protein signal aligned with histological structures rather than offset from them
- Spatial localization is improved relative to unregistered images, consistent with the paper's co-registration claim

## Limitations

- Image placement depends on the quality of the affine registration matrix; non-linear BSpline refinement (apply_bspline_transform_to_msi) is available but optional
- The 50 ppm mass threshold can admit isobaric species, so extraction relies on prior LC-MS/MS confirmation of putative identifications
- Requires both modalities (imzML MSI and H&E) from the same tissue section pipeline; cannot produce histology overlays from MSI data alone

## Evidence
