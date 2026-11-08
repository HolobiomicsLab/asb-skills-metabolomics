---
name: tic-normalization-of-msi-spectra
description: Use when raw imzML MALDI-MSI pixel spectra must be made quantitatively
  comparable across a tissue section before dimensionality reduction or segmentation,
  as the first steps of MSIght's MSI pre-processing workflow.
when_to_use_negative:
- Input is not imzML MALDI-MSI pixel data (e.g. it is LC-MS/MS peptide identification
  output, which MSIght handles in a separate LC-MS/MS workflow)
- Spectra have already been TIC-normalized and baseline-corrected, or the downstream
  goal is H&E histology image pre-processing (MSIght's separate H&E workflow)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3621
tools:
- name: MSIght
  role: Python platform whose common-functions module implements imzML loading, TIC
    normalization, Gaussian smoothing, and tophat baseline correction in the MSI pre-processing
    workflow
  repo: https://github.com/LingjunLiResearch/MSIght
- name: pyimzML
  role: Backend reader/writer used by MSIght to load the raw imzML intensity matrix
    and write the preprocessed spectra back into imzML format
- name: SciPy
  role: Morphology operations used for the white tophat baseline subtraction with
    structuring element size 10
- name: Python
  role: Language in which MSIght's pre-processing and normalization method is written
    and distributed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:3676a2e46f0cfb558377b6dce6b08437fea63a37c06975a7e2645aad15bd45bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LingjunLiResearch/MSIght
  tools:
  - MSIght
  - Python
  - SciPy
  - pyimzML
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# tic-normalization-of-msi-spectra

## Summary

Total ion current (TIC) normalization of MALDI-MSI pixel spectra, performed inside MSIght's imzML pre-processing pipeline together with Gaussian smoothing (sigma = 2) and white tophat baseline correction (structuring element size 10) before downstream dimensionality reduction and segmentation.

## When to use

Use when raw imzML MALDI-MSI pixel spectra must be made quantitatively comparable across a tissue section before dimensionality reduction or segmentation, as the first steps of MSIght's MSI pre-processing workflow.

## When NOT to use

- Input is not imzML MALDI-MSI pixel data (e.g. it is LC-MS/MS peptide identification output, which MSIght handles in a separate LC-MS/MS workflow)
- Spectra have already been TIC-normalized and baseline-corrected, or the downstream goal is H&E histology image pre-processing (MSIght's separate H&E workflow)

## Inputs

- Raw MALDI-MSI imzML file containing spatial pixel spectra
- Intensity matrix of spatial pixels built via pyimzML-backed loading

## Outputs

- preprocessed_msi_data.imzML containing TIC-normalized, Gaussian-smoothed (sigma=2), tophat-baseline-corrected spectra
- Results object (imzML-backed) that can be passed onward for clustering, analysis, or visualization

## How to apply

Load the raw imzML file through MSIght's common-functions module (load_and_preprocess_imzml, backed by pyimzML) to build the intensity matrix of spatial pixels. Apply TIC (total ion current) normalization to each pixel's spectrum so per-pixel intensities are summed to a common scale and comparable across the tissue section. Then apply an iterative Gaussian filter with sigma = 2 to smooth the spectral image data and suppress pixel-level noise, and perform white tophat baseline subtraction with a structuring element of size 10 using SciPy's morphology operations to remove broad chemical background from each spectrum. Write the TIC-normalized, smoothed, baseline-corrected spectra back into the imzML-backed data structure into the designated output directory, keeping the data passable onward to MSIght's clustering pipeline (cluster_msi).

## Related tools

- **MSIght** (Python platform whose common-functions module implements imzML loading, TIC normalization, Gaussian smoothing, and tophat baseline correction in the MSI pre-processing workflow) — https://github.com/LingjunLiResearch/MSIght
- **pyimzML** (Backend reader/writer used by MSIght to load the raw imzML intensity matrix and write the preprocessed spectra back into imzML format)
- **SciPy** (Morphology operations used for the white tophat baseline subtraction with structuring element size 10)
- **Python** (Language in which MSIght's pre-processing and normalization method is written and distributed)

## Examples

```
from MSIght.common_functions import load_and_preprocess_imzml; spectra = load_and_preprocess_imzml('raw_data.imzML', output_dir='out/')
```

## Evaluation signals

- Output file preprocessed_msi_data.imzML exists in imzML-backed format in the designated output directory
- The output imzML loads without error via the same pyimzML reader used for the raw input
- Per-pixel spectra are TIC-normalized (intensities comparable across the tissue section, consistent per-pixel total ion current)
- Smoothing was applied with the Gaussian sigma = 2 parameter and baseline removal used a white tophat structuring element of size 10
- The returned results object can be passed onward to further analysis or visualization (e.g. MSIght's cluster_msi segmentation pipeline)

## Limitations

- The article itself does not state the smoothing or baseline parameters in the main text; sigma = 2 and tophat size 10 come from the supplementary MSI pre-processing workflow and surfaced task cards
- The source cards do not state how TIC normalization handles zero-intensity or empty pixels, which is a known edge case for MALDI-MSI TIC scaling
- MSIght's PyPI release notes list only the initial v0.0.1 release (9/17/2024), so module names like load_and_preprocess_imzml may change between versions

## Evidence

- [full_text] Gaussian smoothing and tophat baseline correction parameters: [evidence span withheld — anchor status: not_found]
