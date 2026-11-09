---
name: roi-detection-parameter-curation
description: Decision support for setting per-method AriumMS ROI search and false-positive
  filter parameters before processing multi-platform CE-MS/HILIC-MS datasets, e.g.
  when peaks differ in width between separation methods and DOE-significant parameters
  must be carried forward.
when_to_use_negative:
- Input is already a processed feature table or peak-integrated matrix — ROI search
  parameters no longer apply at that stage.
- Single-platform analysis where existing single-platform tools (XCMS Online, MetaboAnalyst,
  MetaboAnalystR, MZmine) are sufficient, since those are not designed for augmentation
  of different separation techniques but work fine per platform.
- Tasks requiring MS/MS spectral networking between features of different methods
  — AriumMS does not implement this; it is listed as future development.
edam_topics:
- http://edamontology.org/topic_3337
tools:
- name: AriumMS
  role: Target toolbox whose ROI search, preprocessing, and augmentation steps consume
    the curated per-method parameters; supports flexible parameterization for individual
    separation methods
  repo: https://github.com/AdrianHaun/AriumMS
- name: msconvert
  role: ProteoWizard converter used to prepare .mzXML or .mzML inputs that the curated
    ROI parameters will be applied to
  repo: http://proteowizard.sourceforge.net/download.html
- name: zmat
  role: ZMat compression toolbox bundled as a dependency of the AriumMS MATLAB app
    (MATLAB R2024a or newer with Bioinformatic, Statistics and Machine Learning, Wavelet,
    Image Processing, Signal Processing, Parallel Computing, and Database toolboxes)
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:6218bcb02932e68f517864cdfb3168f3ac04dea04a756f3669215c2e15a5f872
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
  tools:
  - AriumMS
  - msconvert
  - zmat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3337
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# roi-detection-parameter-curation

## Summary

Curate and record the AriumMS non-target ROI-detection parameters for each of the five CE-MS and HILIC-MS analytical methods, prioritizing the four parameters the D-optimal DOE flagged as significant (ROI intensity threshold, mass spectra alignment, m/z error, feature occurrence filter) into method-keyed structured records.

## When to use

Decision support for setting per-method AriumMS ROI search and false-positive filter parameters before processing multi-platform CE-MS/HILIC-MS datasets, e.g. when peaks differ in width between separation methods and DOE-significant parameters must be carried forward.

## When NOT to use

- Input is already a processed feature table or peak-integrated matrix — ROI search parameters no longer apply at that stage.
- Single-platform analysis where existing single-platform tools (XCMS Online, MetaboAnalyst, MetaboAnalystR, MZmine) are sufficient, since those are not designed for augmentation of different separation techniques but work fine per platform.
- Tasks requiring MS/MS spectral networking between features of different methods — AriumMS does not implement this; it is listed as future development.

## Inputs

- Supplementary file1 (DOCX) documenting AriumMS non-target processing settings (Supplementary file1, DOCX 1.51 MB)
- Per-method definitions for three CE-Orbitrap MS and two HILIC-Orbitrap MS methods
- MS data convertible to .mzXML or .mzML (via msconvert) that the parameters will apply to

## Outputs

- Method-keyed JSON record written to processing_parameters, one entry per method with all six parameter fields (ROI intensity threshold, m/z error, mass spectra alignment quantile, minimum relative feature occurrence, minimum relative peak height, minimum ROI size)

## How to apply

Extract the ROI search and false-positive filter settings per method from the supplementary parameter documentation (Supplementary file1, DOCX): ROI intensity threshold at 5-10% of the lowest base peak chromatogram/electropherogram intensity, m/z error of 0.01 Da, the mass spectra alignment quantile, a minimum relative feature occurrence of 50%, and a minimum relative peak height of 25%. Treat ROI intensity threshold, mass spectra alignment, m/z error, and feature occurrence filter as the D-optimal DOE-significant parameters carried forward for optimization, so their values deserve per-method tuning rather than defaults. Set the minimum ROI size per method: 15 for the HILIC methods and a value below 10 for the CE methods, because CE-MS peaks are narrower. Structure all values as method-keyed JSON entries with explicit field names (one entry per separation method), writing the record to processing_parameters and validating that every method carries all six parameter fields. Rationale: AriumMS uses the set parameters to run ROI search, preprocessing, and data augmentation via the ROIpeaks/MSroiaug functions, and it supports flexible parameterization for individual separation methods with different peak characteristics.

## Related tools

- **AriumMS** (Target toolbox whose ROI search, preprocessing, and augmentation steps consume the curated per-method parameters; supports flexible parameterization for individual separation methods) — https://github.com/AdrianHaun/AriumMS
- **msconvert** (ProteoWizard converter used to prepare .mzXML or .mzML inputs that the curated ROI parameters will be applied to) — http://proteowizard.sourceforge.net/download.html
- **zmat** (ZMat compression toolbox bundled as a dependency of the AriumMS MATLAB app (MATLAB R2024a or newer with Bioinformatic, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes)) — http://github.com/fangq/zmat

## Examples

```
processing_parameters = {'HILIC_pos': {'roi_intensity_threshold': 0.05, 'mz_error': 0.01, 'min_roi_size': 15, 'min_feature_occurrence': 0.50, 'min_peak_height': 0.25}, 'CE_cation': {'min_roi_size': 8}}
```

## Evaluation signals

- Every method entry in processing_parameters carries all six parameter fields — validation should pass for all five methods (three CE-MS, two HILIC-MS).
- ROI intensity threshold values fall in the 5-10% range of the lowest base peak chromatogram/electropherogram intensity.
- m/z error is 0.01 Da for each method entry.
- Minimum relative feature occurrence is 50% and minimum relative peak height is 25% in every method entry.
- Minimum ROI size is 15 for HILIC methods and below 10 for CE methods, reflecting the narrower CE peaks.

## Limitations

- Parameter values are method-specific: no single default works across platforms because CE-MS peaks are narrow and require a smaller minimum ROI size than HILIC methods.
- No AriumMS version number is stated in the article; reproducibility relies on the repository and the paper's DOI reference.
- The supplementary material is referenced (Supplementary file1, DOCX 1.51 MB) but no direct access URL is given in the article section.
- MS/MS spectral networking for connecting features across methods is not implemented and is left as future development.

## Evidence

- [readme] Set parameters are then used to perform ROI search, data preprocessing and data augmentation.: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation."
- [supplementary] D-optimal DOE significant parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Per-method flexible parameterization: "including parallel processing of datasets and flexible parameterization for processing of individual separation methods with different peak characteristics"
- [readme] Input file format requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [discussion] AriumMS processing pipeline stages: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation"
