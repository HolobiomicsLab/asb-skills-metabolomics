---
name: bootstrap-variance-estimation
description: Use when a fitted PathIntegrate Multi-View MB-PLS model (e.g. COPDgene,
  tuned to 4 latent variables) needs a resampling check that the per-block y-variance
  explained by each latent variable is stable, complementing permutation p-values
  on MB-VIP.
when_to_use_negative:
- The MB-PLS number of latent variables has not yet been tuned — run the 5-times repeated
  nested 5-fold cross-validation first
- The question of interest is pathway-level significance rather than stability of
  block-level explained variance — use MB-VIP permutation testing instead
- The omics views are not measured on the same individuals, since PathIntegrate cannot
  handle samples missing entire omics rows
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: PathIntegrate
  role: Provides the Multi-View MB-PLS model whose y-variance explained per latent
    variable is re-estimated across bootstrap resamples
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa
  role: Supplies the kPCA single-sample pathway transformation that produces the pathway-scored
    views fed to the bootstrapped MB-PLS refits
  repo: https://github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Compatibility layer used by PathIntegrate for model fitting and resampling
    utilities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:5c7f8e9657fc9f7da6caef04c6b230c6e986641488ce236a135d2e55b2ad531e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bootstrap-variance-estimation

## Summary

Resampling-based stability assessment for PathIntegrate Multi-View MB-PLS models: refit the tuned multi-block PLS on 100 bootstrap resamples and record the y-variance explained per latent variable per omics view to show how stable each block's contribution is.

## When to use

Use when a fitted PathIntegrate Multi-View MB-PLS model (e.g. COPDgene, tuned to 4 latent variables) needs a resampling check that the per-block y-variance explained by each latent variable is stable, complementing permutation p-values on MB-VIP.

## When NOT to use

- The MB-PLS number of latent variables has not yet been tuned — run the 5-times repeated nested 5-fold cross-validation first
- The question of interest is pathway-level significance rather than stability of block-level explained variance — use MB-VIP permutation testing instead
- The omics views are not measured on the same individuals, since PathIntegrate cannot handle samples missing entire omics rows

## Inputs

- Pathway-transformed multi-omics matrices (kPCA/ssPA pathway scores per omics view, samples as rows)
- Binary outcome labels (e.g. COPD status)
- Fitted MB-PLS model specification with the tuned number of latent variables (4 in the COPDgene case)

## Outputs

- Per-latent-variable, per-omics-block y-variance explained across 100 bootstrap resamples (e.g. metabolomics 6-17% range)
- Stability summaries (distributions or ranges of explained variance) supporting interpretation of block importance

## How to apply

After tuning the number of latent variables by 5-times repeated nested 5-fold cross-validation (expected optimum 4 LVs, mean cross-validated AUC 0.70 in the COPDgene case), take the pathway-transformed views (kPCA/ssPA scores on Reactome pathways) and the outcome labels. Draw 100 bootstrap resamples of the samples (sampling rows with replacement, keeping all omics views aligned since PathIntegrate requires all input omics measured on the same individuals), refit the MB-PLS model on each resample, and record the y-variance explained by each latent variable for each omics block. Summarise the per-block, per-LV distributions (in the COPDgene model the metabolomics block explained 6-17% of y-variance across latent variables) and report narrow ranges as evidence that a block's contribution is stable rather than a cross-validation artefact. This complements, but does not replace, block-wise MB-VIP permutation testing (e.g. metabolomics carnitine metabolism p = 0.003, proteomics surfactant metabolism p = 0.002), which addresses pathway-level significance rather than block-level variance stability.

## Related tools

- **PathIntegrate** (Provides the Multi-View MB-PLS model whose y-variance explained per latent variable is re-estimated across bootstrap resamples) — https://github.com/cwieder/PathIntegrate
- **sspa** (Supplies the kPCA single-sample pathway transformation that produces the pathway-scored views fed to the bootstrapped MB-PLS refits) — https://github.com/cwieder/py-ssPA
- **SciKitLearn** (Compatibility layer used by PathIntegrate for model fitting and resampling utilities)

## Evaluation signals

- Bootstrap replicate count equals 100 and each resample is drawn from the same aligned samples across all omics views
- The reported per-LV y-variance explained ranges match expectations, e.g. metabolomics 6-17% across latent variables in the COPDgene Multi-View model
- Bootstrap ranges are computed for each omics block separately, not pooled across blocks
- The base (unresampled) model still reproduces the validation anchors: 4 latent variables and mean cross-validated AUC 0.70
- Stability conclusions are consistent with the independent MB-VIP permutation results (metabolomics-driven carnitine metabolism p=0.003; proteomics-driven surfactant metabolism p=0.002)

## Limitations

- Bootstrap ranges quantify stability of explained variance but are not significance tests; pathway-level significance must come from MB-VIP permutation testing
- PathIntegrate requires all input omics measured on the same individuals, so samples missing any omics must be discarded and cannot contribute to resamples
- Results depend on the tuned latent-variable setting; changing the LV count invalidates comparison with the published 6-17% metabolomics range

## Evidence

- [abstract] PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level, and applies a predictive single-view or multi-view model to integrate the data.: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level, and applies a predictive single-view"
- [supplementary] Nested CV tuning of latent variables is the documented model-selection step preceding stability assessment.: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [readme] ssPA transformations are sklearn-compatible, enabling scripted resampling and refitting pipelines.: [evidence span withheld — anchor status: external_doc]
- [readme] PathIntegrate outputs include the contribution of each omics layer, the quantity whose stability the bootstrap estimates.: "Model outputs include multi-omics pathways ranked by their contribution to the outcome prediction, the contribution of each omics layer"
