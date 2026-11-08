---
name: lcms-feature-filtering
description: Use when an MZmine-exported full-scan feature table (.csv with row ID,
  row m/z, row retention time, per-sample peak areas) must be reduced to biologically
  consistent ions before MS/MS targeted acquisition.
when_to_use_negative:
- The input has no dilution/concentration series — linear-decline correlation criteria
  cannot be computed from a single condition
- The goal is MS2-level processing or molecular networking (use MZmine MS/MS preprocessing,
  GNPS-FBMN, or ChemWalker steps instead)
- Data was not preprocessed by MZmine with row ID/m/z/retention time and Peak area
  columns exported to CSV
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: RegFilter
  role: Python package performing the linear-regression feature filtering and scheduled
    precursor list construction on the MZmine-exported table
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Upstream preprocessing (mass detection, ADAP chromatogram building, deconvolution,
    isotope grouping, join aligner, peak finder) producing the CSV feature table consumed
    by RegFilter
- name: MSConvert
  role: Converts raw MS files to .mzXML before MZmine preprocessing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:c7c25900a9c251ddf957ecc064bc19a45e743db0aa25648ed7fb3e9cff4444c6
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# lcms-feature-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Linear-regression-based filtering of an MZmine-preprocessed LC-MS feature table (RegFilter): retains precursor ions whose MS1 peak areas decrease linearly (R2 >= 0.9 by default) across a serial dilution series, to build scheduled precursor lists (SPL) that recover biological signals missed in DDA mode.

## When to use

Use when an MZmine-exported full-scan feature table (.csv with row ID, row m/z, row retention time, per-sample peak areas) must be reduced to biologically consistent ions before MS/MS targeted acquisition.

## When NOT to use

- The input has no dilution/concentration series — linear-decline correlation criteria cannot be computed from a single condition
- The goal is MS2-level processing or molecular networking (use MZmine MS/MS preprocessing, GNPS-FBMN, or ChemWalker steps instead)
- Data was not preprocessed by MZmine with row ID/m/z/retention time and Peak area columns exported to CSV

## Inputs

- MZmine 2.53-exported full-scan feature table (.csv) containing row ID, row m/z, row retention time, and per-sample Peak area columns
- MS1 peak areas mapped to a five-point dilution/concentration series (1.0, 0.5, 0.25, 0.125, 0.0625 mg/mL)

## Outputs

- Report stating the number of total features and the number of filtered features
- New .csv table of filtered features with their respective retention times, formatted for conversion into a scheduled precursor list (SPL)

## How to apply

Load the MZmine 2.53-exported CSV (with row ID, row m/z, row retention time and Peak area columns) and map peak-area columns onto the five-point dilution series (1.0, 0.5, 0.25, 0.125 and 0.0625 mg/mL). For each feature row, fit a linear regression of MS1 peak-area intensity against concentration and retain only features whose area decreases linearly with a coefficient of determination R2 >= 0.9 (user-adjustable default), controlling the number of points with the npts= parameter. Discard features whose intensity response fails the criteria as noise, then export a report stating the number of total and filtered features, plus a new .csv of filtered features with retention times formatted for conversion into a scheduled precursor list (SPL) for different mass spectrometers. The rationale is to rationally move from untargeted to targeted acquisition: RegFilter covers the entire chromatographic run and selects ions for fragmentation that DDA stochastically missed, including low-relative-abundance and low-m/z ions.

## Related tools

- **RegFilter** (Python package performing the linear-regression feature filtering and scheduled precursor list construction on the MZmine-exported table) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Upstream preprocessing (mass detection, ADAP chromatogram building, deconvolution, isotope grouping, join aligner, peak finder) producing the CSV feature table consumed by RegFilter)
- **MSConvert** (Converts raw MS files to .mzXML before MZmine preprocessing)

## Examples

```
conda env create -f regfilter_env.yml && conda activate regfilter && pip install git+https://github.com/computational-chemical-biology/regression_filter.git
```

## Evaluation signals

- Running the module on the input table produces a report containing both the total-feature count and the filtered-feature count
- The exported filtered .csv has rows that all carry m/z and retention time values, mirroring building_spl_and_assessing_XIC_for_all_ions.ipynb behavior
- Every retained feature's MS1 peak area decreases linearly across the dilution series with R2 >= the chosen threshold (default 0.9)
- Filtered-feature counts are plausible relative to the full-scan feature table (article reference: 135 features for BRA006, 219 for BRA010, 150 for BRA177)

## Limitations

- R2 cutoff (default 0.9) and npts are user-set; inappropriate values will over- or under-filter features
- Requires a serially diluted sample series; cannot be applied to single-condition acquisitions
- Feature selection depends on upstream MZmine parameters (e.g., m/z tolerance 0.001 m/z or 15 ppm, join aligner RT tolerance 0.2); poor preprocessing propagates into filtering

## Evidence

- [other] RegFilter describes the filtering conditions: ions across the dilution series whose area decreases linearly with a coefficient of determination (R2) >= 0.9: [evidence span withheld — anchor status: not_found]
- [methods] Report and SPL construction: [evidence span withheld — anchor status: not_found]
