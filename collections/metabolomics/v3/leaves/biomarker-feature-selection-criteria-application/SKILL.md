---
name: biomarker-feature-selection-criteria-application
description: Use when you have a metabolite intensity matrix with phenotype groups
  and must pick differential variables (e.g., Kruskal-Wallis with Tukey post-hoc P
  < 0.05 plus |log2FC| > 0.25) before classifier construction.
when_to_use_negative:
- Raw spectral data that still needs peak picking and annotation — iMAP deliberately
  does not implement raw data preprocessing or peak picking/annotation modules
- The full dataset including the held-out test set — selection must be performed on
  the train set only to avoid leakage into model validation
- Unnormalized data with excessive missing values — run the data-processing module
  (missing-value handling, normalization) first
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: iMAP
  role: Web server whose metabolites-selection module performs the statistical analysis
    (Kruskal-Wallis/Tukey, log2FC) to select key metabolites after data-processing
  repo: https://imap.metaboprofile.cloud/
- name: IP4M
  role: Predecessor desktop platform whose R scripts (R-scripts-for-iMAP) implement
    the backend analysis for iMAP
  repo: https://github.com/IP4M/R-scripts-for-iMAP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3389/fchem.2021.659656
    title: 'iMAP: A Web Server for Metabolomics Data Integrative Analysis'
  source_content_hash: sha256:ecb7075ddc0e0ddf0bfa3f2b4047ee917edc126e1c5ed3c287c76a28df465b76
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IP4M
  - iMAP
  - pROC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# biomarker-feature-selection-criteria-application

## Summary

Selects candidate biomarker metabolites from an intensity matrix using combined statistical significance and fold-change criteria, as implemented in iMAP's metabolites-selection step, so that only defensible features feed downstream predictive model building.

## When to use

Use when you have a metabolite intensity matrix with phenotype groups and must pick differential variables (e.g., Kruskal-Wallis with Tukey post-hoc P < 0.05 plus |log2FC| > 0.25) before classifier construction.

## When NOT to use

- Raw spectral data that still needs peak picking and annotation — iMAP deliberately does not implement raw data preprocessing or peak picking/annotation modules
- The full dataset including the held-out test set — selection must be performed on the train set only to avoid leakage into model validation
- Unnormalized data with excessive missing values — run the data-processing module (missing-value handling, normalization) first

## Inputs

- Metabolite intensity matrix (samples x metabolites) with group labels, e.g., 234 serum samples x 113 targeted metabolites (Zhu et al., 2014)
- Stratified train-set split of the samples (e.g., 8:2 train-test separation)
- Phenotype group assignments (e.g., HC, PolyP, CRC)

## Outputs

- List of selected differential metabolites (e.g., Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, Linolenic Acid)
- Per-metabolite Kruskal-Wallis / Tukey post-hoc P values and pairwise log2 fold-change values supporting the selection

## How to apply

Load the targeted metabolite intensity matrix (e.g., 234 samples: 66 CRC, 76 PolyP, 92 HC; 113 metabolites from Zhu et al., 2014) and restrict selection to the train set after stratified train-test splitting (8:2). Apply the article's dual criterion: pairwise Tukey post-hoc P < 0.05 in at least one group pair after a Kruskal-Wallis test, AND |log2FC| > 0.25 for both CRC vs. PolyP and CRC vs. HC contrasts. On the Zhu serum dataset this retains four metabolites — Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid — which then serve as inputs to random forest and elastic-net classifiers. Rationale: iMAP's metabolites-selection module performs statistical analysis after data-processing, and combining a nonparametric omnibus test with pairwise post-hoc significance and effect-size cutoffs avoids feeding noisy, non-differential features into model training.

## Related tools

- **iMAP** (Web server whose metabolites-selection module performs the statistical analysis (Kruskal-Wallis/Tukey, log2FC) to select key metabolites after data-processing) — https://imap.metaboprofile.cloud/
- **IP4M** (Predecessor desktop platform whose R scripts (R-scripts-for-iMAP) implement the backend analysis for iMAP) — https://github.com/IP4M/R-scripts-for-iMAP

## Evaluation signals

- On the Zhu et al. 2014 serum dataset with the stated criteria, exactly four metabolites are selected: Adenylosuccinate, Histidine, Hydroxyproline/Aminolevulinate, and Linolenic Acid
- Every selected metabolite satisfies Tukey post-hoc P < 0.05 in at least one group pair AND |log2FC| > 0.25 for both CRC vs. PolyP and CRC vs. HC
- Selection is computed on the train set only; models built from the selected features reproduce Table 2 AUCs (e.g., RF HC vs. CRC 0.743 test / 1 train; elastic net HC vs. PolyP 0.5/0.5)
- Train-set AUC exceeds test-set AUC in all cases except elastic net between HC and PolyP

## Limitations

- Selection criteria are user-editable but the exact thresholds (P < 0.05, |log2FC| > 0.25) were tuned for this three-group CRC/polyp/control serum dataset and may not transfer directly
- No version number is stated for iMAP or IP4M, and the repository has no changelog, so reproducing the exact selection implementation depends on the current R-scripts-for-iMAP code
- Raw data preprocessing and peak picking/annotation are not implemented in iMAP, so inputs must already be an annotated intensity matrix

## Evidence

- [abstract] A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation.: "A new module was developed with an automatic pipeline for train-test set separation, feature selection, and predictive model construction and validation."
- [methods] After data-processing, statistical analysis will be performed in various ways to select key metabolites: [evidence span withheld — anchor status: not_found]
- [intro] Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security: "Modules of raw data preprocessing and peak picking and annotation were not implemented considering the risk of data security"
- [readme] iMAP has more flexible workflows and more optional criteria for differential variables selection.: "Compared with existing platforms, iMAP has more flexible workflows and more optional criteria for differential variables selection."
- [methods] The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP: "The backend R scripts are available at https://github.com/IP4M/R-scripts-for-iMAP"
