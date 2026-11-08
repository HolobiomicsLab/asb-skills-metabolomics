---
name: msi-he-image-overlay-compositing
description: Use when imzML MSI peptide ion images and a registered/binarized H&E
  histology image must be merged into per-protein overlay panels, e.g. after co-registration
  and LC-MS/MS peptide filtering are already done.
when_to_use_negative:
- The H&E and MSI datasets have not yet been co-registered — the registration matrix
  from register_he_msi is a required input, so applying this skill before the linear/non-linear
  registration step will misalign ion images.
- Peptide identifications have not been filtered by untargeted LC-MS/MS confirmation
  — unconfirmed m/z targets would produce low-confidence overlays, contrary to MSIght's
  design of confirming putative MSI identifications with LC-MS/MS.
- The goal is segmentation or clustering of MSI pixels (MSIght.refactor_segment.cluster_msi)
  rather than protein-level histology overlay visualization.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: MSIght
  role: Platform providing extract_mz_image_transform, merge_all_results_gene_wise,
    and overlay_msi_he functions used to extract, transform, aggregate, and overlay
    ion images
  repo: https://github.com/lingjunli-research/MSIght
- name: pyimzML
  role: Parses the imzML dataset to extract per-peptide m/z ion images
- name: OpenCV
  role: cv2.addWeighted performs the weighted blend of the summed protein image with
    the binarized H&E image
- name: Jupyter
  role: Notebook interface through which the analysis pipeline is run step-by-step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:bdf538afae93322b3584904e4ca9ea8856b87952ff7fdf58dacbae4fa7eb2be1
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
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# msi-he-image-overlay-compositing

## Summary

Compositing skill from the MSIght platform (J. Proteome Res. 2025) that transforms per-peptide MALDI-MS ion images into histology coordinate space, sums them per protein, and blends the result onto a binarized H&E image to produce registered protein overlay panels (e.g., Figure 5 bladder biomarkers: smoothelin, Col6a1, fibronectin, myosin-11).

## When to use

Use when imzML MSI peptide ion images and a registered/binarized H&E histology image must be merged into per-protein overlay panels, e.g. after co-registration and LC-MS/MS peptide filtering are already done.

## When NOT to use

- The H&E and MSI datasets have not yet been co-registered — the registration matrix from register_he_msi is a required input, so applying this skill before the linear/non-linear registration step will misalign ion images.
- Peptide identifications have not been filtered by untargeted LC-MS/MS confirmation — unconfirmed m/z targets would produce low-confidence overlays, contrary to MSIght's design of confirming putative MSI identifications with LC-MS/MS.
- The goal is segmentation or clustering of MSI pixels (MSIght.refactor_segment.cluster_msi) rather than protein-level histology overlay visualization.

## Inputs

- Preprocessed imzML MALDI-MSI dataset (loaded via load_and_preprocess_imzml)
- Filtered LC-MS/MS peptide list (output of upstream peptide confirmation step, e.g. task_005)
- Affine registration matrix from the co-registration step (register_he_msi output)
- Binarized/preprocessed H&E histology image

## Outputs

- Per-protein summed m/z images aggregated gene-wise (e.g. smoothelin, Col6a1, fibronectin, myosin-11)
- H&E/MSI overlay panels saved as PNG files constituting per-protein figure panels

## How to apply

Load the preprocessed imzML dataset with MSIght.refactor_common_functions.load_and_preprocess_imzml and read the filtered LC-MS/MS peptide list from the upstream confirmation step. Compute theoretical z=1 monoisotopic m/z values for each filtered peptide and extract per-peptide ion images with MSIght.refactor_mz_image_extract.extract_mz_image_transform using a 50 ppm mass threshold. Transform each ion image into histology coordinate space with the affine registration matrix from register_he_msi (apply_bspline_transform_to_msi is available for non-linear refinement), then aggregate transformed peptide images gene-wise per protein via MSIght.refactor_he_lcmsms_msi_merge.merge_all_results_gene_wise to obtain one summed image per protein. Finally blend each summed protein image with the binarized H&E image using OpenCV cv2.addWeighted through MSIght.refactor_mz_image_extract.overlay_msi_he and export the per-protein overlay panels as PNGs. The rationale is that co-registration of histological stains to MSI datasets improves spatial localization and resolution of identified peptides.

## Related tools

- **MSIght** (Platform providing extract_mz_image_transform, merge_all_results_gene_wise, and overlay_msi_he functions used to extract, transform, aggregate, and overlay ion images) — https://github.com/lingjunli-research/MSIght
- **pyimzML** (Parses the imzML dataset to extract per-peptide m/z ion images)
- **OpenCV** (cv2.addWeighted performs the weighted blend of the summed protein image with the binarized H&E image)
- **Jupyter** (Notebook interface through which the analysis pipeline is run step-by-step)

## Examples

```
pip install msight
```

## Evaluation signals

- Each protein overlay panel is spatially consistent with the registered H&E anatomy — ion signal aligns with histological structures after the affine (and optional B-spline) transform.
- Per-peptide ion extraction succeeded within the specified 50 ppm mass threshold around the theoretical z=1 monoisotopic m/z for every filtered peptide.
- The gene-wise merge (merge_all_results_gene_wise) yields exactly one summed image per target protein (four panels for the bladder biomarkers smoothelin, Col6a1, fibronectin, myosin-11).
- Output overlay panels are saved as valid PNG files with correct dimensions matching the binarized H&E image.
- Protein coverage and image quality can be assessed per accession as in the paper's supplementary protein coverage figures (e.g. Figure S9 for protein Q70IV5).

## Limitations

- Overlays depend on the quality of the co-registration; the workflow offers B-spline non-linear refinement (apply_bspline_transform_to_msi) because affine-only registration may be insufficient.
- The skill does not confirm analyte identity — it relies on upstream untargeted LC-MS/MS peptide confirmation to discriminate isobaric and isomeric species.
- MSIght is written in Python and distributed as a Jupyter notebook workflow; users without programming experience may need the user manual, documentation, or video tutorial provided by the Lingjun Li Lab.

## Evidence

- [other] MSIght extracts each peptide's m/z image via pyimzML (50 ppm threshold), transforms it with the calculated registration matrix: [evidence span withheld — anchor status: not_found]
- [other] Blend each summed protein image with the binarized H&E image: [evidence span withheld — anchor status: not_found]
