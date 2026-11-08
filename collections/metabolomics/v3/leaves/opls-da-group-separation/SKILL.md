---
name: opls-da-group-separation
description: Use when you have a metabolite concentration matrix (e.g., 40 plasma
  samples x 38 NMRQNet-estimated metabolites) with binary group labels and need to
  screen outliers, test group separation, and identify which metabolites differ between
  groups.
when_to_use_negative:
- Input is still raw 1H NMR spectra (chemical shift columns + per-sample spectra)
  — quantify first with the NMRQNet EDTA plasma pipeline before running group separation
- Metabolite predictions have not had the required manual visualization check to filter
  false positives, since NMRQNet regularizes too little on false-positive predictions
- The comparison involves more than two groups or non-plasma biofluids, for which
  no NMRQNet analysis pipeline is provided
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: NMRQNet
  role: Produces the estimated relative concentration matrix of 38 metabolites from
    human plasma 1H NMR spectra that is the input to the PCA/OPLS-DA/t-test workflow
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:09c8ace3743e62b2986dd69da9482f1cec36602000de24af20e3f0141e49c6d7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - NMRQNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# opls-da-group-separation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Supervised group-separation analysis of NMRQNet-estimated metabolite relative concentrations, combining PCA outlier screening, OPLS-DA projection on case/control labels, and per-metabolite two-sample t-tests to nominate biomarkers (e.g., acetate in MDS plasma).

## When to use

Use when you have a metabolite concentration matrix (e.g., 40 plasma samples x 38 NMRQNet-estimated metabolites) with binary group labels and need to screen outliers, test group separation, and identify which metabolites differ between groups.

## When NOT to use

- Input is still raw 1H NMR spectra (chemical shift columns + per-sample spectra) — quantify first with the NMRQNet EDTA plasma pipeline before running group separation
- Metabolite predictions have not had the required manual visualization check to filter false positives, since NMRQNet regularizes too little on false-positive predictions
- The comparison involves more than two groups or non-plasma biofluids, for which no NMRQNet analysis pipeline is provided

## Inputs

- NMRQNet-estimated relative concentration matrix as csv (e.g., 40 MDS family plasma samples x 38 metabolites)
- Binary sample labels (MDS patient vs. healthy family control) aligned to the concentration matrix rows

## Outputs

- PCA screening result flagging outlier samples prior to group comparisons
- OPLS-DA model assessing patient vs. control group separation across metabolites
- mds_metabolite_ttest_results.csv containing t-statistics and p-values for all 38 metabolites
- Fig. 6c-style boxplot of acetate relative concentrations by group

## How to apply

Load the NMRQNet-estimated relative concentration matrix from the csv output of the EDTA plasma analysis pipeline (40 MDS family plasma samples x 38 metabolites). First apply PCA on the concentration matrix to screen for outlier samples before any supervised comparison. Then fit an OPLS-DA model (supervised projection) using the patient vs. healthy family member labels to assess whether metabolite profiles separate the groups. Run two-sample t-tests on each metabolite's relative concentration between the patient and control groups; in the MDS application this verifies acetate is significantly elevated in patients with p=0.016. Write all metabolite t-statistics and p-values to mds_metabolite_ttest_results.csv, accompanied by a Fig. 6c-style boxplot of acetate concentrations.

## Related tools

- **NMRQNet** (Produces the estimated relative concentration matrix of 38 metabolites from human plasma 1H NMR spectra that is the input to the PCA/OPLS-DA/t-test workflow) — github.com/LiuzLab/NMRQNet

## Examples

```
import pandas as pd, scipy.stats as st; X = pd.read_csv('nmrqnet_relative_concentrations.csv', index_col=0); t, p = st.ttest_ind(X.loc[y=='MDS', 'acetate'], X.loc[y=='control', 'acetate'])
```

## Evaluation signals

- The input matrix has the expected shape of 40 MDS family plasma samples x 38 metabolite relative concentrations
- PCA is applied before supervised modeling and outliers are excluded/flagged prior to OPLS-DA and t-tests
- Acetate relative concentration comes out significantly elevated in the MDS patient group with t-test p-value 0.016
- The output csv mds_metabolite_ttest_results.csv contains a t-statistic and p-value for each of the 38 metabolites
- A Fig. 6c-style boxplot of acetate concentrations by patient/control group is produced

## Limitations

- NMRQNet regularizes too little on false-positive predictions, so a final manual visualization check is required to filter metabolites with false-positive predictions before trusting group statistics
- Potential biomarkers identified by statistical analysis are not yet confirmed; further experiments were needed
- Statistical analysis currently applies to the EDTA plasma pipeline; biofluid-specific analysis pipelines for other sample types are not yet developed

## Evidence
