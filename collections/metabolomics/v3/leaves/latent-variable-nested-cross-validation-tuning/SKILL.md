---
name: latent-variable-nested-cross-validation-tuning
description: Use when fitting PathIntegrate Multi-View MB-PLS models on pathway-transformed
  (ssPA) or molecular-level multi-omics matrices and the number of latent variables
  must be optimised before reporting cross-validated AUC.
when_to_use_negative:
- The model being optimised has no latent-variable/number-of-components parameter
  (e.g. non-PLS classifiers in Single-View benchmarks) — there is nothing to tune
  with this scheme
- Only a single fixed-complexity model is needed without performance evaluation; running
  5-times repeated nested 5-fold CV adds substantial compute for no benefit
- Input omics are not measured on the same individuals — PathIntegrate cannot handle
  rows with missing omics, so samples must be discarded before cross-validation splits
  are meaningful
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3391
tools:
- name: PathIntegrate
  role: Provides the Multi-View MB-PLS model whose number of latent variables is tuned
    via repeated nested 5-fold cross-validation
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Supplies GridSearchCV / cross-validation machinery used for latent-variable
    parameter tuning, exploiting PathIntegrate's sklearn compatibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:e51636affeb3e3693ce1cfd552b61528fa734865f97d4cca80b04ba512c8a4b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9310344827586207
n_claims: 29
---

# latent-variable-nested-cross-validation-tuning

## Summary

Tunes the number of latent variables of MB-PLS multi-omics models via 5-times repeated nested 5-fold cross-validation, the parameter-tuning scheme used in PathIntegrate Multi-View, so that models of different input representation (pathway-level vs molecular-level) are compared at matched, optimised complexity.

## When to use

Use when fitting PathIntegrate Multi-View MB-PLS models on pathway-transformed (ssPA) or molecular-level multi-omics matrices and the number of latent variables must be optimised before reporting cross-validated AUC.

## When NOT to use

- The model being optimised has no latent-variable/number-of-components parameter (e.g. non-PLS classifiers in Single-View benchmarks) — there is nothing to tune with this scheme
- Only a single fixed-complexity model is needed without performance evaluation; running 5-times repeated nested 5-fold CV adds substantial compute for no benefit
- Input omics are not measured on the same individuals — PathIntegrate cannot handle rows with missing omics, so samples must be discarded before cross-validation splits are meaningful

## Inputs

- Multi-omics abundance matrices (samples as rows, molecule IDs as columns), e.g. COPDgene plasma proteomics/metabolomics data, log-transformed and z-score standardised
- Optionally ssPA pathway-scored matrices (e.g. sspa_kpca kPCA pathway scores on Reactome/KEGG pathways) for the pathway-level variant
- Binary outcome labels (e.g. COPD status) for supervised classification
- Candidate range of latent variable counts for the MB-PLS model

## Outputs

- Optimal number of latent variables per model (e.g. 4 vs 6 in the COPDgene molecular-vs-pathway comparison, Table 2)
- Mean cross-validated AUC with spread (expected ~0.70 +/- 0.02 for both COPDgene MB-PLS variants)
- Fitted MB-PLS model with the tuned latent variable count
- Comparison table (e.g. copdgene_molecular_vs_pathway_mbpls_performance.csv)

## How to apply

Wrap the MB-PLS classifier (PathIntegrate Multi-View, or a molecular-level MB-PLS fitted directly on molecule abundances) in a SciKitLearn-compatible hyperparameter search over the number of latent variables, since PathIntegrate is deliberately designed to be compatible with SciKitLearn functions such as GridSearchCV. Use 5-times repeated nested 5-fold cross-validation for the tuning, as specified for PathIntegrate Multi-View parameter tuning in the paper's supplementary material; tune only on inner folds and report performance on the outer loop to avoid optimistic bias. Apply the identical tuning scheme to every model being compared — e.g. the pathway-level model on ssPA-transformed data and the molecular-level model on raw molecule abundances — so that differences in optimal model complexity are attributable to the representation, not the search. On the COPDgene plasma multi-omics data this procedure yields an average AUC of 0.70 (+/-0.02) for both molecular-level and pathway-level MB-PLS, with the pathway-level model reaching its optimum with fewer latent variables than the molecular-level model (Table 2). Record the optimal latent variable count and the cross-validated AUC (mean and spread) as the comparison metrics.

## Related tools

- **PathIntegrate** (Provides the Multi-View MB-PLS model whose number of latent variables is tuned via repeated nested 5-fold cross-validation) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Supplies GridSearchCV / cross-validation machinery used for latent-variable parameter tuning, exploiting PathIntegrate's sklearn compatibility)

## Examples

```
gs = GridSearchCV(mbpls_model, param_grid={'n_components': range(2, 11)}, cv=RepeatedKFold(n_splits=5, n_repeats=5), scoring='roc_auc'); gs.fit(X, y)
```

## Evaluation signals

- Cross-validation configuration must match 5-times repeated nested 5-fold CV (5 folds, 5 repeats), with latent-variable selection confined to inner folds
- Identical tuning scheme must be applied to both compared models (pathway-level and molecular-level MB-PLS); differing schemes invalidate the comparison
- On COPDgene data, both MB-PLS variants should reproduce average AUC ~0.70 +/- 0.02
- The pathway-level (ssPA-transformed) model should reach its AUC optimum with fewer latent variables than the molecular-level model
- Reported AUC must come from the outer CV loop, not from the inner-tuning folds or a refit on full data

## Limitations

- PathIntegrate requires all input omics measured on the same individuals; samples from individuals lacking any omics layer must be discarded, shrinking fold sizes
- Comparing molecular vs pathway-level models is inherently difficult because the number of differentially abundant molecules per pathway is unknown and pathway definitions/sizes vary between databases
- Pathway-level models cannot use molecules unannotated to any pathway, an information loss that partly explains the higher latent-variable complexity of the molecular-level model
- Benchmarking scripts were developed for a PBS HPC cluster, reflecting the computational cost of repeated nested cross-validation

## Evidence

- [supplementary] Nested CV scheme for Multi-View tuning: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [methods] MB-PLS is the tuned model class: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
- [readme] sklearn compatibility enables GridSearchCV tuning: [evidence span withheld — anchor status: external_doc]
- [discussion] Why molecular vs pathway comparison is hard: "it remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant at any one time"
