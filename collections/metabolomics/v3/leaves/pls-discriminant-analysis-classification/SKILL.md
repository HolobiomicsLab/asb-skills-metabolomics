---
name: pls-discriminant-analysis-classification
description: Use when a binary outcome (e.g. COPD vs control, mild vs severe COVID)
  must be predicted from high-dimensional multi-omics data — either pathway-score
  matrices or molecular-level blocks — and an interpretable latent-variable model
  is needed.
when_to_use_negative:
- The goal is unsupervised or purely descriptive pathway analysis (ORA/GSEA) rather
  than outcome prediction — conventional ORA, GSEA, MultiGSEA and IMPaLA were not
  directly compared by the authors
- Individuals lack measurements for all omics views, since PathIntegrate requires
  complete multi-omics samples and discards samples with entire rows of missing data
- The analysis targets molecules that cannot be annotated to pathways (no ChEBI/UniProt/gene
  mapping), as un-annotated molecules cannot be incorporated and are lost
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: PathIntegrate
  role: Framework providing Single-View (PLS-DA on multi-omics pathway scores) and
    Multi-View (MB-PLS) supervised models
  repo: https://github.com/cwieder/PathIntegrate
- name: DIABLO
  role: Baseline multi-omics integration method benchmarked against the PLS-based
    pathway models (used at molecular level and on pathway scores)
- name: SciKitLearn
  role: Hosts the SKlearn-compatible PLS-DA classifier and enables pipeline/GridSearchCV
    model optimisation and evaluation
- name: py-ssPA
  role: Generates the pathway-score input matrices (sspa_kpca etc.) and downloads
    KEGG/Reactome pathway definitions
  repo: https://github.com/cwieder/py-ssPA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:5aa705c80a288c884ec5e5c1db7cdc886577f3dbaa219604f728d4bdfdc47e21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  - py-ssPA
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# PLS-discriminant-analysis classification

## Summary

Fits a PLS-DA classifier (scikit-learn compatible) on pathway-transformed multi-omics scores to predict binary outcomes such as COPD or COVID-19 status, with the number of latent variables tuned by repeated nested cross-validation and performance reported as held-out test AUROC.

## When to use

Use when a binary outcome (e.g. COPD vs control, mild vs severe COVID) must be predicted from high-dimensional multi-omics data — either pathway-score matrices or molecular-level blocks — and an interpretable latent-variable model is needed.

## When NOT to use

- The goal is unsupervised or purely descriptive pathway analysis (ORA/GSEA) rather than outcome prediction — conventional ORA, GSEA, MultiGSEA and IMPaLA were not directly compared by the authors
- Individuals lack measurements for all omics views, since PathIntegrate requires complete multi-omics samples and discards samples with entire rows of missing data
- The analysis targets molecules that cannot be annotated to pathways (no ChEBI/UniProt/gene mapping), as un-annotated molecules cannot be incorporated and are lost

## Inputs

- Processed multi-omics abundance DataFrames (samples as rows, molecule IDs as columns), e.g. COPDgene metabolomics and proteomics
- Pathway matrices from KEGG or Reactome downloaded via the sspa package (GMT/custom GMT also supported)
- Binary outcome labels (e.g. COPD status, mild vs severe COVID-19)

## Outputs

- Fitted PLS-DA classifier with tuned number of latent variables
- Held-out test AUROC per model, effect size, and pathway database (tabulated e.g. as fig4A_auroc_results.csv)
- Pathway-level feature contributions for interpretation

## How to apply

First convert molecular abundance matrices (samples as rows, molecule IDs such as ChEBI/UniProt as columns) into pathway-level scores using an ssPA transformation (e.g. sspa_kpca with min_entity=2) for KEGG or Reactome pathway definitions. For the Single-View design, concatenate multi-omics pathway scores into one matrix and fit an SKlearn-compatible PLS-DA classifier; for Multi-View, use MB-PLS with a shared pathway-based latent space. Tune the number of latent variables via 5-times repeated nested 5-fold cross-validation, then record held-out test AUROC for each model, effect size, and pathway database. In the semi-synthetic COPDgene benchmark, pathway-based PLS models were compared against DIABLO at the molecular level as a baseline. Rationale: pathway transformation groups molecules into functional sets, boosting sensitivity to low signal-to-noise differential-abundance signals compared with molecular-level features.

## Related tools

- **PathIntegrate** (Framework providing Single-View (PLS-DA on multi-omics pathway scores) and Multi-View (MB-PLS) supervised models) — https://github.com/cwieder/PathIntegrate
- **DIABLO** (Baseline multi-omics integration method benchmarked against the PLS-based pathway models (used at molecular level and on pathway scores))
- **SciKitLearn** (Hosts the SKlearn-compatible PLS-DA classifier and enables pipeline/GridSearchCV model optimisation and evaluation)
- **py-ssPA** (Generates the pathway-score input matrices (sspa_kpca etc.) and downloads KEGG/Reactome pathway definitions) — https://github.com/cwieder/py-ssPA

## Examples

```
from sklearn.cross_decomposition import PLSRegression; pls = PLSRegression(n_components=5); pls.fit(X_pathway_scores, y); auroc = roc_auc_score(y_test, pls.predict(X_test))
```

## Evaluation signals

- Held-out test AUROC is recorded for every model, effect size, and pathway database via 5-times repeated nested 5-fold cross-validation
- Benchmark reproduces the reported pattern: pathway-based models (PathIntegrate Multi-View, Single-View, DIABLO pathway) outperform molecular-level DIABLO across all effect sizes
- KEGG pathway scores give marginally better AUROC than Reactome at larger effect sizes (Fig 4A/4C comparisons)
- ssPA transformation uses fit on training data and transform on held-out test data, so no leakage of test samples into pathway scoring

## Limitations

- PathIntegrate requires all input omics measured on the same individuals; samples missing an omics view must be discarded
- Pathway coverage (proportion of molecules observable in a pathway) is an inherent bottleneck, particularly for metabolomics
- Comparing molecular vs pathway-level models is difficult because the number of differentially abundant molecules per pathway is unknown and pathway definitions/sizes vary between databases

## Evidence

- [intro] single-view PLS-DA design: "PathIntegrate Single-View, which produces a multi-omics pathway-transformed dataset and applies a classification or regression model to the data"
- [methods] SKlearn-compatible model on pathway scores: "Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model"
- [supplementary] nested CV tuning of latent variables: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [readme] ssPA sklearn compatibility: [evidence span withheld — anchor status: external_doc]
