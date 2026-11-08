---
name: multi-omics-data-integration
description: Use when you have >=2 molecular abundance matrices measured on the same
  individuals and need a predictive, pathway-level multi-omics model that ranks pathways
  and quantifies each omics layer's contribution.
when_to_use_negative:
- Input is already a pathway-level or feature table rather than molecular abundance
  matrices requiring ssPA transformation
- Omics views are not measured on the same individuals, since PathIntegrate discards
  samples without data on all omics
- Goal is conventional per-omics hypothesis testing (ORA, GSEA, MultiGSEA, IMPaLA)
  rather than predictive, pathway-level multivariate integration
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0219
tools:
- name: PathIntegrate
  role: Fits the Multi-View MB-PLS model on pathway-transformed views, computes block-wise
    MB-VIP importance, permutation p-values and bootstraps
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa
  role: Downloads Reactome human pathways and performs kPCA single-sample pathway
    transformation of each omics matrix to pathway scores
  repo: https://github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Provides compatible model optimisation and evaluation (e.g. nested cross-validation
    for latent-variable tuning)
- name: Python
  role: Runtime environment for the PathIntegrate and sspa packages
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:60c9af17e4e8d8d80c8f77050560e9b6fd0684350e612b65dd7646d026cac7ca
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-omics-data-integration

## Summary

Integrates two or more molecular abundance matrices (e.g. metabolomics, proteomics, transcriptomics) by transforming them to Reactome pathway-level scores with single-sample pathway analysis (kPCA) and fitting a multi-block PLS (MB-PLS) model, yielding pathways ranked by MB-VIP with omics-layer attribution.

## When to use

Use when you have >=2 molecular abundance matrices measured on the same individuals and need a predictive, pathway-level multi-omics model that ranks pathways and quantifies each omics layer's contribution.

## When NOT to use

- Input is already a pathway-level or feature table rather than molecular abundance matrices requiring ssPA transformation
- Omics views are not measured on the same individuals, since PathIntegrate discards samples without data on all omics
- Goal is conventional per-omics hypothesis testing (ORA, GSEA, MultiGSEA, IMPaLA) rather than predictive, pathway-level multivariate integration

## Inputs

- COPDgene Phase-2 metabolomics, proteomics and transcriptomics abundance matrices (samples as rows, molecule IDs as columns matching Reactome identifiers)
- Reactome human pathway database (GMT-style, retrieved via sspa)
- Sample-level outcome labels (e.g. COPD status)

## Outputs

- mbpls_pathway_importance.csv containing pathway name, omics view, MB-VIP score and permutation p-value
- Fitted MB-PLS Multi-View model with tuned number of latent variables and cross-validated AUC
- Per-omics y-variance explained per latent variable with 100-bootstrap stability estimates

## How to apply

Load each omics matrix with samples as rows and molecule identifiers matching the pathway database (e.g. ChEBI/UniProt/ENSEMBL for Reactome), then download Reactome human pathways via the sspa package. Transform each matrix to pathway scores using the kPCA single-sample pathway analysis (ssPA) method, bringing the disparate omics datatypes to a common scale. Fit the PathIntegrate Multi-View multi-block PLS (MB-PLS) model on the pathway-transformed views, tuning the number of latent variables with 5-times repeated nested 5-fold cross-validation and selecting the setting with best cross-validated AUC (COPDgene optimum: 4 latent variables, mean AUC 0.70). Compute block-wise MB-VIP pathway importance per omics view and assess significance by permutation testing of the MB-VIP statistic (e.g. carnitine metabolism: metabolomics p = 0.003, transcriptomics p = 0.55; surfactant metabolism: proteomics p = 0.002). Run 100 bootstrap resamples to estimate stability of y-variance explained per latent variable (metabolomics: 6-17%). Write the ranked table with pathway name, omics view, MB-VIP score and permutation p-value to mbpls_pathway_importance.csv.

## Related tools

- **PathIntegrate** (Fits the Multi-View MB-PLS model on pathway-transformed views, computes block-wise MB-VIP importance, permutation p-values and bootstraps) — https://github.com/cwieder/PathIntegrate
- **sspa** (Downloads Reactome human pathways and performs kPCA single-sample pathway transformation of each omics matrix to pathway scores) — https://github.com/cwieder/py-ssPA
- **SciKitLearn** (Provides compatible model optimisation and evaluation (e.g. nested cross-validation for latent-variable tuning))
- **Python** (Runtime environment for the PathIntegrate and sspa packages)

## Examples

```
reactome_pathways = sspa.process_reactome(organism="Homo sapiens"); kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(omics_matrix)
```

## Evaluation signals

- Model reproduces mean cross-validated AUC 0.70 with 4 latent variables on COPDgene Phase-2 data
- Carnitine metabolism attains metabolomics permutation p = 0.003 but transcriptomics p = 0.55; surfactant metabolism attains proteomics p = 0.002
- 100-bootstrap y-variance explained for the metabolomics view falls in the 6-17% range across latent variables
- Output CSV mbpls_pathway_importance.csv contains the required columns: pathway name, omics view, MB-VIP score and permutation p-value
- Each omics matrix maps to Reactome identifiers (ChEBI, UniProt or ENSEMBL) before pathway scoring

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics
- Un-annotated molecules cannot be incorporated into pathway-based models, causing information loss
- No direct comparison was performed to conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) as it was beyond the scope of the work

## Evidence

- [abstract] PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [intro] PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
- [supplementary] 5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [discussion] PathIntegrate requires all input omics to be measured on the same individuals.: "PathIntegrate requires all input omics to be measured on the same individuals."
- [readme] kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2]): [evidence span withheld — anchor status: external_doc]
