---
name: multi-block-partial-least-squares-modeling
description: Use when integrating two or more pathway-transformed omics matrices (samples
  as rows, pathway scores as columns) to predict a binary outcome and rank pathways
  with block-specific importances.
when_to_use_negative:
- Input data is raw molecular abundances and you want to include molecules not annotated
  to any pathway — a molecular-level MB-PLS (skipping ssPA) achieved the same AUC
  0.70 but needed more latent variables (6 vs 4)
- Samples lack measurements across all omics views — PathIntegrate requires all input
  omics measured on the same individuals and discards incomplete samples
- You only need unsupervised pathway enrichment without outcome prediction — conventional
  approaches (ORA, GSEA, MultiGSEA, IMPaLA) serve that purpose and were not benchmarked
  here
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_0091
tools:
- name: PathIntegrate
  role: Provides the Multi-View MB-PLS framework with MB-VIP importance and permutation
    testing on pathway-transformed views
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa
  role: Performs the kPCA single-sample pathway transformation and downloads Reactome
    pathways that form the MB-PLS input blocks
  repo: https://github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Supplies compatible model optimisation and cross-validation machinery for
    tuning latent variables
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:7f04293d0770b2a353d2cff54f4a9b0303a35de8c27f6193dde82e4812200064
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# multi-block-partial-least-squares-modeling

## Summary

Fits a multi-block partial least squares (MB-PLS) model on pathway-transformed multi-omics views to integrate them in a shared latent space, producing block-wise MB-VIP pathway importances and outcome prediction. Used in PathIntegrate Multi-View to associate pathways with outcomes across omics layers.

## When to use

Use when integrating two or more pathway-transformed omics matrices (samples as rows, pathway scores as columns) to predict a binary outcome and rank pathways with block-specific importances.

## When NOT to use

- Input data is raw molecular abundances and you want to include molecules not annotated to any pathway — a molecular-level MB-PLS (skipping ssPA) achieved the same AUC 0.70 but needed more latent variables (6 vs 4)
- Samples lack measurements across all omics views — PathIntegrate requires all input omics measured on the same individuals and discards incomplete samples
- You only need unsupervised pathway enrichment without outcome prediction — conventional approaches (ORA, GSEA, MultiGSEA, IMPaLA) serve that purpose and were not benchmarked here

## Inputs

- Pathway-transformed omics matrices (samples as rows, Reactome pathway IDs as columns) for each omics view (e.g. COPDgene metabolomics, proteomics, transcriptomics)
- Binary outcome labels (e.g. COPD status)
- Reactome human pathway definitions retrieved via the sspa package

## Outputs

- Cross-validated model performance (mean AUC) and optimal number of latent variables
- Block-wise MB-VIP pathway importance scores per omics view
- Permutation p-values for MB-VIP pathway statistics
- Bootstrap y-variance explained per latent variable
- Ranked pathway table (pathway name, omics view, MB-VIP score, permutation p-value, e.g. mbpls_pathway_importance.csv)

## How to apply

Transform each omics matrix to pathway-level scores using kPCA single-sample pathway analysis (ssPA) on Reactome pathways, then fit the PathIntegrate Multi-View MB-PLS model with the outcome as response. Tune the number of latent variables by 5-times repeated nested 5-fold cross-validation, selecting the setting with best cross-validated AUC (for COPDgene, 4 latent variables gave mean AUC 0.70). Compute block-wise MB-VIP pathway importance per omics view and assess significance by permutation testing of the MB-VIP statistic (e.g. carnitine metabolism: metabolomics p=0.003, transcriptomics p=0.55). Optionally run 100 bootstrap resamples to estimate stability of y-variance explained per latent variable (metabolomics 6-17% across latent variables). Export a ranked pathway table with pathway name, omics view, MB-VIP score, and permutation p-value.

## Related tools

- **PathIntegrate** (Provides the Multi-View MB-PLS framework with MB-VIP importance and permutation testing on pathway-transformed views) — https://github.com/cwieder/PathIntegrate
- **sspa** (Performs the kPCA single-sample pathway transformation and downloads Reactome pathways that form the MB-PLS input blocks) — https://github.com/cwieder/py-ssPA
- **SciKitLearn** (Supplies compatible model optimisation and cross-validation machinery for tuning latent variables)

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(omics_matrix); pi = PathIntegrate.MultiViewMBPLS(views=[kpca_scores]); pi.fit_predict(y, n_components=4)
```

## Evaluation signals

- Cross-validated AUC matches the expected optimum: COPDgene pathway-level MB-PLS with 4 latent variables gives mean AUC 0.70 (±0.02)
- Reported pathways attain the stated permutation p-values: carnitine metabolism significant in metabolomics (p=0.003) but not transcriptomics (p=0.55); surfactant metabolism significant in proteomics (p=0.002)
- Bootstrap y-variance explained for metabolomics falls in the 6-17% range across latent variables over 100 resamples
- Output table contains exactly the columns pathway name, omics view, MB-VIP score, and permutation p-value

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples with missing entire omics rows must be discarded
- Un-annotated molecules cannot be incorporated into the pathway-based model, causing information loss
- Pathway coverage (proportion of observable molecules per pathway) is an inherent bottleneck, particularly for metabolomics
- Comparing molecular vs pathway-level models is challenging because the number of differentially abundant molecules per pathway is unknown and pathway definitions vary between databases

## Evidence

- [intro] Multi-View: apply a multi-block partial least regression (MB-PLS) model to pathway-transformed omics datasets: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
- [supplementary] Nested CV tuning of latent variables: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [methods] Shared pathway-based latent space: "Multi-View model: Integrates multiple omics datasets using a shared pathway-based latent space"
- [discussion] Same-individual requirement: "PathIntegrate requires all input omics to be measured on the same individuals. This means samples from individuals without data on all omics will have to be discarded"
- [readme] ssPA SciKitLearn compatibility: [evidence span withheld — anchor status: external_doc]
