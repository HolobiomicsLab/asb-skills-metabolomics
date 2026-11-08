---
name: python-scientific-module-implementation
description: Use when a paper describes a custom semi-automated data-processing package
  (e.g. RegFilter selecting precursor ions by linear regression over a dilution series)
  that must be rebuilt as an installable Python module from its described workflow
  and README.
when_to_use_negative:
- Input is raw MS data (.mzXML) rather than an MZmine-preprocessed feature table —
  preprocessing with MSConvert/MZmine 2.53 must happen first
- The goal is DDA-style stochastic fragmentation rather than targeted selection of
  dilution-consistent ions for a scheduled precursor list
- Features lack a serial dilution / concentration series, so no linear regression
  of intensity vs. concentration can be fit
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: RegFilter
  role: Custom Python package being implemented; performs linear-regression-based
    filtering of MZmine feature tables and SPL construction
  repo: https://github.com/computational-chemical-biology/regression_filter
- name: MZmine 2.53
  role: Upstream preprocessing tool whose exported .csv feature tables are the module's
    required input
- name: MSConvert
  role: Converts raw MS files to .mzXML before MZmine preprocessing feeds the module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2024.08.07.607073
    title: Optimizing the detection of biological signals through a semi-automated
      feature selection tool
  source_content_hash: sha256:7691eb960a31f27a42d154a6f6808f0952b15dd55d324ca980362d78b20da439
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# python-scientific-module-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct a Python scientific package (RegFilter-style regression filter) as a reusable module with a conda environment, filtering notebook, and CSV report/export logic, mirroring the BSD-3-Clause regression_filter repository.

## When to use

Use when a paper describes a custom semi-automated data-processing package (e.g. RegFilter selecting precursor ions by linear regression over a dilution series) that must be rebuilt as an installable Python module from its described workflow and README.

## When NOT to use

- Input is raw MS data (.mzXML) rather than an MZmine-preprocessed feature table — preprocessing with MSConvert/MZmine 2.53 must happen first
- The goal is DDA-style stochastic fragmentation rather than targeted selection of dilution-consistent ions for a scheduled precursor list
- Features lack a serial dilution / concentration series, so no linear regression of intensity vs. concentration can be fit

## Inputs

- MZmine-exported full-scan feature table (.csv) with row ID, row m/z, row retention time, and per-sample Peak area columns
- Dilution-series mapping of peak-area columns to concentrations 1.0, 0.5, 0.25, 0.125, 0.0625 mg/mL

## Outputs

- Report stating the number of total features and the number of filtered features
- Filtered .csv of selected features with their m/z and retention times, formatted for scheduled precursor list (SPL) construction

## How to apply

Package the algorithm as an installable Python module (pip-installable from the repo, e.g. 'pip install git+https://github.com/computational-chemical-biology/regression_filter.git') inside a dedicated conda environment defined by an environment YAML (regfilter_env.yml, activated via 'conda activate regfilter'). The core module loads an MZmine-exported .csv feature table containing row ID, row m/z, row retention time, and per-sample Peak area columns, maps peak areas onto the five-point dilution series (1.0, 0.5, 0.25, 0.125, 0.0625 mg/mL), and fits a linear regression of MS1 peak-area intensity versus concentration per feature row. Retain features whose intensity decreases linearly with R2 >= 0.9 (user-adjustable default), with the number of points controlled by the user-set npts= parameter as exposed in the building_spl_and_assessing_XIC_for_all_ions.ipynb notebook. Export a report with the number of total features and filtered features, and a filtered .csv with m/z and retention times formatted for conversion into a scheduled precursor list (SPL).

## Related tools

- **RegFilter** (Custom Python package being implemented; performs linear-regression-based filtering of MZmine feature tables and SPL construction) — https://github.com/computational-chemical-biology/regression_filter
- **MZmine 2.53** (Upstream preprocessing tool whose exported .csv feature tables are the module's required input)
- **MSConvert** (Converts raw MS files to .mzXML before MZmine preprocessing feeds the module)

## Examples

```
conda env create -f regfilter_env.yml; conda activate regfilter; pip install git+https://github.com/computational-chemical-biology/regression_filter.git
```

## Evaluation signals

- Running the module on the input table produces a report containing both the total-feature count and the filtered-feature count
- The exported filtered .csv has rows that all carry m/z and retention time values, mirroring building_spl_and_assessing_XIC_for_all_ions.ipynb
- Every retained feature satisfies R2 >= 0.9 for linear decrease of peak area across the dilution series points selected by npts=
- Sanity check against published results: RegFilter selected 135 features for BRA006, 219 for BRA010, and 150 for BRA177 from full scan mode
- Filtered table is convertible into a scheduled precursor list for different mass spectrometers

## Limitations

- R2 >= 0.9 threshold is a default value adjustable by the user; results depend on this choice and on npts=
- Module requires an MZmine-preprocessed feature table with the dilution series mapped to specific peak-area columns; raw files must be converted with MSConvert and preprocessed (MZmine 2.53: ADAP min group size 4, join aligner m/z tolerance 0.001 m/z or 15 ppm) before use
- Only features whose intensity response decreases linearly with concentration are kept; non-monotonic biological signals are discarded as noise

## Evidence

- [methods] RegFilter describes the filtering conditions: [evidence span withheld — anchor status: not_found]
- [methods] Input format: [evidence span withheld — anchor status: not_found]
