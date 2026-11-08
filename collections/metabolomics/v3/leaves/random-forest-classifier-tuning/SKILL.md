---
name: random-forest-classifier-tuning
description: Use when a binary-classification outcome is being predicted from pathway-transformed
  multi-omics abundance data (e.g. Su et al. COVID-19 mild vs moderate/severe metabolomics/proteomics
  scores) and an interpretable pathway-level model with feature selection is required.
when_to_use_negative:
- When the goal is pathway-level integration of omics with latent-variable modelling
  of interactions between omics blocks — use PathIntegrate Multi-View (MB-PLS) instead
  of a single-view random forest.
- When the input is already a molecular-level feature table and no pathway transformation
  is desired — applying a random forest directly to concatenated molecular data yields
  the higher-dimensional baseline (AUC ≈0.98) rather than an interpretable pathway-level
  model.
- When samples from individuals lack data on all omics views — PathIntegrate requires
  all input omics measured on the same individuals, so such samples must be discarded
  before model fitting.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: PathIntegrate
  role: Produces the multi-omics pathway-transformed dataset (Single-View) that the
    RandomForestClassifier is trained on, and handles automatic standardisation of
    the scores.
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa
  role: Downloads Reactome multi-omics pathway definitions and computes single-sample
    pathway scores (SVD/PLAGE) that become the random-forest features.
  repo: https://github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Provides RandomForestClassifier, GridSearchCV/nested cross-validation, and
    RFECV for pathway selection and model evaluation.
- name: Python
  role: Runtime environment in which PathIntegrate, sspa and scikit-learn are orchestrated
    (scripts run under Python 10 per the benchmarking repo).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:72df4f4958c504f2e9099662082dd642d56d8fe8529ac32e2f71ae055b88c1fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# random-forest-classifier-tuning

## Summary

Train and hyperparameter-tune a RandomForestClassifier on multi-omics pathway score matrices produced by PathIntegrate Single-View, then use recursive feature elimination with cross-validation (RFECV) to select the predictive pathways for a binary outcome such as COVID-19 severity (mild vs moderate/severe).

## When to use

Use when a binary-classification outcome is being predicted from pathway-transformed multi-omics abundance data (e.g. Su et al. COVID-19 mild vs moderate/severe metabolomics/proteomics scores) and an interpretable pathway-level model with feature selection is required.

## When NOT to use

- When the goal is pathway-level integration of omics with latent-variable modelling of interactions between omics blocks — use PathIntegrate Multi-View (MB-PLS) instead of a single-view random forest.
- When the input is already a molecular-level feature table and no pathway transformation is desired — applying a random forest directly to concatenated molecular data yields the higher-dimensional baseline (AUC ≈0.98) rather than an interpretable pathway-level model.
- When samples from individuals lack data on all omics views — PathIntegrate requires all input omics measured on the same individuals, so such samples must be discarded before model fitting.

## Inputs

- Multi-omics pathway score matrix produced by PathIntegrate Single-View (samples × pathways, e.g. SVD/PLAGE scores over Reactome multi-omics pathways with ≥2 measured molecules)
- Binary outcome label vector (e.g. mild vs moderate/severe COVID-19 status from Su et al. 2020, 127 samples)
- Reactome (or KEGG) multi-omics pathway definitions from sspa.process_reactome / process_gmt
- Molecular-level concatenated omics matrix (metabolomics + proteomics abundance values) for the baseline comparison model

## Outputs

- Tuned RandomForestClassifier fitted to pathway scores
- Ranked list of selected predictive pathways from RFECV (20 pathways in the COVID-19 severity case study)
- Held-out test-set classification metrics, primarily AUC (≈0.95 for pathway-level vs ≈0.98 for molecular-level baseline)
- Per-pathway pathway coverage (proportion of pathway molecules measured in the data)

## How to apply

First transform the molecular-level abundance matrices (Su et al. COVID-19 metabolomics and proteomics, log-transformed, ChEBI/UniProt-identified) into multi-omics pathway scores using the SVD-based PLAGE method via sspa (retaining pathways with at least 2 measured molecules; ~732 multi-omics Reactome pathways retained). Fit a scikit-learn RandomForestClassifier to the pathway score matrix under nested cross-validation, tuning hyperparameters such as the number of trees and depth via GridSearchCV on the inner folds. Then apply recursive feature elimination with cross-validation (RFECV) on the tuned forest to select the most predictive pathways — for the COVID-19 severity case study this reduced the feature set to 20 pathways (e.g. Interleukin-5/13 signalling). Evaluate the tuned pathway-level model on held-out test folds and compare its AUC against a molecular-level concatenated baseline (expected pathway AUC ≈ 0.95 vs 0.98 for the molecular model), also reporting the per-pathway molecule coverage.

## Related tools

- **PathIntegrate** (Produces the multi-omics pathway-transformed dataset (Single-View) that the RandomForestClassifier is trained on, and handles automatic standardisation of the scores.) — https://github.com/cwieder/PathIntegrate
- **sspa** (Downloads Reactome multi-omics pathway definitions and computes single-sample pathway scores (SVD/PLAGE) that become the random-forest features.) — https://github.com/cwieder/py-ssPA
- **SciKitLearn** (Provides RandomForestClassifier, GridSearchCV/nested cross-validation, and RFECV for pathway selection and model evaluation.)
- **Python** (Runtime environment in which PathIntegrate, sspa and scikit-learn are orchestrated (scripts run under Python 10 per the benchmarking repo).)

## Examples

```
pathway_scores = sspa.sspa_SVD(reactome_pathways, min_entity=2).fit_transform(log2_covid_multiomics); rf = RandomForestClassifier(n_estimators=500); tuned = GridSearchCV(rf, param_grid, cv=inner_cv); tuned.fit(pathway_scores, labels); sel = RFECV(tuned.best_estimator_, cv=outer_cv).fit(pathway_scores, labels)
```

## Evaluation signals

- The selected pathway count from RFECV matches the reported 20 pathways and includes biologically plausible examples such as Interleukin-5/13 signalling.
- Held-out test-set AUC is ≈0.95 for the pathway-level model, consistent with the published value, and remains close to (within ~0.03 of) the molecular-level concatenated baseline AUC of ≈0.98.
- The pathway score matrix contains only pathways with at least 2 measured molecules (multi-omics Reactome pathways ≈732 at this threshold).
- Nested cross-validation hyperparameter tuning is performed on inner folds only, and no tuning information leaks into the held-out test evaluation.
- Per-pathway pathway coverage values are computed and reported for the selected pathways, indicating how many molecules in each pathway were measured.

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded because PathIntegrate currently does not support entire rows of missing data.
- Pathway coverage — the proportion of molecules in a pathway observable in the data — is an inherent bottleneck of pathway-based analyses, particularly for metabolomics.
- Comparing molecular vs pathway-level models is challenging because it is unknown how many molecules in a pathway are differentially abundant at any one time, and pathway definitions and sizes vary between databases.
- The reliance on current pathway annotation knowledge is a widely accepted limitation; un-annotated molecules cannot be incorporated, causing information loss.

## Evidence

- [methods] single-view framework uses sklearn RFECV for pathway selection: [evidence span withheld — anchor status: not_found]
- [readme] SSPA min_entity=2 threshold for pathway scoring: [evidence span withheld — anchor status: external_doc]
- [methods] PathIntegrate Single-View applies sklearn-compatible model: "Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model"
- [intro] Article claims no existing method does predictive pathway-level multi-omics integration: "we are unaware of any one method which provides predictive, integrative modelling of multi-omics data at the pathway-level."
- [discussion] Sample discard limitation when omics views are incomplete: "PathIntegrate requires all input omics to be measured on the same individuals. This means samples from individuals without data on all omics will have to be discarded"
- [readme] ssPA methods fit/transform interface compatible with sklearn Pipeline and GridSearchCV: [evidence span withheld — anchor status: external_doc]
