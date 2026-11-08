---
name: ion-mobility-ms-lipid-dataset-harmonization
description: Use when lipid CCS values from one IM platform (e.g., negative-mode TIMS
  from Vasilopoulou et al. or Tsugawa et al.) must be calibrated toward reference
  values from another platform (DT) before pooled error metrics like MDRE are computed.
when_to_use_negative:
- CCS values are already measured on the same instrument/platform as the reference
  values, so no cross-instrument linear correction is needed
- The goal is de novo CCS prediction via machine learning (e.g., the LiPydomics trained
  CCS models) rather than post-acquisition calibration of measured values
- Input values are HILIC retention times rather than collision cross sections
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: LiPydomics
  role: Python package providing the experimental m/z/CCS lipid database and stats
    modules used to analyze ion mobility-MS lipidomics data within which the harmonized
    CCS values are handled
  repo: https://github.com/dylanhross/lipydomics
- name: pandas
  role: loads each constituent TIMS CCS dataset into a DataFrame, applies per-dataset
    linear corrections, concatenates corrected values, and writes corrected_ccs_table.csv
- name: SciPy
  role: numerical support for applying the linear correction equations and computing
    the median relative error statistic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:b60433b8eae407eb8558c1b89c6dd77dd4882a7df63990a2bbd51afed5ab02d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# ion-mobility-ms-lipid-dataset-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonizes lipid CCS measurements collected on different ion mobility platforms (e.g., TIMS vs. drift-tube/DT) by applying published linear correction equations per constituent dataset, then pooling corrected values and computing pooled error statistics such as the median relative error (MDRE).

## When to use

Use when lipid CCS values from one IM platform (e.g., negative-mode TIMS from Vasilopoulou et al. or Tsugawa et al.) must be calibrated toward reference values from another platform (DT) before pooled error metrics like MDRE are computed.

## When NOT to use

- CCS values are already measured on the same instrument/platform as the reference values, so no cross-instrument linear correction is needed
- The goal is de novo CCS prediction via machine learning (e.g., the LiPydomics trained CCS models) rather than post-acquisition calibration of measured values
- Input values are HILIC retention times rather than collision cross sections

## Inputs

- negative-mode TIMS CCS measurements from Vasilopoulou et al. (tabular/CSV)
- negative-mode TIMS CCS measurements from Tsugawa et al. (tabular/CSV)
- dataset-specific linear correction equations from Figure S2 of each source paper
- DT reference CCS values paired to each measurement

## Outputs

- corrected_ccs_table.csv containing a corrected CCS entry for every input measurement
- computed post-correction MDRE (target: −0.36%) recorded alongside the reported value

## How to apply

Load each constituent negative-mode TIMS CCS dataset (Vasilopoulou et al., Tsugawa et al.) into a separate pandas DataFrame. Select the dataset-specific linear correction equation reported in Figure S2 of each source paper and apply it to that dataset's CCS values so they are calibrated toward the DT reference values — coefficients are instrument- and dataset-specific and must not be pooled. Concatenate the corrected CCS values from both datasets into a single corrected CCS table, keeping the DT reference value alongside each corrected entry. Compute the post-correction median relative error (MDRE) of corrected CCS against DT references; for negative-mode TIMS the expected result is −0.36%. Write the table to corrected_ccs_table.csv with the computed MDRE recorded next to the reported value for comparison.

## Related tools

- **LiPydomics** (Python package providing the experimental m/z/CCS lipid database and stats modules used to analyze ion mobility-MS lipidomics data within which the harmonized CCS values are handled) — https://github.com/dylanhross/lipydomics
- **pandas** (loads each constituent TIMS CCS dataset into a DataFrame, applies per-dataset linear corrections, concatenates corrected values, and writes corrected_ccs_table.csv)
- **SciPy** (numerical support for applying the linear correction equations and computing the median relative error statistic)

## Examples

```
import pandas as pd; corr = pd.concat([pd.read_csv('vasilopoulou_neg.csv'), pd.read_csv('tsugawa_neg.csv')]); corr.to_csv('corrected_ccs_table.csv')
```

## Evaluation signals

- Computed post-correction MDRE reproduces the reported −0.36% within reasonable numerical tolerance
- Corrected CCS table contains a corrected CCS entry for every input measurement (no rows dropped during concatenation)
- Each corrected value is traceable to the specific Figure S2 linear correction equation of its source dataset, not a pooled or shared correction
- Corrected CCS values lie systematically closer to their paired DT reference values than the raw TIMS values (relative errors shrink toward zero)
- Output file corrected_ccs_table.csv parses cleanly and carries the computed MDRE alongside the reported −0.36% for diff comparison

## Limitations

- Linear corrections are instrument- and dataset-specific; applying one dataset's Figure S2 coefficients to another dataset's measurements will not reproduce the −0.36% MDRE
- MDRE is a median statistic, so large per-lipid outliers can be masked by the pooled value
- Experimental CCS collections 'do not yet comprehensively cover the vast lipid chemical space (both in terms of class and composition)', so harmonization is bounded by the coverage of the constituent datasets
- The LiPydomics database and prediction models are 'built to be highly extensible and customizable, allowing easy expansion as more data becomes available' — corrections may need re-fitting as reference data grows

## Evidence
