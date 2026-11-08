---
name: vip-score-feature-selection
description: Use when a fitted PLS-DA or OPLS-DA model on metabolomics data (e.g.
  158 annotated lipids from VAT/SAT of 59 CRC patients) must be reduced to significant/discriminative
  features for downstream comparison of feature sets.
when_to_use_negative:
- Input is raw MS data that has not yet been pre-processed (noise reduction, peak
  matching, retention time correction) or log2-transformed and modeled — VIP selection
  requires a fitted PLS-DA/OPLS-DA model
- The analysis goal is unsupervised exploration (e.g. PCA of SAT vs VAT separation);
  VIP scores are defined only for supervised PLS-based classification
- Selecting features from held-out test data; the article selects significant features
  from training-set models and evaluates prediction on the corresponding test set
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: mixOmics
  role: Performs PLS-DA and supplies vip() to extract variable importance in projection
    scores for feature selection
- name: ropls
  role: Performs OPLS-DA and provides getVipVn for the OPLS-DA variant of VIP-based
    feature selection
- name: lmm2met
  role: Produces the LMM-processed data matrix whose PLS-DA VIP-selected features
    are compared against the M0 selection
  repo: https://github.com/kwanjeeraw/lmm2met
- name: caret
  role: Supports the repeated 4-fold cross-validation evaluation of the discriminative
    features selected at VIP > 1
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:0b11a221fed06632754a71913bcf70d1883dccadee65b718667d5c28a4bc63f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - caret
  - ggplot2
  - lmm2met
  - mixOmics
  - ropls
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# vip-score-feature-selection

## Summary

Selects discriminative metabolites from PLS-DA (or OPLS-DA) models by extracting variable importance in projection (VIP) scores and applying a VIP > 1 threshold on the first component, as used to compare LMM- vs M0-processed CRC adipose tissue lipidomics data.

## When to use

Use when a fitted PLS-DA or OPLS-DA model on metabolomics data (e.g. 158 annotated lipids from VAT/SAT of 59 CRC patients) must be reduced to significant/discriminative features for downstream comparison of feature sets.

## When NOT to use

- Input is raw MS data that has not yet been pre-processed (noise reduction, peak matching, retention time correction) or log2-transformed and modeled — VIP selection requires a fitted PLS-DA/OPLS-DA model
- The analysis goal is unsupervised exploration (e.g. PCA of SAT vs VAT separation); VIP scores are defined only for supervised PLS-based classification
- Selecting features from held-out test data; the article selects significant features from training-set models and evaluates prediction on the corresponding test set

## Inputs

- Fitted PLS-DA or OPLS-DA model object from mixOmics or ropls trained on a log2-transformed metabolite/lipid matrix (e.g. 158 annotated lipids from VAT and SAT of 59 colorectal carcinoma patients)
- Metabolite feature names/annotations column-aligned with the model matrix

## Outputs

- List of discriminative metabolites with VIP > 1 (e.g. 69 for LMM-PLS-DA, 66 for M0-PLS-DA)
- Per-metabolite VIP scores and ΔVIP values between processing strategies (e.g. cholesterol ΔVIP = 0.52)
- Venn-region counts of shared/unique features, optionally written to venn_intersection_union.csv

## How to apply

After fitting PLS-DA with mixOmics (or OPLS-DA with ropls) on the training data, extract VIP scores using mixOmics::vip or ropls::getVipVn and retain features with VIP > 1 on the first component. In the CRC adipose case study this yielded 69 discriminative metabolites for the LMM-processed data versus 66 for the M0 (unprocessed, log2-transformed) data, with 60 metabolites shared. Compare the selected sets between processing strategies via set intersection (60 shared), set difference (9 uniquely LMM, 6 uniquely M0), and union, verifying |union| = |LMM list| + |M0 list| − |intersection|. The rationale is that VIP > 1 flags features contributing above-average discriminatory power, and shifts in VIP scores (e.g. cholesterol ΔVIP = 0.52, SM (d40:2) ΔVIP = 0.43, Acylcarnitine C18:1 ΔVIP = 0.32) quantify the effect of the LMM processing step.

## Related tools

- **mixOmics** (Performs PLS-DA and supplies vip() to extract variable importance in projection scores for feature selection)
- **ropls** (Performs OPLS-DA and provides getVipVn for the OPLS-DA variant of VIP-based feature selection)
- **lmm2met** (Produces the LMM-processed data matrix whose PLS-DA VIP-selected features are compared against the M0 selection) — https://github.com/kwanjeeraw/lmm2met
- **caret** (Supports the repeated 4-fold cross-validation evaluation of the discriminative features selected at VIP > 1)

## Examples

```
vip_scores <- mixOmics::vip(plsda_model); sig_metabolites <- rownames(plsda_model$X)[vip_scores[,1] > 1]
```

## Evaluation signals

- Every reported metabolite appears in at least one input VIP > 1 list
- Counts satisfy |union| = |LMM list| + |M0 list| − |intersection| (69 + 66 − 60 gives the Venn structure: 9 LMM-unique, 6 M0-unique, 60 shared)
- LMM-processed data yields 69 metabolites with VIP > 1 and M0 data yields 66, matching Table S-3
- Computed ΔVIP values for cholesterol (0.52), SM (d40:2) (0.43) and Acylcarnitine C18:1 (0.32) match reported values within reasonable numerical tolerance
- Feature selection uses the VIP > 1 threshold on the first component, as stated in the methods

## Limitations

- Cholesterol was not identified by PLS-DA in the M0 dataset before LMM fitting (signals randomly mixed and indistinguishable), so M0-based selection can miss key lipids
- Increased sensitivity after LMM fitting carries the potential introduction of false positives
- Further experiments are recommended to validate the results (validation gap)

## Evidence

- [methods] VIP threshold on first component: "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [results] VIP-selected metabolite counts: "significant metabolites or discriminative metabolites (VIP > 1)"
- [methods] Tool roles for VIP extraction: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [methods] OPLS-DA variant: "OPLS-DA was performed with the ropls package"
- [results] Cholesterol missed in M0: "Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable"
