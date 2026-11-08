---
name: multi-omics-predictive-modelling
description: Use when you have two or more omics data matrices measured on the same
  individuals (e.g. COPDgene plasma proteomics/metabolomics or the Su et al. 2020
  COVID-19 data) and need supervised prediction plus pathway-level interpretation
  of the outcome.
when_to_use_negative:
- Many molecules of interest are un-annotated to pathways — un-annotated molecules
  cannot currently be incorporated into pathway-based models, causing information
  loss (a molecular-level MB-PLS on raw abundances should be compared instead)
- Input omics are not measured on the same individuals — PathIntegrate does not support
  entire rows of missing data, so samples without all omics must be discarded
- The goal is conventional differential-enrichment analysis (ORA/GSEA/MultiGSEA/IMPaLA)
  rather than predictive modelling; no direct comparison to these was performed in
  the article
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0157
tools:
- name: PathIntegrate
  role: Core framework providing Single-View and Multi-View (MB-PLS) supervised models
    on pathway-transformed multi-omics data
  repo: https://github.com/cwieder/PathIntegrate
- name: sspa
  role: Performs the single-sample pathway analysis (e.g. sspa_kpca) that converts
    molecular abundance matrices into pathway score matrices, and downloads/maps pathway
    databases
  repo: https://github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Compatibility layer for model optimisation/evaluation (Pipeline, GridSearchCV,
    cross-validation of latent-variable tuning)
- name: DIABLO
  role: Benchmark comparator with a similar predictive framework but no pathway transformation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:e60b73542b9dbe701533bb54d1fa0b43fd1fc0db7464443c058f23de45d4e660
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-omics-predictive-modelling

## Summary

Predictive multi-omics integration at the pathway level using the PathIntegrate framework: molecules are transformed to pathway scores via single-sample pathway analysis (ssPA), then classified/regressed with a Single-View (SKlearn-compatible) or Multi-View multi-block PLS model. It provides outcome prediction, model performance evaluation, and pathway-ranked interpretation, unlike molecule-list-based pathway analysis tools.

## When to use

Use when you have two or more omics data matrices measured on the same individuals (e.g. COPDgene plasma proteomics/metabolomics or the Su et al. 2020 COVID-19 data) and need supervised prediction plus pathway-level interpretation of the outcome.

## When NOT to use

- Many molecules of interest are un-annotated to pathways — un-annotated molecules cannot currently be incorporated into pathway-based models, causing information loss (a molecular-level MB-PLS on raw abundances should be compared instead)
- Input omics are not measured on the same individuals — PathIntegrate does not support entire rows of missing data, so samples without all omics must be discarded
- The goal is conventional differential-enrichment analysis (ORA/GSEA/MultiGSEA/IMPaLA) rather than predictive modelling; no direct comparison to these was performed in the article

## Inputs

- Preprocessed molecular-level multi-omics abundance matrices (pandas DataFrames, samples as rows, ChEBI/UniProt/gene-symbol molecule IDs as columns), all measured on the same individuals
- Outcome/phenotype labels (e.g. COPD status or COVID mild-vs-severe outcome)
- Pathway database (Reactome or KEGG, or a custom GMT file) covering the relevant molecule identifier types

## Outputs

- Cross-validated predictive performance metrics (e.g. average AUC of 0.70 +/- 0.02 on COPDgene) and tuned number of latent variables
- Multi-omics pathways ranked by contribution to outcome prediction, per-omics-layer contributions, and molecule-level importance within pathways
- Outcome class predictions for held-out samples

## How to apply

Load each omics matrix (samples as rows, molecule IDs as columns) with pandas and preprocess by removing high-missingness columns, median imputation, log2 transformation and z-score standardisation, as recommended before PathIntegrate input. Convert molecule identifiers (ChEBI, UniProt, or gene symbols) to pathway members and run the ssPA step (e.g. sspa.sspa_kpca) to transform molecular-level matrices into pathway-level matrices. Choose Single-View (a SKlearn-compatible model on the concatenated pathway score set) or Multi-View (MB-PLS with a shared pathway-based latent space) depending on whether omics-block interactions should be modelled. Tune the number of latent variables by 5-times repeated nested 5-fold cross-validation, then evaluate average cross-validated AUC (e.g. ~0.70 on the COPDgene outcome) and rank pathways by contribution to prediction. Rationale: pathway transformation brings disparate omics to a common scale and enhances sensitivity in low signal-to-noise scenarios, and PathIntegrate was benchmarked against DIABLO, which does not use pathway transformation.

## Related tools

- **PathIntegrate** (Core framework providing Single-View and Multi-View (MB-PLS) supervised models on pathway-transformed multi-omics data) — https://github.com/cwieder/PathIntegrate
- **sspa** (Performs the single-sample pathway analysis (e.g. sspa_kpca) that converts molecular abundance matrices into pathway score matrices, and downloads/maps pathway databases) — https://github.com/cwieder/py-ssPA
- **SciKitLearn** (Compatibility layer for model optimisation/evaluation (Pipeline, GridSearchCV, cross-validation of latent-variable tuning))
- **DIABLO** (Benchmark comparator with a similar predictive framework but no pathway transformation)

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- Nested cross-validated performance: expect average AUC around 0.70 +/- 0.02 for both molecular-level and pathway-level MB-PLS on COPDgene (Table 2)
- Model complexity check: the pathway-level model required fewer latent variables than the molecular-level model (4 vs 6 per Table 2 comparison)
- Schema check: the ssPA transformation output must be a pathway-level matrix whose columns correspond to pathways and is compatible with SKlearn Pipeline/GridSearchCV
- Benchmark sanity: PathIntegrate's performance should be compared against DIABLO on the same splits, since it was benchmarked as a non-pathway-transformed comparator
- Low signal-to-noise semi-synthetic data should show pathway transformation improving sensitivity, per the paper's semi-synthetic evaluation

## Limitations

- Pathway coverage and pathway annotation are inherent bottlenecks, particularly for metabolomics; reliance on current pathway knowledge is a widely accepted limitation
- Un-annotated molecules cannot be incorporated into pathway-based models, causing information loss (molecular-level models mitigate this)
- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded
- It remains challenging to compare molecular vs. pathway-level models because the number of differentially abundant molecules per pathway and pathway definitions vary between databases

## Evidence

- [intro] two supervised frameworks: "PathIntegrate consists of two supervised learning frameworks for pathway-based multi-omics integration: PathIntegrate Single-View, which produces a multi-omics pathway-transformed dataset"
- [intro] multi-view MB-PLS: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
- [supplementary] CV scheme: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [methods] shared latent space: "Multi-View model: Integrates multiple omics datasets using a shared pathway-based latent space"
- [readme] ssPA transformation: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level, and applies a predictive single-view or multi-view model"
- [readme] SKlearn compatibility: [evidence span withheld — anchor status: external_doc]
