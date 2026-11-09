---
name: machine-readable-parameter-serialization
description: Use when extracting processing parameter sets from supplementary tables
  or software documentation of a multi-platform metabolomics workflow (e.g., five
  CE-MS and HILIC-MS methods) and they must be persisted as structured, method-keyed
  records.
when_to_use_negative:
- Input is already a processed feature table or augmented data matrix — serialization
  of processing parameters only applies to the upstream ROI search / preprocessing
  stage
- Only a single separation platform is being processed and per-method parameterization
  is not needed — AriumMS's flexible per-method parameterization targets datasets
  with different peak characteristics across methods
- MS/MS spectral networking or augmentation of spectroscopic data is the goal — these
  are not implemented and listed as future developments
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: AriumMS
  role: Source of the per-method non-target processing parameters (ROI search, preprocessing,
    false-positive filter settings) that are serialized into the method-keyed JSON
    record
  repo: github.com/AdrianHaun/AriumMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:f62914a0cbac6de2f34622eb6e05a611e94b9cc44ecbe8ceba360dc62046c500
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# machine-readable-parameter-serialization

## Summary

Serializes per-method non-target processing parameters (e.g., AriumMS ROI intensity threshold, m/z error, mass spectra alignment quantile, feature occurrence filter) into method-keyed JSON records so that multi-platform CE-MS/HILIC-MS settings from supplementary documentation are machine-readable and reproducible.

## When to use

Use when extracting processing parameter sets from supplementary tables or software documentation of a multi-platform metabolomics workflow (e.g., five CE-MS and HILIC-MS methods) and they must be persisted as structured, method-keyed records.

## When NOT to use

- Input is already a processed feature table or augmented data matrix — serialization of processing parameters only applies to the upstream ROI search / preprocessing stage
- Only a single separation platform is being processed and per-method parameterization is not needed — AriumMS's flexible per-method parameterization targets datasets with different peak characteristics across methods
- MS/MS spectral networking or augmentation of spectroscopic data is the goal — these are not implemented and listed as future developments

## Inputs

- Supplementary file1 (DOCX 1.51 MB) with AriumMS non-target processing settings (Supplement Tables S1-S2)
- AriumMS parameter settings for three CE-Orbitrap MS and two HILIC-Orbitrap MS methods
- D-optimal DOE significance results identifying ROI intensity threshold, mass spectra alignment, m/z error, and feature occurrence filter

## Outputs

- processing_parameters JSON record with method-keyed entries, each carrying all six parameter fields (ROI intensity threshold, m/z error, mass spectra alignment quantile, minimum relative feature occurrence, minimum relative peak height, minimum ROI size)

## How to apply

Open the supplementary parameter documentation (Supplementary file1, DOCX 1.51 MB) describing AriumMS non-target processing and extract the ROI search and false-positive filter settings for each of the five methods: ROI intensity threshold (5–10% of the lowest base peak chromatogram/electropherogram intensity), m/z error of 0.01 Da, mass spectra alignment quantile, minimum relative feature occurrence of 50%, and minimum relative peak height of 25%. Record the per-method minimum ROI size, using 15 for the HILIC methods and a value below 10 for the CE methods, reflecting the narrower CE peaks. Structure all values as method-keyed JSON entries with explicit field names for each parameter, and write the structured record to processing_parameters. Rationale: AriumMS applies flexible parameterization for individual separation methods with different peak characteristics, so a single flat parameter set is insufficient; per-method keys preserve the platform-specific thresholds identified as significant by the D-optimal DOE (ROI intensity threshold, mass spectra alignment, m/z error, feature occurrence filter).

## Related tools

- **AriumMS** (Source of the per-method non-target processing parameters (ROI search, preprocessing, false-positive filter settings) that are serialized into the method-keyed JSON record) — github.com/AdrianHaun/AriumMS

## Examples

```
params = {m: {'roi_intensity_threshold_pct': 5, 'mz_error_da': 0.01, 'alignment_quantile': None, 'min_feature_occurrence': 0.5, 'min_peak_height': 0.25, 'min_roi_size': 15} for m in methods}; json.dump(params, open('processing_parameters.json', 'w'))
```

## Evaluation signals

- Every method key in processing_parameters carries all six parameter fields after validation
- m/z error equals 0.01 Da for all five methods
- Minimum ROI size is 15 for the HILIC methods and below 10 for the CE methods
- Minimum relative feature occurrence is 50% and minimum relative peak height is 25% for every method entry
- ROI intensity threshold falls in the documented 5–10% range of the lowest base peak chromatogram/electropherogram intensity

## Limitations

- Per-method thresholds are specific to the five CE-MS and HILIC-MS methods of this study; CE methods need smaller minimum ROI sizes due to narrower peaks
- The supplementary parameter file is referenced but no direct access link/URL is given in some article sections
- No version number is stated for AriumMS, complicating exact parameter provenance

## Evidence

- [supplementary] The D-optimal DOE identified four parameters as significant for further optimization in AriumMS non-target processing: ROI intensity threshold, mass spectra alignment, m/z error, and feature occurrence filter.: [evidence span withheld — anchor status: not_found]
- [abstract] parallel processing of datasets and flexible parameterization for processing of individual separation methods with different peak characteristics: "including parallel processing of datasets and flexible parameterization for processing of individual separation methods with different peak characteristics"
- [intro] AriumMS contains a universal and user-friendly toolbox, capable of handling multi-platform datasets.: "AriumMS contains a universal and user-friendly toolbox, capable of handling multi-platform datasets."
- [readme] Set parameters are then used to perform ROI search, data preprocessing and data augmentation.: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation."
