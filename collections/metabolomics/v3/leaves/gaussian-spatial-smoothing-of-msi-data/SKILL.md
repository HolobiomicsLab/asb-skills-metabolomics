---
name: gaussian-spatial-smoothing-of-msi-data
description: Use when raw MALDI-MSI imzML pixel spectra must be denoised prior to
  dimensionality reduction/segmentation, and pixel-level noise in the spectral image
  data needs suppression via Gaussian smoothing as part of the MSIght pre-processing
  pipeline.
when_to_use_negative:
- Input is already pre-processed (TIC-normalized, smoothed, baseline-corrected) MSIght
  output such as preprocessed_msi_data.imzML — re-smoothing would compound the Gaussian
  filter
- The analysis goal is MSIght's other modalities (H&E image pre-processing, linear/non-linear
  registration, or LC-MS/MS peptide confirmation) rather than MSI spectral pre-processing
- Only LC-MS/MS or histology data are available and no imzML MSI dataset exists to
  build a pixel intensity matrix from
edam_operation: http://edamontology.org/operation_3627
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MSIght
  role: Hosts the MSI pre-processing workflow (Figure S1) that performs the sigma=2
    Gaussian smoothing step between TIC normalization and tophat baseline correction;
    installable via pip
  repo: github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language of the distributable MSIght package used to execute
    the pre-processing pipeline
- name: pyimzML
  role: Backend reader that loads the raw imzML file into the intensity matrix of
    spatial pixels consumed by the smoothing step
- name: SciPy
  role: Provides morphology operations used in the adjacent pre-processing steps of
    the pipeline (white tophat baseline subtraction with structuring element size
    10)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:aa6bcd4698d0fa0582c1dd6c72024ae6b75508558316b3ad9770a8921ada602e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MSIght
  - Python
  - SciPy
  - pyimzML
  edam_operation: http://edamontology.org/operation_3627
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gaussian-spatial-smoothing-of-msi-data

## Summary

MSIght's MSI image pre-processing step that applies an iterative Gaussian filter (sigma = 2) to smooth spectral image data from imported MALDI-MSI imzML files, suppressing pixel-level noise between TIC normalization and white tophat baseline correction before downstream dimensionality reduction and segmentation.

## When to use

Use when raw MALDI-MSI imzML pixel spectra must be denoised prior to dimensionality reduction/segmentation, and pixel-level noise in the spectral image data needs suppression via Gaussian smoothing as part of the MSIght pre-processing pipeline.

## When NOT to use

- Input is already pre-processed (TIC-normalized, smoothed, baseline-corrected) MSIght output such as preprocessed_msi_data.imzML — re-smoothing would compound the Gaussian filter
- The analysis goal is MSIght's other modalities (H&E image pre-processing, linear/non-linear registration, or LC-MS/MS peptide confirmation) rather than MSI spectral pre-processing
- Only LC-MS/MS or histology data are available and no imzML MSI dataset exists to build a pixel intensity matrix from

## Inputs

- Raw MALDI-MSI imzML file imported via pyimzML-backed load_and_preprocess_imzml
- TIC (total ion current)-normalized per-pixel intensity matrix of spatial pixels

## Outputs

- Gaussian-smoothed (sigma = 2) spectral image data held in the imzML-backed data structure
- preprocessed_msi_data.imzML written to the designated output directory, passing onward to further analysis or visualization

## How to apply

First load the raw imzML file with the MSIght common-functions module (load_and_preprocess_imzml, backed by pyimzML) to build the intensity matrix of spatial pixels, then apply TIC (total ion current) normalization per pixel so intensities are comparable across the tissue section. With the TIC-normalized spectral image data in hand, apply an iterative Gaussian filter with sigma = 2 to smooth the data and suppress pixel-level noise; this step sits between TIC normalization and white tophat baseline subtraction (structuring element size 10 via SciPy morphology), so order matters. The smoothed spectra are written back into the imzML-backed data structure and results are saved into the designated output directory. Rationale: smoothing at sigma = 2 reduces per-pixel noise so that the subsequent baseline-corrected, TIC-normalized spectra feed cleaner inputs into MSIght's segmentation (cluster_msi) and visualization stages.

## Related tools

- **MSIght** (Hosts the MSI pre-processing workflow (Figure S1) that performs the sigma=2 Gaussian smoothing step between TIC normalization and tophat baseline correction; installable via pip) — github.com/lingjunli-research/MSIght
- **Python** (Implementation language of the distributable MSIght package used to execute the pre-processing pipeline)
- **pyimzML** (Backend reader that loads the raw imzML file into the intensity matrix of spatial pixels consumed by the smoothing step)
- **SciPy** (Provides morphology operations used in the adjacent pre-processing steps of the pipeline (white tophat baseline subtraction with structuring element size 10))

## Examples

```
pip install msight; from MSIght.common_functions import load_and_preprocess_imzml; results = load_and_preprocess_imzml('raw_data.imzml')
```

## Evaluation signals

- Output file preprocessed_msi_data.imzML exists in imzML-backed format in the designated output directory
- The output imzML loads without error via the same pyimzML reader that ingested the raw file
- The returned results object can be passed onward for further analysis or visualization without format errors
- Per-spectrum pixel-level noise is visibly suppressed relative to the TIC-normalized input spectra (sigma = 2 Gaussian applied exactly once)
- Downstream steps of the MSIght pipeline (e.g., cluster_msi segmentation) accept the smoothed spectra as input

## Limitations

- The Gaussian sigma = 2 and tophat structuring element size 10 are the pipeline's fixed pre-processing parameters; no alternative smoothing settings are described in the source material
- MSIght is an early-stage release (version 0.0.1, initial release 9/17/2024), so behavior around pre-processing parameters may still evolve
- The smoothing operates on spectra after TIC normalization, so it is not a substitute for intensity normalization across the tissue section

## Evidence

- [other] Gaussian smoothing parameters in MSIght pre-processing: [evidence span withheld — anchor status: not_found]
- [supplementary] MSI pre-processing workflow figure: "Figure S1: MSI pre-processing workflow"
- [readme] MSIght as the integrating platform: "MSIght is a program produced by the Lingjun Li Lab at the University of Wisconsin-Madison for automated integration of H&E, MSI, and LC-MS/MS datasets"
- [methods] Repository source availability: "git clone https://github.com/lingjunli-research/MSIght.git"
