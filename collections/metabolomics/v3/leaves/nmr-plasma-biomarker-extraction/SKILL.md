---
name: nmr-plasma-biomarker-extraction
description: Use when a relative-concentration matrix (e.g. 40 MDS family plasma samples
  x 38 metabolites from NMRQNet) needs group-difference biomarker calls between patient
  and control groups via PCA, OPLS-DA, and t-tests.
when_to_use_negative:
- Input is raw NMR spectra rather than a post-NMRQNet relative concentration table
  — run the NMRQNet quantification pipeline first
- Plasma collected with heparin or citrate anticoagulants, or other biofluids, where
  the EDTA-plasma-specific tutorial/pipeline does not apply
- The goal is spectral identification/quantification quality benchmarking (RMSE, Pearson
  correlation) rather than downstream biomarker statistics
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0153
tools:
- name: NMRQNet
  role: Deep-learning pipeline that produces the relative concentration matrix of
    dominant metabolites (e.g. 38 metabolites) from plasma 1H NMR spectra, which is
    the input to the biomarker statistics
  repo: github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Tutorial workflow (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) whose
    csv concentration output is the direct input for PCA, OPLS-DA, and t-tests
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:ef6b61e39a9ea5468d7e56040074d1dd236be6f87a5335e7c624d92610d29360
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  tools:
  - EDTA plasma analysis (NMRQNet tutorial)
  - NMRQNet
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nmr-plasma-biomarker-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract metabolite biomarkers from NMR-quantified plasma concentrations by combining PCA outlier screening, OPLS-DA supervised projection, and per-metabolite two-sample t-tests, as demonstrated for acetate elevation in Mecp2 duplication syndrome (MDS) patients.

## When to use

Use when a relative-concentration matrix (e.g. 40 MDS family plasma samples x 38 metabolites from NMRQNet) needs group-difference biomarker calls between patient and control groups via PCA, OPLS-DA, and t-tests.

## When NOT to use

- Input is raw NMR spectra rather than a post-NMRQNet relative concentration table — run the NMRQNet quantification pipeline first
- Plasma collected with heparin or citrate anticoagulants, or other biofluids, where the EDTA-plasma-specific tutorial/pipeline does not apply
- The goal is spectral identification/quantification quality benchmarking (RMSE, Pearson correlation) rather than downstream biomarker statistics

## Inputs

- NMRQNet-estimated relative concentration matrix (csv output of the EDTA plasma analysis pipeline; e.g. 40 samples x 38 metabolites)
- Binary patient vs. control group labels (e.g. MDS patient vs. healthy family member)

## Outputs

- Per-metabolite two-sample t-test statistics and p-values written to mds_metabolite_ttest_results.csv
- Significant biomarker call (acetate elevated in MDS patients, t-test p-value 0.016)
- Fig. 6c-style boxplot of acetate concentrations between groups
- PCA outlier-screening result and OPLS-DA group-separation assessment

## How to apply

Load the NMRQNet-estimated relative concentration matrix (the csv output of the EDTA plasma analysis pipeline, e.g. 40 MDS family plasma samples x 38 metabolites) and first run PCA on the concentration matrix to screen for outlier samples before any group comparison. Fit an OPLS-DA model on the patient vs. control labels to assess multivariate group separation across metabolites. Then run two-sample t-tests on each metabolite's relative concentration between the patient and control groups; for the MDS reproduction this verifies acetate is significantly elevated in patients with p=0.016. Write all metabolite t-statistics and p-values to a results table (mds_metabolite_ttest_results.csv) and accompany the significant hit with a Fig. 6c-style boxplot of acetate concentrations. Remember NMRQNet regularizes less on false-positive metabolite predictions, so apply a final manual visualization check on flagged metabolites before trusting biomarker calls.

## Related tools

- **NMRQNet** (Deep-learning pipeline that produces the relative concentration matrix of dominant metabolites (e.g. 38 metabolites) from plasma 1H NMR spectra, which is the input to the biomarker statistics) — github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (Tutorial workflow (NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb) whose csv concentration output is the direct input for PCA, OPLS-DA, and t-tests) — github.com/LiuzLab/NMRQNet

## Examples

```
python run_mds_metabolite_ttest.py --input mds_relative_concentrations.csv --labels patient,control --out mds_metabolite_ttest_results.csv
```

## Evaluation signals

- t-test on acetate relative concentration between MDS patients and healthy family members reproduces p=0.016 with elevated concentrations in the patient group
- PCA on the 40x38 concentration matrix flags outlier samples before any group comparison, and OPLS-DA shows separation between patient and control labels
- Output csv contains t-statistics and p-values for all 38 metabolites (mds_metabolite_ttest_results.csv), not only significant hits
- Significant biomarker calls pass a final manual visualization (spectrum check) since NMRQNet regularizes insufficiently on false-positive predictions

## Limitations

- Potential biomarkers are not yet confirmed; further experiments are needed to confirm them
- NMRQNet regularizes insufficiently on false-positive predictions, so a final manual visualization check is required to filter false-positive metabolites
- No automated pre-filtering or post-processing step exists for false-positive predictions; biofluid-specific pipelines beyond plasma anticoagulant variants are not yet developed

## Evidence

- [results] acetate significantly elevated, p=0.016: [evidence span withheld — anchor status: not_found]
