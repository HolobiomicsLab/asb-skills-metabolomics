---
name: imzml-spectral-data-loading
description: Use when raw MALDI-MSI imaging data must be imported from an imzML file
  into a spatial-pixel intensity matrix as the entry point of an MSIght pre-processing
  pipeline before dimensionality reduction or segmentation.
when_to_use_negative:
- Input is not imzML-formatted MALDI-MSI imaging data (e.g., already-preprocessed
  spectra, peak tables, or LC-MS/MS files handled by other MSIght modules)
- The goal is downstream segmentation/clustering (cluster_msi) or histology co-registration
  rather than the initial import step
- Working outside Python with tools like Cardinal (R) or MSiReader, which provide
  their own imzML import paths
tools:
- name: MSIght
  role: Provides the common-functions module with load_and_preprocess_imzml for pyimzML-backed
    import of raw imzML imaging data
  repo: github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language of the distributable MSIght package and its Jupyter
    notebook application of the pipeline
  repo: github.com/laurenfields/MSIght
- name: Jupyter
  role: Notebook application that explains the analysis pipeline (starting from data
    loading) step-by-step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:c8accbc1e9cf02c5653c129b037e268b9cd5950a268d54c44d6f3966d6be9edb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Jupyter
  - MSIght
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# imzml-spectral-data-loading

## Summary

Loads raw MALDI-MSI imzML files via MSIght's common-functions module (load_and_preprocess_imzml, backed by pyimzML) to build the pixel-by-m/z intensity matrix that feeds downstream TIC normalization, Gaussian smoothing (sigma=2), and tophat baseline correction.

## When to use

Use when raw MALDI-MSI imaging data must be imported from an imzML file into a spatial-pixel intensity matrix as the entry point of an MSIght pre-processing pipeline before dimensionality reduction or segmentation.

## When NOT to use

- Input is not imzML-formatted MALDI-MSI imaging data (e.g., already-preprocessed spectra, peak tables, or LC-MS/MS files handled by other MSIght modules)
- The goal is downstream segmentation/clustering (cluster_msi) or histology co-registration rather than the initial import step
- Working outside Python with tools like Cardinal (R) or MSiReader, which provide their own imzML import paths

## Inputs

- Raw MALDI-MSI imaging data in imzML format (with associated .ibd binary file)

## Outputs

- Pixel-by-m/z intensity matrix of spatial pixels held in an imzML-backed MSIght results object
- imzML-backed data structure writable to a designated output directory (e.g., preprocessed_msi_data.imzML)

## How to apply

Install the distributable Python package (pip install msight) or clone the LingjunLiResearch/MSIght repository, then use the MSIght common-functions module's load_and_preprocess_imzml function, which is backed by pyimzML, to read the raw imzML file. The reader parses the imzML file (with its companion .ibd binary) and assembles the intensities of all spatial pixels into an intensity matrix suitable for the subsequent MSI pre-processing steps: TIC normalization per pixel, iterative Gaussian filtering with sigma=2, and white tophat baseline subtraction with a structuring element of size 10 (SciPy morphology). The loaded results object is designed to be passed onward for further analysis or visualization within the modular MSIght platform, which integrates MSI with H&E histology and LC-MS/MS confirmation.

## Related tools

- **MSIght** (Provides the common-functions module with load_and_preprocess_imzml for pyimzML-backed import of raw imzML imaging data) — github.com/lingjunli-research/MSIght
- **Python** (Implementation language of the distributable MSIght package and its Jupyter notebook application of the pipeline) — github.com/laurenfields/MSIght
- **Jupyter** (Notebook application that explains the analysis pipeline (starting from data loading) step-by-step)

## Examples

```
pip install msight
```

## Evaluation signals

- The pyimzML-backed reader opens the raw imzML file (and its .ibd companion) without import errors and returns an intensity matrix spanning all spatial pixels
- The returned results object is an imzML-backed structure that can be passed onward to downstream MSIght steps (TIC normalization, smoothing, baseline correction, segmentation)
- After downstream pre-processing, the output file (e.g., preprocessed_msi_data.imzML) exists in imzML-backed format and reloads without error via the same pyimzML reader
- A quick round-trip following the README pattern — load data, apply an analysis pipeline, visualize the results — completes without failure

## Limitations

- Requires imzML input (backed by pyimzML); other vendor or open formats are not handled by this loading step
- Loading is only the entry point: spectra remain un-normalized and uncorrected until TIC normalization, Gaussian smoothing (sigma=2), and tophat baseline subtraction (structuring element size 10) are applied downstream
- The article notes many available tools require users to be experts at both MSI and image analysis; MSIght mitigates this via a Jupyter notebook, but the loading step still assumes Python use

## Evidence

- [readme] You can install MSIght directly from PyPI: pip install msight: [evidence span withheld — anchor status: external_doc]
