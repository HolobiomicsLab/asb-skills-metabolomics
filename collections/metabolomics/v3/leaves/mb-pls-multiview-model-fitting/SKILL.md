---
name: mb-pls-multiview-model-fitting
description: Use when two or more omics datasets (e.g. metabolomics and proteomics)
  measured on the same samples have been transformed to pathway-level scores and you
  need a predictive, integrative multi-view model that ranks pathways and omics-layer
  contributions.
when_to_use_negative:
- You want to benchmark a molecular-level model — DIABLO at the molecular level is
  the appropriate non-pathway baseline, not MB-PLS on pathway scores
- Samples are not measured across all omics views — PathIntegrate requires all input
  omics on the same individuals and discards incomplete samples
- A purely exploratory pathway analysis without outcome prediction is needed — conventional
  ORA/GSEA-style tools are out of scope for this skill
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_0091
tools:
- name: PathIntegrate
  role: Provides the Multi-View MB-PLS modelling framework that fits a shared pathway-based
    latent space across pathway-transformed omics blocks
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa (py-ssPA)
  role: Supplies single-sample pathway analysis (e.g. sspa_kpca) to generate the pathway-level
    input matrices and to download KEGG/Reactome pathways
  repo: https://github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Backend for model optimisation and evaluation; the Multi-View model is built
    to be compatible with sklearn functions such as GridSearchCV for latent-variable
    tuning
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:5544da69d6c5be0da03286ed0a033f90de3e8c8b2105385c24df6ea2ac1bc050
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa (py-ssPA)
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# mb-pls-multiview-model-fitting

## Summary

Fit a PathIntegrate Multi-View model that integrates multiple pathway-transformed omics datasets with a multi-block partial least squares (MB-PLS) regression using a shared pathway-based latent space, for supervised outcome prediction and pathway interpretation.

## When to use

Use when two or more omics datasets (e.g. metabolomics and proteomics) measured on the same samples have been transformed to pathway-level scores and you need a predictive, integrative multi-view model that ranks pathways and omics-layer contributions.

## When NOT to use

- You want to benchmark a molecular-level model — DIABLO at the molecular level is the appropriate non-pathway baseline, not MB-PLS on pathway scores
- Samples are not measured across all omics views — PathIntegrate requires all input omics on the same individuals and discards incomplete samples
- A purely exploratory pathway analysis without outcome prediction is needed — conventional ORA/GSEA-style tools are out of scope for this skill

## Inputs

- Pathway-transformed multi-omics DataFrames (samples as rows, pathway scores as columns) from ssPA transformation of e.g. COPDgene metabolomics and proteomics data
- Outcome labels (e.g. COPD status or gender) for supervised classification
- KEGG and Reactome pathway matrices downloaded via the sspa package

## Outputs

- Held-out test AUROC per model, effect size, and pathway database (e.g. fig4A_auroc_results.csv)
- Multi-omics pathways ranked by contribution to outcome prediction
- Per-omics-layer contribution scores and molecule-level importance within pathways
- Tuned number of latent variables from nested cross-validation

## How to apply

First transform each molecular-level abundance matrix (samples as rows, molecule IDs such as ChEBI/UniProt as columns) into pathway-level scores using single-sample pathway analysis (ssPA, e.g. sspa_kpca) for each pathway database (KEGG, Reactome). Then fit the PathIntegrate Multi-View model, an MB-PLS classifier that models interactions between the pathway-transformed omics blocks through a shared pathway-based latent space, using the scikit-learn-compatible PathIntegrate package. Tune the number of latent variables via 5-times repeated nested 5-fold cross-validation before evaluating performance. Record held-out test AUROC per effect size and pathway database, and extract model outputs including pathways ranked by contribution to outcome prediction, per-omics-layer contributions, and molecule importance within pathways.

## Related tools

- **PathIntegrate** (Provides the Multi-View MB-PLS modelling framework that fits a shared pathway-based latent space across pathway-transformed omics blocks) — https://github.com/cwieder/PathIntegrate
- **sspa (py-ssPA)** (Supplies single-sample pathway analysis (e.g. sspa_kpca) to generate the pathway-level input matrices and to download KEGG/Reactome pathways) — https://github.com/cwieder/py-ssPA
- **SciKitLearn** (Backend for model optimisation and evaluation; the Multi-View model is built to be compatible with sklearn functions such as GridSearchCV for latent-variable tuning)

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2]); model = PathIntegrate.MultiView(n_latent_vars=cv_best).fit(kpca_scores, y)
```

## Evaluation signals

- Nested cross-validation must be used for latent-variable tuning (5-times repeated nested 5-fold CV) before reporting test AUROC
- Held-out test AUROC should be recorded for every model, effect size, and pathway database, tabulated per effect size
- Benchmark acceptance: pathway-based models (Multi-View, Single-View, DIABLO pathway) outperform molecular-level DIABLO across all effect sizes, with KEGG marginally better than Reactome at larger effect sizes
- All input omics blocks must contain the same individuals in the same rows; samples missing any omics must be dropped beforehand

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics
- Un-annotated molecules cannot be incorporated into pathway-based models, causing information loss
- Comparison of molecular vs. pathway-level models is challenging because the number of differentially abundant molecules per pathway is unknown and pathway definitions/sizes vary between databases

## Evidence

- [methods] Multi-View model: Integrates multiple omics datasets using a shared pathway-based latent space: "Multi-View model: Integrates multiple omics datasets using a shared pathway-based latent space"
- [intro] MB-PLS modelling of interactions: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
- [supplementary] Nested CV parameter tuning: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [supplementary] Benchmark outcome: "Pathway transformation enhances sensitivity to low signal-to-noise signals (COPDgene semi synthetic data)"
- [readme] Model outputs include pathway rankings and omics-layer contributions: "Model outputs include multi-omics pathways ranked by their contribution to the outcome prediction, the contribution of each omics layer, and the importance of each molecule in a pathway."
