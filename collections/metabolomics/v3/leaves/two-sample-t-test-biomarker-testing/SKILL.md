---
name: two-sample-t-test-biomarker-testing
description: Use when you have a sample x metabolite relative concentration matrix
  produced by NMR quantification (NMRQNet csv output) and need to determine which
  metabolites separate two sample groups, e.g. MDS patients vs. healthy family controls.
when_to_use_negative:
- When the input is still raw 1H NMR spectra rather than a NMRQNet-estimated concentration
  matrix — use NMRQNet identification/quantification first.
- When you need to pre-filter metabolites with false-positive NMRQNet predictions
  before statistics — the article notes this requires a final manual visualization
  check rather than an automated post-processing step.
- When comparing more than two sample groups or when the goal is unsupervised clustering
  rather than patient-vs-control biomarker testing.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NMRQNet
  role: Produces the metabolite relative concentration matrix that this statistical
    biomarker-testing skill consumes; EDTA plasma analysis tutorial is provided at
    NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb.
  repo: https://github.com/LiuzLab/NMRQNet
- name: rDolphin
  role: Alternative NMR quantification method compared against NMRQNet in the article;
    mentioned as a method requiring manual library adjustment, not used for the biomarker
    statistics described here.
  repo: https://github.com/danielcanueto/rDolphin
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2023.03.01.530642
    title: 'NMRQNet: a deep learning approach for automatic identification and quantification
      of metabolites using Nuclear Magnetic Resonance (NMR) in human plasma samples'
  source_content_hash: sha256:73d8d8bd88874baf38f92ce58837683e052212aa0cb3a6be8f1f80b3b2b70b0d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/LiuzLab/NMRQNet
  repo_url: https://github.com/LiuzLab/NMRQNet
  tools:
  - NMRQNet
  - rDolphin
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# two-sample-t-test-biomarker-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical biomarker extraction skill that runs PCA outlier screening, supervised OPLS-DA group separation, and per-metabolite two-sample t-tests on the NMRQNet-estimated relative concentration matrix (e.g. 40 Mecp2 duplication syndrome family plasma samples x 38 metabolites) to identify metabolites that differ significantly between patient and control groups, such as acetate with t-test p = 0.016.

## When to use

Use when you have a sample x metabolite relative concentration matrix produced by NMR quantification (NMRQNet csv output) and need to determine which metabolites separate two sample groups, e.g. MDS patients vs. healthy family controls.

## When NOT to use

- When the input is still raw 1H NMR spectra rather than a NMRQNet-estimated concentration matrix — use NMRQNet identification/quantification first.
- When you need to pre-filter metabolites with false-positive NMRQNet predictions before statistics — the article notes this requires a final manual visualization check rather than an automated post-processing step.
- When comparing more than two sample groups or when the goal is unsupervised clustering rather than patient-vs-control biomarker testing.

## Inputs

- NMRQNet csv output: chemical-shift column plus one column of 1H NMR spectra per plasma sample (e.g. NMRQNet/data/tutorial_data/control_human_plasma.csv)
- Sample x metabolite relative concentration matrix from the EDTA plasma analysis pipeline (40 MDS family samples x 38 metabolites)
- Binary patient-vs-control labels for each sample (e.g. MDS patient vs. healthy family member)

## Outputs

- Per-metabolite t-statistics and p-values written to a csv such as mds_metabolite_ttest_results.csv
- Identification of significantly differing metabolites (e.g. acetate, p = 0.016)
- Fig. 6c-style boxplot of the significant metabolite's concentrations by group
- PCA outlier-screening plot and OPLS-DA group separation assessment

## How to apply

Load the NMRQNet-estimated relative concentration matrix from the csv output of the EDTA plasma analysis pipeline (e.g. 40 MDS family plasma samples x 38 metabolites). First apply PCA on the concentration matrix to screen for outlier samples before making group comparisons. Then fit a supervised OPLS-DA model on the patient vs. control labels to assess group separation across the 38 metabolites. Finally run two-sample t-tests on each metabolite's relative concentration between patient and control groups, and rank metabolites by t-statistic and p-value; in the paper this identified acetate as significantly elevated in the MDS patient group with p = 0.016. Write all metabolite t-statistics and p-values to a results csv (e.g. mds_metabolite_ttest_results.csv) and accompany significant hits with a Fig. 6c-style boxplot of the acetate concentrations.

## Related tools

- **NMRQNet** (Produces the metabolite relative concentration matrix that this statistical biomarker-testing skill consumes; EDTA plasma analysis tutorial is provided at NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb.) — https://github.com/LiuzLab/NMRQNet
- **rDolphin** (Alternative NMR quantification method compared against NMRQNet in the article; mentioned as a method requiring manual library adjustment, not used for the biomarker statistics described here.) — https://github.com/danielcanueto/rDolphin

## Examples

```
python NMRQNet/tutorial/tutorial_for_EDTA_plasma.ipynb --input plasma_concentrations.csv --labels patient_control.csv --output mds_metabolite_ttest_results.csv
```

## Evaluation signals

- The t-test p-value for acetate between MDS patients and healthy family controls reproduces to approximately 0.016.
- PCA on the concentration matrix flags the same outlier samples before group comparisons are run.
- OPLS-DA on patient vs. control labels shows group separation consistent with the paper's Fig. 6 style of supervised projection.
- The output csv contains one t-statistic and one p-value per metabolite across all 38 tested metabolites.
- Significant metabolites are visually confirmed with a Fig. 6c-style boxplot, mirroring the article's presentation.

## Limitations

- Further experiments were needed to confirm the potential biomarkers; the statistical result alone is not confirmation.
- NMRQNet regularized less on false-positive predictions, which requires a final visualization check to filter metabolites with false-positive predictions before trusting downstream statistics.
- Only 40 MDS family plasma samples were tested, so per-metabolite t-tests may be underpowered for small effect sizes.

## Evidence
