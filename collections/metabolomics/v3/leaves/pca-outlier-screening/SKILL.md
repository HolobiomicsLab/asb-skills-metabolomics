---
name: pca-outlier-screening
description: Use when you have a matrix of NMR-estimated metabolite relative concentrations
  across plasma samples (e.g. NMRQNet CSV output) and need to detect outlier samples
  prior to supervised group-difference modeling (OPLS-DA, t-tests).
when_to_use_negative:
- Input is raw 1D NMR spectra (chemical shift × sample matrix) rather than estimated
  metabolite relative concentrations — run NMRQNet preprocessing/quantification first
- The metabolite concentration table contains NMRQNet false-positive metabolite predictions
  that have not been filtered by the final manual visualization check
- The goal is a purely supervised model of group separation without any need to screen
  for sample-driven outliers first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0593
tools:
- name: NMRQNet
  role: Produces the metabolite relative-concentration matrix that serves as input
    for the PCA outlier screen
  repo: github.com/LiuzLab/NMRQNet
- name: EDTA plasma analysis (NMRQNet tutorial)
  role: Reference pipeline that generates the CSV concentration output used for PCA
    screening in the MDS-family plasma workflow
  repo: github.com/LiuzLab/NMRQNet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:b401360b74be3c4947b70037ee3f57b401f24b1d30e81aeac2428b3c24e5b370
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pca-outlier-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies PCA to the NMRQNet-estimated metabolite relative-concentration matrix (e.g. 40 MDS-family plasma samples × 38 metabolites) to screen for outlier samples before supervised group comparisons such as OPLS-DA and per-metabolite t-tests.

## When to use

Use when you have a matrix of NMR-estimated metabolite relative concentrations across plasma samples (e.g. NMRQNet CSV output) and need to detect outlier samples prior to supervised group-difference modeling (OPLS-DA, t-tests).

## When NOT to use

- Input is raw 1D NMR spectra (chemical shift × sample matrix) rather than estimated metabolite relative concentrations — run NMRQNet preprocessing/quantification first
- The metabolite concentration table contains NMRQNet false-positive metabolite predictions that have not been filtered by the final manual visualization check
- The goal is a purely supervised model of group separation without any need to screen for sample-driven outliers first

## Inputs

- CSV matrix of NMRQNet-estimated metabolite relative concentrations (samples × metabolites), e.g. the CSV output of the NMRQNet EDTA plasma analysis tutorial
- Optional binary patient-vs-control group labels used for the downstream OPLS-DA / t-test step

## Outputs

- PCA scores/loadings used to identify outlier plasma samples
- List or table of flagged outlier samples to exclude or re-examine before OPLS-DA and t-tests
- Downstream statistics enabled by clean data, e.g. per-metabolite t-statistics and p-values (mds_metabolite_ttest_results.csv)

## How to apply

Load the NMRQNet-estimated relative-concentration matrix (rows = plasma samples, e.g. 40 MDS family samples; columns = metabolites, e.g. 38) produced by the EDTA plasma analysis pipeline's CSV output. Run PCA on this concentration matrix and inspect the score plot to flag samples that fall far from the main cluster, screening them before any supervised modeling. Once outliers are handled, proceed to supervised comparisons (OPLS-DA on patient vs. control labels and two-sample t-tests per metabolite); in the MDS analysis this chain confirmed acetate as significantly elevated with a t-test p-value of 0.016. Rationale: PCA is an unsupervised projection, so it reveals sample-driven structure (including outliers) without using group labels, preventing outliers from distorting OPLS-DA separation or inflating false t-test significance. Note that NMRQNet itself can produce false-positive metabolite predictions, so concentrations feeding the PCA should be sanity-checked against spectra/visualizations before treating flagged samples as biological outliers.

## Related tools

- **NMRQNet** (Produces the metabolite relative-concentration matrix that serves as input for the PCA outlier screen) — github.com/LiuzLab/NMRQNet
- **EDTA plasma analysis (NMRQNet tutorial)** (Reference pipeline that generates the CSV concentration output used for PCA screening in the MDS-family plasma workflow) — github.com/LiuzLab/NMRQNet

## Examples

```
python -c "import pandas as pd; from sklearn.decomposition import PCA; X = pd.read_csv('NMRQNet_output.csv', index_col=0); scores = PCA(n_components=2).fit_transform(X)"
```

## Evaluation signals

- PCA score plot shows most plasma samples clustered together with flagged outliers visually separable from the main cluster
- PCA is run on the full NMRQNet concentration matrix (e.g. 40 samples × 38 metabolites) with no group labels supplied, confirming it is unsupervised
- Downstream OPLS-DA group separation and per-metabolite t-tests are computed on the screened data; the MDS benchmark expects acetate to surface as significant (t-test p = 0.016)
- Number of samples entering OPLS-DA / t-tests equals the original count minus PCA-flagged outliers, and every exclusion is traceable to the PCA scores

## Limitations

- NMRQNet regularizes insufficiently on false-positive metabolite predictions, so the final visualization check to filter false positives must precede or accompany PCA — a false-positive metabolite can drive an apparent outlier
- PCA flags outliers relative to the full concentration matrix; outliers caused by NMR quantification error cannot be distinguished from true biological outliers without spectral visualization
- Biofluid-specific pipelines beyond plasma (e.g. other biofluids) are not yet developed, so this screening pattern is currently validated for plasma (EDTA/heparin/citrate) workflows

## Evidence
