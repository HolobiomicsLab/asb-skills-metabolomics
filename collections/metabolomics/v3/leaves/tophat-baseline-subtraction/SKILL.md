---
name: tophat-baseline-subtraction
description: Use when raw imzML spectra from a MALDI-MSI experiment exhibit a slowly
  varying chemical background that must be subtracted pixel-wise before downstream
  dimensionality reduction, clustering, or MS image generation.
when_to_use_negative:
- The spectra have already been baseline-corrected upstream, so applying tophat subtraction
  would strip genuine broad signal
- The task concerns the H&E histology image or LC-MS/MS peptide-confirmation workflows,
  which use MSIght's separate pre-processing pipelines rather than the MSI spectral
  pre-processing module
- Input is not an imzML-backed MSI pixel matrix (e.g., peak-picked peak lists or LC-MS
  chromatograms), where the pixel-wise spectrum assumptions of this step do not hold
tools:
- name: MSIght
  role: Platform whose MSI pre-processing pipeline orchestrates TIC normalization,
    Gaussian smoothing (sigma=2), and white tophat baseline correction on imzML data
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language for the pre-processing module, chosen for accessibility
    and ecosystem access to SciPy morphology routines
- name: SciPy
  role: Provides the morphology operations used to execute the white tophat baseline
    subtraction with structuring element size 10
- name: pyimzML
  role: Backs loading of the raw imzML file and re-writing of the corrected spectra
    into the imzML-backed data structure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:1ccfdfcc948ef113cfd7a45e30b6723c4b053e1583b3ddf4e02101963402e86b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - Python
  - SciPy
  - pyimzML
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tophat-baseline-subtraction

## Summary

Removes broad chemical background from each pixel's spectrum in MALDI-MSI imzML data using a white tophat morphological filter with a structuring element of size 10, as implemented in the MSIght MSI pre-processing pipeline before dimensionality reduction and segmentation.

## When to use

Use when raw imzML spectra from a MALDI-MSI experiment exhibit a slowly varying chemical background that must be subtracted pixel-wise before downstream dimensionality reduction, clustering, or MS image generation.

## When NOT to use

- The spectra have already been baseline-corrected upstream, so applying tophat subtraction would strip genuine broad signal
- The task concerns the H&E histology image or LC-MS/MS peptide-confirmation workflows, which use MSIght's separate pre-processing pipelines rather than the MSI spectral pre-processing module
- Input is not an imzML-backed MSI pixel matrix (e.g., peak-picked peak lists or LC-MS chromatograms), where the pixel-wise spectrum assumptions of this step do not hold

## Inputs

- Raw MALDI-MSI imzML file (spatial pixel-by-m/z intensity matrix)
- imzML-backed intensity data structure produced by the MSIght common-functions loader

## Outputs

- Baseline-corrected spectra written as preprocessed_msi_data.imzML in the designated output directory
- TIC-normalized, Gaussian-smoothed (sigma=2), tophat-corrected results object passed onward for dimensionality reduction or segmentation

## How to apply

Within the MSIght MSI pre-processing workflow, first load the raw imzML file with the pyimzML-backed common-functions loader to build the intensity matrix of spatial pixels, then apply TIC normalization so intensities are comparable across the tissue section. Smooth each spectrum with an iterative Gaussian filter using sigma = 2 to suppress pixel-level noise prior to baseline handling. Subtract the baseline with a white tophat filter (SciPy morphology operations) using a structuring element of size 10, which estimates the broad, low-frequency chemical background and removes it from each spectrum while preserving narrow peptide peaks. Write the TIC-normalized, smoothed, baseline-corrected spectra back into the imzML-backed data structure in the designated output directory as preprocessed_msi_data.imzML. The structuring element size controls the width of background features that are treated as baseline: features wider than the element are removed, narrower ones (peaks) are retained.

## Related tools

- **MSIght** (Platform whose MSI pre-processing pipeline orchestrates TIC normalization, Gaussian smoothing (sigma=2), and white tophat baseline correction on imzML data) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language for the pre-processing module, chosen for accessibility and ecosystem access to SciPy morphology routines)
- **SciPy** (Provides the morphology operations used to execute the white tophat baseline subtraction with structuring element size 10)
- **pyimzML** (Backs loading of the raw imzML file and re-writing of the corrected spectra into the imzML-backed data structure)

## Evaluation signals

- The output file preprocessed_msi_data.imzML exists in the designated output directory and loads without error via the same pyimzML reader
- Each corrected spectrum shows a flat low-frequency baseline with narrow peaks retained, consistent with a size-10 structuring element
- Processing parameters (TIC normalization, Gaussian sigma=2, tophat structuring size 10) are recorded and reproducible from the pre-processing step
- The returned results object can be passed onward to the segmentation/clustering step (e.g., MSIght.refactor_segment.cluster_msi) without format errors
- Pixel dimensions and m/z axis of the output match the input imzML, confirming spectra were corrected in place rather than re-gridded

## Limitations

- The structuring element size is fixed at 10 in this workflow; other m/z ranges or peak widths may be over- or under-corrected without re-tuning
- Baseline subtraction is performed after TIC normalization and Gaussian smoothing (sigma=2), so errors introduced in those earlier steps propagate into the corrected spectra
- Requires imzML-backed continuous profile spectra from MALDI-MSI; it is not designed for centroided peak lists or non-imaging MS data

## Evidence
