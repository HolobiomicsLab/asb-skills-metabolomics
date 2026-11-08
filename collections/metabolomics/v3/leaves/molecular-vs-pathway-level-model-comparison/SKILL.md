---
name: molecular-vs-pathway-level-model-comparison
description: Use when you must decide whether pathway transformation before multi-omics
  integration (ssPA + MB-PLS) degrades predictive performance relative to modelling
  raw molecule abundances, e.g. on COPDgene plasma metabolomics/proteomics/transcriptomics.
when_to_use_negative:
- Samples from individuals without data on all omics views are present — PathIntegrate
  requires all input omics measured on the same individuals and discards incomplete
  samples, so the comparison cannot be run fairly
- The goal is pathway-level interpretation of associated biological processes rather
  than an explicit predictive-performance comparison of the two modelling levels
- The question calls for comparison against conventional pathway analysis (ORA, GSEA,
  MultiGSEA, IMPaLA) — the authors state no direct comparison to these was performed
  as it was beyond the scope of the work
edam_operation: http://edamontology.org/operation_2945
tools:
- name: PathIntegrate
  role: Provides the Multi-View MB-PLS framework used for both arms — fitted once
    on raw molecule abundances (molecular-level, skipping ssPA) and once on ssPA pathway-transformed
    data (pathway-level)
  repo: https://github.com/cwieder/PathIntegrate
- name: py-ssPA
  role: Computes the single-sample pathway (ssPA, e.g. kPCA) scores used for the pathway-level
    arm, with sklearn-compatible fit/transform so it can sit in the cross-validated
    pipeline
  repo: github.com/cwieder/py-ssPA
- name: SciKitLearn
  role: Compatibility layer for model optimisation and evaluation — nested cross-validation,
    GridSearchCV-style latent-variable tuning, and AUC scoring of the MB-PLS classifiers
- name: PathIntegrate_scripts
  role: Benchmarking and application scripts for the manuscript; the COPDgene comparison
    requires COPDgene consortium data access (phs000179.v6.p2, phs000765.v3.p2, Metabolomics
    Workbench PR000907)
  repo: cwieder/PathIntegrate_scripts
- name: Python
  role: Language environment (pandas for loading/log-transform/standardisation, scripts
    run in Jupyter notebooks for applications)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:57d3180f0ddb9b5f2e65fb05a3e1fa1d5d8cbfe29e38990f34b382670dd0382a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  - py-ssPA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# molecular-vs-pathway-level-model-comparison

## Summary

A benchmarking skill for quantifying whether pathway-based multi-omics modelling (ssPA-transformed MB-PLS in PathIntegrate Multi-View) trades off predictive performance against a molecular-level MB-PLS model fitted on raw molecule abundances, using COPDgene plasma multi-omics as the reference experiment.

## When to use

Use when you must decide whether pathway transformation before multi-omics integration (ssPA + MB-PLS) degrades predictive performance relative to modelling raw molecule abundances, e.g. on COPDgene plasma metabolomics/proteomics/transcriptomics.

## When NOT to use

- Samples from individuals without data on all omics views are present — PathIntegrate requires all input omics measured on the same individuals and discards incomplete samples, so the comparison cannot be run fairly
- The goal is pathway-level interpretation of associated biological processes rather than an explicit predictive-performance comparison of the two modelling levels
- The question calls for comparison against conventional pathway analysis (ORA, GSEA, MultiGSEA, IMPaLA) — the authors state no direct comparison to these was performed as it was beyond the scope of the work

## Inputs

- COPDgene plasma multi-omics data matrices (samples as rows, molecule IDs as columns) with outcome labels (e.g. COPD status)
- Reactome or KEGG multi-omics pathway databases (ChEBI/UniProt/gene-symbol mapped) for the pathway-level arm
- Trained-model configuration for PathIntegrate Multi-View MB-PLS (number of latent variables, nested-CV scheme)

## Outputs

- copdgene_molecular_vs_pathway_mbpls_performance.csv — comparison table with average cross-validated AUC and optimal number of latent variables for molecular-level vs pathway-level MB-PLS
- Fitted molecular-level and pathway-level Multi-View MB-PLS models with their tuned latent-variable counts

## How to apply

Load the COPDgene plasma multi-omics matrices (samples as rows, molecule IDs such as ChEBI/UniProt as columns) with pandas, log-transform and z-score standardise as recommended before PathIntegrate input. Fit the molecular-level arm: a PathIntegrate Multi-View MB-PLS classifier on raw molecule abundances, deliberately skipping the ssPA pathway-scoring step so molecules not annotated to any pathway are retained. Fit the pathway-level arm on ssPA (e.g. kPCA) pathway-transformed data, where pathway scores are computed with fit/transform-compatible sspa methods. Tune the number of latent variables for both arms identically by 5-times repeated nested 5-fold cross-validation, as used for PathIntegrate Multi-View parameter tuning. Record average cross-validated AUC and the optimal latent-variable count for each model (expected result per Table 2: both ~0.70 ± 0.02 AUC, with the molecular-level model needing more latent variables — 6 vs 4 for the pathway-level model), then write the comparison table to copdgene_molecular_vs_pathway_mbpls_performance.csv.

## Related tools

- **PathIntegrate** (Provides the Multi-View MB-PLS framework used for both arms — fitted once on raw molecule abundances (molecular-level, skipping ssPA) and once on ssPA pathway-transformed data (pathway-level)) — https://github.com/cwieder/PathIntegrate
- **py-ssPA** (Computes the single-sample pathway (ssPA, e.g. kPCA) scores used for the pathway-level arm, with sklearn-compatible fit/transform so it can sit in the cross-validated pipeline) — github.com/cwieder/py-ssPA
- **SciKitLearn** (Compatibility layer for model optimisation and evaluation — nested cross-validation, GridSearchCV-style latent-variable tuning, and AUC scoring of the MB-PLS classifiers)
- **PathIntegrate_scripts** (Benchmarking and application scripts for the manuscript; the COPDgene comparison requires COPDgene consortium data access (phs000179.v6.p2, phs000765.v3.p2, Metabolomics Workbench PR000907)) — cwieder/PathIntegrate_scripts
- **Python** (Language environment (pandas for loading/log-transform/standardisation, scripts run in Jupyter notebooks for applications))

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])  # pathway-level arm input; molecular-level arm uses the raw standardised matrix instead
```

## Evaluation signals

- Average cross-validated AUC of the molecular-level and pathway-level MB-PLS models should match within ±0.02 (expected 0.70 for both on COPDgene, per Table 2)
- The molecular-level model should require a higher optimal number of latent variables than the pathway-level model (6 vs 4 per Table 2), i.e. a more complex model for equivalent AUC
- Both arms must use the identical tuning scheme: 5-times repeated nested 5-fold cross-validation over the number of latent variables
- The molecular-level matrix must contain strictly more columns than the pathway-level matrix — it includes molecules not annotated to any pathway, which the pathway-transformed data cannot represent
- Output CSV exists with one row per modelling level and columns for AUC and latent-variable count

## Limitations

- Un-annotated molecules cannot be incorporated into the pathway-based arm, causing information loss that partially motivates this comparison; future work is needed to include them
- Comparing molecular vs pathway-level models is inherently challenging because the number of differentially abundant molecules per pathway is unknown and pathway definitions/sizes vary between databases (e.g. KEGG vs Reactome)
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, which constrains what the pathway-level model can capture
- COPDgene multi-omics data require consortium/dbGaP/Metabolomics Workbench access approval, so this comparison cannot be run on open data alone (COVID-19 Su et al. 2020 analyses are the openly reproducible alternative)

## Evidence

- [abstract] molecular-level vs pathway-level benchmark design: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [intro] Multi-View MB-PLS is the model class compared: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
- [supplementary] cross-validation scheme for latent-variable tuning: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [discussion] difficulty of the molecular vs pathway-level comparison: "it remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant at any one time"
- [discussion] un-annotated molecules drive information loss: "future work should focus on how un-annotated molecules could be incorporated into pathway-based models, to minimise information loss"
- [readme] ssPA methods are sklearn-pipeline compatible for both arms: [evidence span withheld — anchor status: external_doc]
