---
name: mzmine-peak-list-parsing
description: Use when the input is an MZmine 2 aligned peak list (from 'More Restrictive'
  or 'Less Restrictive' extraction) that must be decomposed into candidate peak groups
  with sample-level intensity profiles and RT window boundaries before filtering or
  classification.
when_to_use_negative:
- Input is already a filtered feature table of peak-group summary parameters with
  no per-sample intensity shapes — use the peak-group parameter logistic regression
  filtering path instead
- Peak lists were produced by a tool other than MZmine 2 (e.g., XCMS centWave output
  without MZmine-style aligned peak groups and grouped peak windows), since the parsing
  logic assumes MZmine 2 peak-list structure
- The goal is spectral identification or annotation of metabolites rather than peak
  quality classification of extracted LC-MS features
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Generates the non-filtered aligned peak lists (v2.14, 'More Restrictive' and
    'Less Restrictive' extraction parameters) that this skill parses into candidate
    peak groups
- name: R
  role: Custom R scripts implement the peak-list parsing and the extraction of per-group
    windows and sample selections feeding the raster builder
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:e95a98f7567294d1c3e4cc1753bdd6d78415668f3b81c77826c5c10d99dc1866
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mzmine-peak-list-parsing

## Summary

Load MZmine 2-aligned peak lists extracted from non-targeted HILIC LC-MS plasma data and extract per-peak-group m/z and retention-time windows, selecting the 63 samples in which each candidate peak was most intense as the basis for downstream image-raster construction and true/false peak classification.

## When to use

Use when the input is an MZmine 2 aligned peak list (from 'More Restrictive' or 'Less Restrictive' extraction) that must be decomposed into candidate peak groups with sample-level intensity profiles and RT window boundaries before filtering or classification.

## When NOT to use

- Input is already a filtered feature table of peak-group summary parameters with no per-sample intensity shapes — use the peak-group parameter logistic regression filtering path instead
- Peak lists were produced by a tool other than MZmine 2 (e.g., XCMS centWave output without MZmine-style aligned peak groups and grouped peak windows), since the parsing logic assumes MZmine 2 peak-list structure
- The goal is spectral identification or annotation of metabolites rather than peak quality classification of extracted LC-MS features

## Inputs

- MZmine 2 (v2.14) aligned peak list from non-targeted HILIC LC-MS plasma metabolomics runs (Cohort 1 [N=78] or Cohort 2 [N=526])
- Grouped peak windows (m/z and retention-time windows surrounding each candidate feature apex)
- Raw LC-MS datasets downloadable from https://doi.org/doi:10.25345/C5FD2F

## Outputs

- Per-peak-group records containing per-sample intensity profiles from the 63 most intense samples
- Per-peak-group m/z and retention-time window boundaries (row-64 window-boundary coding source)
- Peak groups labeled true/false, ready for rasterization into 64x64 images and serialization (e.g., raster_images.npz)

## How to apply

Load the MZmine 2 (v2.14) aligned peak list generated after raw LC-MS data extraction under either 'More Restrictive' or 'Less Restrictive' peak extraction parameters, and pair it with the grouped peak windows defined for each candidate feature. For each aligned peak group, extract the m/z and retention-time windows surrounding the group apex, then select peak information from the 63 samples in which the candidate peak was most intense — this subsampling yields the 63 sample rows that later become rows 1–63 of the 64x64 raster, with row 64 encoding the retention-time window boundary (1 inside window, 0 outside). Parse each group's true/false peak label alongside its intensity profile so downstream scripts can attach labels during serialization. The rationale is that conventional centWave/local-minimum-search peak picking alone leaves a large false-positive burden (~50% under restrictive, ~75% under less restrictive settings), so the parsed peak groups must retain per-sample shape information for ML-based classification rather than only summary peak parameters.

## Related tools

- **MZmine 2** (Generates the non-filtered aligned peak lists (v2.14, 'More Restrictive' and 'Less Restrictive' extraction parameters) that this skill parses into candidate peak groups)
- **R** (Custom R scripts implement the peak-list parsing and the extraction of per-group windows and sample selections feeding the raster builder)

## Examples

```
python EDML_deep_learning2.py
```

## Evaluation signals

- Each parsed peak group yields exactly 63 selected samples (the samples in which the candidate peak was most intense), matching the 63 intensity rows of the downstream 64x64 raster
- Every peak group carries an m/z and retention-time window whose boundary can be encoded as the row-64 binary mask (1 inside, 0 outside)
- Peak-list composition matches the paper's expectations: ~50% false positives under More Restrictive settings (11,026 features total) and ~75% false positives under Less Restrictive settings on Cohort 2
- Every parsed group has an attached true/false label so the serialized raster set can be used for supervised training (e.g., 1304 training / 740 calibration / 726 test manually reviewed examples from Cohort 1)

## Limitations

- Parsing depends on MZmine 2-specific peak-list structure and grouped peak windows; the authors note implementation is 'using commonly used MZmine 2 workflows and custom R scripts'
- The skill only prepares peak groups for filtering; it does not itself remove false peaks — ML filters (DNN, logistic regression, Random Forest) operate on the parsed output
- No versioned neural-net implementation details are given beyond the script name EDML_deep_learning2.py, so reproducibility relies on the manuscript's datasets at https://doi.org/doi:10.25345/C5FD2F

## Evidence

- [results] Non-filtered peak lists were then generated using MZmine 2 where the raw data was extracted using both "More Restrictive" and "Less Restrictive" peak extraction parameters: [evidence span withheld — anchor status: not_found]
