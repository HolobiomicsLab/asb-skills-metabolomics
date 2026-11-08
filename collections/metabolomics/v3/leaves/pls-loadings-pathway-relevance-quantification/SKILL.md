---
name: pls-loadings-pathway-relevance-quantification
description: Use when a fitted pathway-level multivariate model (MB-PLS, PLS, DIABLO/RGCCA)
  must be turned into ranked pathway importance scores, e.g. to detect an enriched
  target pathway or rank pathways by outcome contribution.
when_to_use_negative:
- The goal is a direct comparison against conventional ORA/GSEA-style pathway analysis
  (MultiGSEA, IMPaLA, ActivePathways) — the article states a full comparison was beyond
  its scope
- Samples are not measured on all omics layers — PathIntegrate requires all input
  omics on the same individuals and discards incomplete rows
- The analysis is at the molecular level with strong signal-to-noise — the pathway-loading
  approach is motivated by low signal-to-noise, pathway-level detection
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PathIntegrate
  role: Provides the MB-PLS Multi-View and Single-View frameworks whose VIP/MB-VIP
    statistics quantify pathway relevance
  repo: https://github.com/cwieder/PathIntegrate
- name: DIABLO
  role: Benchmark multi-omics integration model whose RGCCA component loadings (and
    sparse non-zero loadings) yield pathway relevance scores
- name: SciKitLearn
  role: Used for model optimisation and evaluation of the pathway-transformed predictive
    models
- name: py-ssPA
  role: Generates the single-sample pathway scores that feed the PLS-family models
    before loading-based relevance quantification
  repo: github.com/cwieder/py-ssPA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:54011baafd9a2669361e23173ba5647310610606284a6fa1aa449270d72f0c48
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# PLS-loadings pathway relevance quantification

## Summary

Quantifies the relevance of multi-omics pathways by converting component loadings and variable-importance statistics from fitted PLS-family models (MB-PLS in PathIntegrate Multi-View, PLS in Single-View, RGCCA in DIABLO) into pathway importance scores (VIP, MB-VIP, loading-based relevance).

## When to use

Use when a fitted pathway-level multivariate model (MB-PLS, PLS, DIABLO/RGCCA) must be turned into ranked pathway importance scores, e.g. to detect an enriched target pathway or rank pathways by outcome contribution.

## When NOT to use

- The goal is a direct comparison against conventional ORA/GSEA-style pathway analysis (MultiGSEA, IMPaLA, ActivePathways) — the article states a full comparison was beyond its scope
- Samples are not measured on all omics layers — PathIntegrate requires all input omics on the same individuals and discards incomplete rows
- The analysis is at the molecular level with strong signal-to-noise — the pathway-loading approach is motivated by low signal-to-noise, pathway-level detection

## Inputs

- Pathway-transformed multi-omics matrices (ssPA scores per omics block, e.g. kPCA-based pathway scores)
- Fitted PathIntegrate Single-View (PLS) and Multi-View (MB-PLS) models
- Fitted DIABLO / sparse DIABLO models with RGCCA component loadings
- Permutation-based null distributions of pathway importance statistics
- Outcome labels (e.g. COPD status, COVID mild vs severe) and known target pathway membership

## Outputs

- Per-pathway importance scores: VIP, MB-VIP, RGCCA loading-derived relevance, bootstrap non-zero-loading statistics
- Pathway detection confusion matrices at each effect size
- Confusion-matrix AUC per method and effect size (e.g. written to pathway_detection_auc.csv)
- Multi-omics pathways ranked by contribution to outcome prediction

## How to apply

Transform molecular-level abundance matrices into pathway-level matrices via single-sample pathway analysis (ssPA), then fit the predictive model on the pathway-transformed data. For PathIntegrate Single-View compute VIP on the pathway-transformed dataset; for PathIntegrate Multi-View compute MB-VIP from the multi-block partial least squares (MB-PLS) model that shares a pathway-based latent space across omics blocks. For DIABLO, derive pathway relevance scores from RGCCA component loadings; for the sparse L1-norm DIABLO variant use the bootstrap non-zero-loading statistic as the detection statistic. To assess pathway detection, build null distributions via permutation (10,000 permutations in the benchmark) with Benjamini-Hochberg FDR correction, form confusion matrices against the embedded target pathway, and compute confusion-matrix AUC. The resulting statistics can be exported as a table such as pathway_detection_auc.csv.

## Related tools

- **PathIntegrate** (Provides the MB-PLS Multi-View and Single-View frameworks whose VIP/MB-VIP statistics quantify pathway relevance) — https://github.com/cwieder/PathIntegrate
- **DIABLO** (Benchmark multi-omics integration model whose RGCCA component loadings (and sparse non-zero loadings) yield pathway relevance scores)
- **SciKitLearn** (Used for model optimisation and evaluation of the pathway-transformed predictive models)
- **py-ssPA** (Generates the single-sample pathway scores that feed the PLS-family models before loading-based relevance quantification) — github.com/cwieder/py-ssPA

## Evaluation signals

- Pathway detection AUC ≥ 0.90 for the embedded target pathway, even at effect size 0.25 with high noise
- Null distributions built from 10,000 permutations with Benjamini-Hochberg FDR correction before thresholding importance scores
- Sparse DIABLO shows a slight improvement over standard DIABLO at high effect sizes when using the bootstrap non-zero-loading statistic
- Importance scores are computable within the SciKitLearn-compatible pipeline (fit/transform/GridSearchCV semantics)
- Ranked pathway output includes per-omics-layer and per-molecule contributions alongside pathway relevance

## Limitations

- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics
- Un-annotated molecules cannot be incorporated into pathway-based models, causing information loss
- Results depend on current pathway annotation knowledge; pathway definitions and sizes vary between databases (e.g. KEGG vs Reactome), complicating molecular-vs-pathway model comparison
- Lack of appropriate benchmarking data (n ≥ 1000, multiple high-quality omics views, validated signals) makes absolute performance assessment difficult

## Evidence

- [abstract] PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [intro] MB-PLS multi-view model basis: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
- [intro] DIABLO benchmark framing: "PathIntegrate was benchmarked against DIABLO [11], a popular multi-omics integration tool with a similar predictive framework, but which does not use pathway transformation."
- [readme] Model output description: "Model outputs include multi-omics pathways ranked by their contribution to the outcome prediction, the contribution of each omics layer, and the importance of each molecule in a pathway."
- [other] Permutation-based detection workflow: [evidence span withheld — anchor status: not_found]
- [results] Detection performance finding: [evidence span withheld — anchor status: not_found]
