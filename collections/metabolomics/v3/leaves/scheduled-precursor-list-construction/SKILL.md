---
name: scheduled-precursor-list-construction
description: Use when a user has an MZmine-exported full-scan feature table (.csv
  with row ID, row m/z, row retention time, per-sample Peak area) from a dilution
  series and wants to build a targeted precursor list for scheduled fragmentation
  of ions missed in DDA mode.
when_to_use_negative:
- The input is already an MZmine-preprocessed feature table with no serial dilution
  structure — the linear regression against concentration requires a dilution series
  and will not produce meaningful R2/slope criteria without it.
- The goal is a general DDA/untargeted molecular networking run with no need to re-target
  specific precursor ions — the SPL is a targeted-acquisition construct, not a replacement
  for the standard untargeted workflow.
- A DDA run already fragmented the features of interest — RegFilter's value is capturing
  ions 'previously missed in DDA mode', not re-listing ions already confidently fragmented.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: RegFilter
  role: Custom Python package that performs the linear-regression-based filtering
    and exports the filtered .csv feature table used to build the SPL
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Preprocessing tool that produces the full-scan feature table (row ID, row
    m/z, row retention time, Peak area) that RegFilter consumes; parameters include
    Join aligner m/z tolerance 0.001 m/z or 15 ppm and RT tolerance 0.2
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:49859f4c8cb1051fcb05432fa3eb6cfdecdfb59f96cf0471a9903f2c482c6d54
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
  - MZmine
  - MZmine 2.53
  - RegFilter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scheduled-precursor-list-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a scheduled precursor list (SPL) for MS/MS acquisition from serially diluted LC-MS/MS feature tables by retaining ions whose peak area decreases linearly with concentration (R2 ≥ 0.9), then exporting the filtered features' m/z and retention times in a format convertible to an SPL for different mass spectrometers.

## When to use

Use when a user has an MZmine-exported full-scan feature table (.csv with row ID, row m/z, row retention time, per-sample Peak area) from a dilution series and wants to build a targeted precursor list for scheduled fragmentation of ions missed in DDA mode.

## When NOT to use

- The input is already an MZmine-preprocessed feature table with no serial dilution structure — the linear regression against concentration requires a dilution series and will not produce meaningful R2/slope criteria without it.
- The goal is a general DDA/untargeted molecular networking run with no need to re-target specific precursor ions — the SPL is a targeted-acquisition construct, not a replacement for the standard untargeted workflow.
- A DDA run already fragmented the features of interest — RegFilter's value is capturing ions 'previously missed in DDA mode', not re-listing ions already confidently fragmented.

## Inputs

- MZmine-exported full-scan feature table (.csv) with columns: row ID, row m/z, row retention time, and per-sample Peak area values
- Five-point dilution/concentration series metadata mapping sample columns to concentrations: 1.0, 0.5, 0.25, 0.125, and 0.0625 mg/mL
- User-defined npts= parameter controlling the number of points used for the regression filtering

## Outputs

- Report stating the number of total features and the number of filtered features
- New .csv file containing only the filtered features with their respective retention times (m/z and RT), formatted for conversion into a scheduled precursor list (SPL) for different mass spectrometers

## How to apply

Load the MZmine-preprocessed full-scan feature table (.csv) and map the Peak area columns onto the five-point dilution series (1.0, 0.5, 0.25, 0.125, and 0.0625 mg/mL). For each feature row, fit a linear regression of MS1 peak-area intensity against concentration and retain only features with a coefficient of determination (R2) ≥ 0.9 and a negative slope (intensity decreasing with dilution), with the number of points to filter set by the user via the npts= parameter in the repository's building_spl_and_assessing_XIC_for_all_ions.ipynb notebook. The rationale is to separate true biological signals from noise, since ions across a dilution series whose area decreases linearly are consistent biological signals rather than random noise. Discard features whose intensity response does not satisfy the filtering requirements. Export a report showing the number of total features and the number of filtered features, then export a new .csv containing only the filtered features with their respective retention times, formatted so it can be converted into a scheduled precursor list (SPL) for different mass spectrometers.

## Related tools

- **RegFilter** (Custom Python package that performs the linear-regression-based filtering and exports the filtered .csv feature table used to build the SPL) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Preprocessing tool that produces the full-scan feature table (row ID, row m/z, row retention time, Peak area) that RegFilter consumes; parameters include Join aligner m/z tolerance 0.001 m/z or 15 ppm and RT tolerance 0.2)

## Examples

```
python -m jupyter nbconvert --to notebook --execute notebooks/building_spl_and_assessing_XIC_for_all_ions.ipynb  # set csv path and npts= in the notebook
```

## Evaluation signals

- The exported .csv contains only features meeting the linear-regression filtering criteria (R2 ≥ 0.9, negative slope across the dilution series).
- The report lists both the number of total features and the number of filtered features (e.g. 135 filtered of the total for BRA006, 219 for BRA010, 150 for BRA177).
- The filtered .csv rows all carry m/z and retention time values, mirroring the columns of the MZmine feature table.
- The exported file is placed in the notebooks directory of the installed package, as the notebook writes it there.
- Biological-signal recovery percentages from the dilution series can be compared against the article's reported 9.3%, 9.9%, and 9.0% for BRA006, BRA010, and BRA177 respectively.

## Limitations

- The filtering criteria require a serial dilution series; without concentration-mapped peak areas the linear regression is not meaningful.
- The default R2 threshold of 0.9 is adjustable by the user but the number of points (npts=) must be set manually, introducing user-dependent variability.
- The approach is designed for LC-MS/MS feature tables preprocessed by MZmine; other preprocessing outputs may not expose the row ID / row m/z / row retention time / Peak area schema the notebook expects.
- The article notes that 'the proper automation of the process, as well as the storage of the resulting fragmentation spectra remain challenges'.

## Evidence

- [results] RegFilter describes the filtering conditions: ions across the dilution series whose area decreases linearly with a coefficient of determination (R2) ≥ 0.9 (a default value adjustable by the user) are selected.: [evidence span withheld — anchor status: not_found]
