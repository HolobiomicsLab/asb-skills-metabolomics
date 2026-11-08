---
name: multi-omics-pathway-score-classification
description: Use when you have two or more molecular omics matrices measured on the
  same samples with an outcome label (e.g. COVID vs non-COVID or mild vs severe) and
  want predictive, interpretable multi-omics integration at the pathway level.
when_to_use_negative:
- Samples from individuals without measurements on all omics layers — PathIntegrate
  requires all input omics on the same individuals and such samples must be discarded
- The goal is conventional pathway analysis (ORA, GSEA, MultiGSEA, IMPaLA) without
  outcome prediction — a direct comparison was explicitly out of scope of this work
- Only one omics layer is available with no multi-omics integration requirement —
  a single-view pathway model still works but the multi-omics integration motivation
  is absent
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3365
- http://edamontology.org/topic_3517
tools:
- name: PathIntegrate
  role: Single-View framework that combines multi-omics pathway scores into one matrix
    and applies the SciKitLearn-compatible predictive model
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Provides the predictive classifiers (random forest, LogisticRegression, LinearSVC),
    cross-validation, and Pipeline/GridSearchCV machinery used on the pathway-score
    matrix
- name: Python
  role: Implementation language for the pathway transformation, model fitting and
    CSV output steps
  repo: https://github.com/cwieder/PathIntegrate
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:e86223fc7b8eedb87ebf0bc934f7cd708becff118700d7c2ed2528302c7e8014
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3365
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multi-omics-pathway-score-classification

## Summary

PathIntegrate Single-View supervised classification on ssPA pathway-transformed multi-omics data: combine all omics layers into one multi-omics pathway-score matrix and fit any SciKitLearn-compatible predictive model (demonstrated with a random forest achieving test AUC 0.95 on COVID-19 data from Su et al. 2020).

## When to use

Use when you have two or more molecular omics matrices measured on the same samples with an outcome label (e.g. COVID vs non-COVID or mild vs severe) and want predictive, interpretable multi-omics integration at the pathway level.

## When NOT to use

- Samples from individuals without measurements on all omics layers — PathIntegrate requires all input omics on the same individuals and such samples must be discarded
- The goal is conventional pathway analysis (ORA, GSEA, MultiGSEA, IMPaLA) without outcome prediction — a direct comparison was explicitly out of scope of this work
- Only one omics layer is available with no multi-omics integration requirement — a single-view pathway model still works but the multi-omics integration motivation is absent

## Inputs

- Multi-omics molecular abundance matrices (e.g. metabolomics and proteomics) with samples as rows, measured on the same individuals
- Binary or continuous outcome labels for the same samples (e.g. COVID vs non-COVID, mild vs severe)
- Multi-omics pathway database (Reactome v78 with ChEBI/UniProt/Gene Symbol identifiers, or KEGG v98) loadable via sspa.process_reactome / sspa.process_kegg

## Outputs

- Multi-omics ssPA pathway-score matrix (samples x pathway scores) feeding the Single-View model
- Cross-validated test AUROC of the pathway-level classifier (random-forest benchmark: 0.95)
- Ranked table of important multi-omics pathways (model-specific feature importances or coefficients)

## How to apply

First transform each molecular-level abundance matrix (e.g. the COVID-19 metabolomics and proteomics data from Su et al. 2020) into pathway-level scores using single-sample pathway analysis (ssPA), such as the kPCA method in py-ssPA with multi-omics Reactome/KEGG pathway databases. PathIntegrate Single-View then merges these into a single multi-omics pathway-score matrix (samples as rows, pathway scores as columns) and applies an SKlearn-compatible predictive model — the published benchmark used a random forest with repeated k-fold cross-validation, while alternative classifiers such as LogisticRegression or LinearSVC can be substituted without changing the framework. Evaluate performance via cross-validated AUROC on held-out out-of-fold predictions; the COVID-19 multi-omics benchmark achieved AUC 0.95 (molecular-level model: 0.98). Extract model-specific importance (e.g. standardised coefficients or permutation importance) to rank multi-omics pathways by contribution to prediction. The ssPA transformers expose fit/transform/fit_transform, enabling use inside SciKitLearn Pipeline and GridSearchCV so pathway scoring is fit on training folds only.

## Related tools

- **PathIntegrate** (Single-View framework that combines multi-omics pathway scores into one matrix and applies the SciKitLearn-compatible predictive model) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Provides the predictive classifiers (random forest, LogisticRegression, LinearSVC), cross-validation, and Pipeline/GridSearchCV machinery used on the pathway-score matrix)
- **Python** (Implementation language for the pathway transformation, model fitting and CSV output steps) — https://github.com/cwieder/PathIntegrate

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- The cross-validated test AUROC on the COVID-19 multi-omics pathway scores should be reported and compared against the published random-forest Single-View benchmark of 0.95 (molecular-level model: 0.98)
- The output must contain a numeric cross-validated AUROC derived from out-of-fold predictions, not training-set fit statistics
- The ranked pathway table must be non-empty and contain multi-omics pathway identifiers drawn from the Reactome/KEGG database used for transformation
- ssPA transformation must be fit on training data only (fit/transform or within Pipeline), confirmed by the transformers' SciKitLearn fit_transform compatibility
- All samples in the final pathway-score matrix must have complete measurements across every input omics layer

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, and un-annotated molecules cannot be incorporated, causing information loss
- It remains a challenge to compare molecular vs pathway-level models because it is unknown how many molecules in a pathway are differentially abundant and pathway definitions/sizes vary between databases

## Evidence

- [methods] Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model: "Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model"
- [intro] Single-View definition: "PathIntegrate consists of two supervised learning frameworks for pathway-based multi-omics integration: PathIntegrate Single-View, which produces a multi-omics pathway-transformed dataset"
- [intro] SciKitLearn compatibility: "designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation."
- [readme] ssPA sklearn compatibility: [evidence span withheld — anchor status: external_doc]
- [discussion] Applied datasets: "We applied PathIntegrate to two datasets: COPDgene and COVID19 multi-omics."
