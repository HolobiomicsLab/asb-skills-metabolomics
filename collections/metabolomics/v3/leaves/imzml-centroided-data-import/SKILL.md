---
name: imzml-centroided-data-import
description: Use when the input consists of centroided .imzML (or AnnData .h5ad /
  MuData .h5mu) MSI files that must be imported for lipid imaging analysis, and pixel-level
  peak alignment, recalibration, and ion-image extraction are needed before quantitation.
when_to_use_negative:
- Input is already a processed feature table or concentration matrix rather than raw
  pixel-level imzML/h5ad/h5mu imaging data
- The data are profile-mode rather than centroided spectra — LipidQMap's workflow
  assumes centroided input for its dynamic binning
- The goal is generic MSI exploration (clustering, segmentation) without lipid annotation;
  broader tools such as Cardinal or SCiLS Lab may be more appropriate
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidQMap
  role: GUI application that opens centroided .imzML, .h5ad, and .h5mu files, auto-detects
    ion mode, performs binning, imputation, and lock-mass recalibration during import
  repo: https://github.com/swinnenteam/LipidQMap
- name: alexandrovteam/pyimzML
  role: Python parser for the imzML 1.1.0 format returning data as python lists and
    dicts; usable for programmatic imzML access
  repo: https://github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:609248bcdb876df0b3ebe617d8afd81fe63bd7d76b871efdcfcfd6722a0fecb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/swinnenteam/LipidQMap
  tools:
  - LipidQMap
  - alexandrovteam/pyimzML
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# imzml-centroided-data-import

## Summary

Import vendor-neutral centroided .imzML mass spectrometry imaging files into LipidQMap (or parse them with pyimzML), handling automatic ion-mode detection, mass-error and binning settings, optional 3x3 missing-pixel imputation, and optional lock-mass recalibration so downstream lipid annotation and quantitation operate on aligned peak data.

## When to use

Use when the input consists of centroided .imzML (or AnnData .h5ad / MuData .h5mu) MSI files that must be imported for lipid imaging analysis, and pixel-level peak alignment, recalibration, and ion-image extraction are needed before quantitation.

## When NOT to use

- Input is already a processed feature table or concentration matrix rather than raw pixel-level imzML/h5ad/h5mu imaging data
- The data are profile-mode rather than centroided spectra — LipidQMap's workflow assumes centroided input for its dynamic binning
- The goal is generic MSI exploration (clustering, segmentation) without lipid annotation; broader tools such as Cardinal or SCiLS Lab may be more appropriate

## Inputs

- centroided .imzML files (vendor-neutral MSI format), multiple files loadable simultaneously
- AnnData .h5ad files with var['mz'], var['mz_mode'], obsm['spatial'], optional obs['sample_id'], obs['foreground'], uns['spot_size'], layers['raw']
- MuData .h5mu files containing an MSI modality with the same feature metadata and an X matrix or layers['raw']

## Outputs

- aligned, binned centroided spectra across all pixels (constant ppm resolution)
- ion images for lipid database entries per sample, with positive/negative ion mode labels
- optionally recalibrated (lock-mass corrected) and imputed (3x3 mean) datasets ready for isotope correction and quantitation

## How to apply

Open one or more centroided .imzML files via LipidQMap's imzML import dialog; ion mode (positive/negative) is detected automatically per file and mixed-mode files can be loaded together. Set the maximum tolerated mass error in ppm for ion-image extraction and the bin size for the average spectrum (default 5 mDa for TOF instruments; decrease for higher-resolution instruments) — LipidQMap's dynamic binning aligns centroided peaks across all pixels while maintaining constant ppm resolution across the mass range. Optionally enable imputation of missing pixels by averaging the surrounding 3x3 neighborhood, and enable online (lock-mass) calibration with a reference m/z, a tolerance in ppm (default 30 ppm), and a minimum reference intensity (default 10 000); avoid low-abundance references such as <8 000 a.u. on timsTOF instruments to prevent incorrect signal assignment. AnnData .h5ad and MuData .h5mu inputs are supported as alternatives, requiring var['mz'], var['mz_mode'], and obsm['spatial'] (plus optional obs['sample_id'], obs['foreground'], uns['spot_size'], and layers['raw']).

## Related tools

- **LipidQMap** (GUI application that opens centroided .imzML, .h5ad, and .h5mu files, auto-detects ion mode, performs binning, imputation, and lock-mass recalibration during import) — https://github.com/swinnenteam/LipidQMap
- **alexandrovteam/pyimzML** (Python parser for the imzML 1.1.0 format returning data as python lists and dicts; usable for programmatic imzML access) — https://github.com/alexandrovteam/pyimzML

## Examples

```
LipidQMap: File > Open > imzML import dialog > Open Files (centroided .imzML) > set max mass error (ppm), bin size 5 mDa, 3x3 imputation, lock-mass 30 ppm / min intensity 10000 > Import Data
```

## Evaluation signals

- Ion mode is correctly auto-detected for each file and positive/negative files can be loaded in the same session
- Ion images are extracted within the user-set ppm mass-error tolerance after dynamic binning with constant ppm across the mass range
- Lock-mass calibration only engages when the reference m/z peak meets the tolerance and minimum intensity criteria (defaults 30 ppm / 10 000); low-abundance references (<8 000 a.u. on timsTOF) are not used
- Missing pixels are imputed as the mean of the surrounding 3x3 neighborhood when that option is selected
- A 5 GB imzML file with 2 500 ion images imports and quantifies in roughly 20 seconds on an M2 MacBook as a performance sanity check

## Limitations

- Low-abundance signals, e.g. intensities <8 000 a.u. on timsTOF instruments, should be avoided for recalibration as they may lead to incorrect signal assignment
- Recalibration parameters (tolerance, minimum intensity) may need adjustment for datasets acquired on different mass spectrometer platforms
- Bin size should be decreased below the 5 mDA TOF default for higher-resolution instruments
- Standalone LipidQMap builds are provided only for Windows (x64) and macOS (Apple Silicon); Linux users must run from source

## Evidence

- [readme] README imzML import dialog options: [evidence span withheld — anchor status: external_doc]
- [readme] README bin size guidance: [evidence span withheld — anchor status: external_doc]
