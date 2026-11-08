---
name: per-feature-linear-regression-modeling
description: Use when a full-scan feature table was acquired across a serial dilution
  series (e.g. 1.0, 0.5, 0.25, 0.125, 0.0625 mg/mL) and you must select biologically
  consistent ions by their linear intensity-vs-concentration response.
when_to_use_negative:
- The input data were not acquired as a serial dilution/concentration series, so no
  intensity-vs-concentration regression can be fit
- The feature table lacks per-sample Peak area columns or row m/z / row retention
  time fields from MZmine export
- The goal is general untargeted feature detection rather than building a targeted
  scheduled precursor list for DDA-complementing fragmentation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: RegFilter
  role: Custom Python package whose filtering algorithm implements the per-feature
    linear regression of peak area vs concentration and exports the filtered table
    and report
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Upstream preprocessing of full-scan LC-MS data (mass detection, ADAP, deconvolution,
    join aligner, peak finder, CSV export) that produces the feature table RegFilter
    consumes
- name: MSConvert
  role: Converts raw MS files to .mzXML format before MZmine preprocessing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:7802f2c69123e6aa4f0b9c533606a8526b5fbb8f6ba34da8c8c4371ada767e1a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/computational-chemical-biology/regression_filter
  repo_url: https://github.com/computational-chemical-biology/regression_filter
  tools:
  - MSConvert
  - MZmine
  - MZmine 2.53
  - RegFilter
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# per-feature-linear-regression-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a linear regression of MS1 peak-area intensity against concentration for each feature row of an MZmine-preprocessed dilution-series table, retaining features whose response decreases linearly with R2 >= 0.9. This statistical criterion is the core of RegFilter's selection of precursor ions for a scheduled precursor list.

## When to use

Use when a full-scan feature table was acquired across a serial dilution series (e.g. 1.0, 0.5, 0.25, 0.125, 0.0625 mg/mL) and you must select biologically consistent ions by their linear intensity-vs-concentration response.

## When NOT to use

- The input data were not acquired as a serial dilution/concentration series, so no intensity-vs-concentration regression can be fit
- The feature table lacks per-sample Peak area columns or row m/z / row retention time fields from MZmine export
- The goal is general untargeted feature detection rather than building a targeted scheduled precursor list for DDA-complementing fragmentation

## Inputs

- MZmine-exported full-scan feature table (.csv) with row ID, row m/z, row retention time, and per-sample Peak area columns
- Dilution-series concentration mapping (1.0, 0.5, 0.25, 0.125, 0.0625 mg/mL) for the peak-area columns
- User parameters: npts= (number of points to filter) and the R2 threshold (default 0.9, adjustable)

## Outputs

- Report stating the number of total features and the number of filtered features
- New .csv table of filtered features with row m/z and retention times, formatted for conversion into a scheduled precursor list (SPL)

## How to apply

Load the MZmine-exported .csv feature table containing row ID, row m/z, row retention time, and per-sample Peak area columns, and map the peak-area columns onto the five-point dilution series (1.0, 0.5, 0.25, 0.125, and 0.0625 mg/mL). For each feature row, fit a linear regression of MS1 peak-area intensity against concentration and retain only features whose area decreases linearly with a coefficient of determination R2 >= 0.9 (a default value adjustable by the user); the number of points used is controlled by the user-set npts= parameter in the notebook. Discard rows whose intensity response fails the correlation/consistency criteria as noise. Export a report stating the number of total features and the number of filtered features, plus a new .csv of the filtered features with their retention times, formatted so it can be converted into a scheduled precursor list (SPL) for different mass spectrometers. The rationale is that ions tracking the dilution factor linearly are genuine sample-derived biological signals rather than noise, and their selection rationally moves the workflow from untargeted to targeted fragmentation.

## Related tools

- **RegFilter** (Custom Python package whose filtering algorithm implements the per-feature linear regression of peak area vs concentration and exports the filtered table and report) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Upstream preprocessing of full-scan LC-MS data (mass detection, ADAP, deconvolution, join aligner, peak finder, CSV export) that produces the feature table RegFilter consumes)
- **MSConvert** (Converts raw MS files to .mzXML format before MZmine preprocessing)

## Examples

```
conda activate regfilter && pip install git+https://github.com/computational-chemical-biology/regression_filter.git && jupyter notebook notebooks/building_spl_and_assessing_XIC_for_all_ions.ipynb
```

## Evaluation signals

- Running the module on the input table produces a report with both the total-feature count and the filtered-feature count
- Every row of the exported filtered .csv carries m/z and retention time values, mirroring the behavior of building_spl_and_assessing_XIC_for_all_ions.ipynb
- Each retained feature meets the R2 >= 0.9 linear-decrease criterion across the npts= dilution points
- The filtered feature count is a strict subset of the total features in the input table (paper case: 135 features for BRA006, 219 for BRA010, 150 for BRA177 out of a larger total)
- The output .csv is formatted so it can be converted into a scheduled precursor list (SPL) for the target mass spectrometer

## Limitations

- The default R2 threshold (0.9) and npts are user-adjustable, so results depend on chosen parameters
- The regression filter requires a proper serial dilution series; it cannot be applied to data without replicate concentration levels
- The DDA comparison shows the SPL selects ions stochastically missed by DDA, but 76, 135, and 77 features (BRA006, BRA010, BRA177) were common between SPL and DDA fragments, so the filtered list is not exhaustive

## Evidence

- [methods] Regression-filter selection criterion: [evidence span withheld — anchor status: not_found]
