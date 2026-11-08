---
name: bootstrap-loading-stability-statistics
description: Use when benchmarking whether a multivariate multi-omics model (DIABLO,
  sparse DIABLO, or PathIntegrate VIP/MB-VIP) can detect a known enriched target pathway,
  especially in semi-synthetic simulations with varying effect sizes and noise.
when_to_use_negative:
- When the goal is biological interpretation of real (non-simulated) multi-omics data
  rather than validating a method's pathway-detection power with a known ground-truth
  target pathway
- When a direct comparison to conventional pathway analysis approaches (ORA, GSEA,
  MultiGSEA, IMPaLA) is required — the authors state this was beyond the scope of
  the work
- When samples from individuals lack data on all omics layers, since PathIntegrate-type
  models require complete multi-omics per individual
tools:
- name: DIABLO
  role: Provides the standard and sparse L1-norm multi-omics models whose RGCCA component
    loadings yield pathway relevance scores and bootstrap non-zero-loading statistics
    for pathway detection
- name: PathIntegrate
  role: Benchmarked framework producing VIP (Single-View) and MB-VIP (Multi-View/MB-PLS)
    pathway importance scores evaluated against permutation nulls in the same detection
    task
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Backend used for model fitting/evaluation of the pathway-transformed predictive
    models
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:ce7316395e605d245d812febba118532e0d5cce741eb2330f3e0b37fd3749d86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# bootstrap-loading-stability-statistics

## Summary

A pathway-detection evaluation approach in which a bootstrap non-zero-loading statistic from a sparse L1-norm DIABLO model (and permutation-derived null distributions for VIP/MB-VIP pathway importance scores) is used to compute confusion-matrix AUCs for detecting an enriched target pathway in semi-synthetic multi-omics data.

## When to use

Use when benchmarking whether a multivariate multi-omics model (DIABLO, sparse DIABLO, or PathIntegrate VIP/MB-VIP) can detect a known enriched target pathway, especially in semi-synthetic simulations with varying effect sizes and noise.

## When NOT to use

- When the goal is biological interpretation of real (non-simulated) multi-omics data rather than validating a method's pathway-detection power with a known ground-truth target pathway
- When a direct comparison to conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) is required — the authors state this was beyond the scope of the work
- When samples from individuals lack data on all omics layers, since PathIntegrate-type models require complete multi-omics per individual

## Inputs

- Semi-synthetic multi-omics simulation datasets with an embedded target pathway signal at multiple effect sizes (e.g. 0.25 and higher)
- Pathway definitions (e.g. KEGG, Reactome human pathway databases)
- Fitted sparse L1-norm DIABLO model with bootstrap resamples
- Permutation-based null distributions of VIP/MB-VIP pathway importance scores (10,000 permutations)

## Outputs

- Bootstrap non-zero-loading statistic per pathway for the sparse DIABLO model
- Pathway detection confusion matrices per method and effect size
- Confusion-matrix AUC per method per effect size
- Results table pathway_detection_auc.csv

## How to apply

Embed a known target pathway signal into semi-synthetic multi-omics simulation data across a grid of effect sizes (down to 0.25). Fit the sparse L1-norm DIABLO variant and compute, for each pathway, the bootstrap non-zero-loading statistic (frequency of non-zero RGCCA component loadings across bootstrap resamples) as the pathway detection statistic. In parallel, fit PathIntegrate Multi-View (MB-PLS) and Single-View models, compute MB-VIP/VIP pathway importance scores, and build detection confusion matrices using permutation-based null distributions with 10,000 permutations and Benjamini-Hochberg FDR correction. Derive DIABLO pathway relevance scores from RGCCA component loadings and apply the same detection criterion. Compute confusion-matrix AUCs per method and effect size, then write the results to pathway_detection_auc.csv. The expected behaviour is detection AUC ≥ 0.90 even at effect size 0.25, with sparse DIABLO showing a slight improvement over standard DIABLO at high effect sizes.

## Related tools

- **DIABLO** (Provides the standard and sparse L1-norm multi-omics models whose RGCCA component loadings yield pathway relevance scores and bootstrap non-zero-loading statistics for pathway detection)
- **PathIntegrate** (Benchmarked framework producing VIP (Single-View) and MB-VIP (Multi-View/MB-PLS) pathway importance scores evaluated against permutation nulls in the same detection task) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Backend used for model fitting/evaluation of the pathway-transformed predictive models)

## Evaluation signals

- Detection confusion-matrix AUC ≥ 0.90 for the target enriched pathway even at effect size = 0.25 (low effect size, high noise)
- Null distributions for VIP/MB-VIP pathway statistics are built from exactly 10,000 permutations with Benjamini-Hochberg FDR correction applied
- Sparse L1-norm DIABLO shows a slight improvement over standard DIABLO at high effect sizes
- A per-method, per-effect-size AUC table is produced and written to pathway_detection_auc.csv

## Limitations

- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck of pathway-based analyses, particularly for metabolomics
- It remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant at any one time, and pathway definitions and sizes vary between databases
- No appropriate benchmarking dataset with known validated pathway-level signals was available, motivating the semi-synthetic simulation design rather than real-data ground truth

## Evidence

- [abstract] low-signal benefit of pathway grouping: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [abstract] precise identification at low effect sizes: "as well as the ability of PathIntegrate to precisely identify important pathways at low effect sizes"
- [intro] DIABLO as benchmark: "PathIntegrate was benchmarked against DIABLO [11], a popular multi-omics integration tool with a similar predictive framework"
- [discussion] scope limit vs conventional pathway analysis: "A full comparison of PathIntegrate, a pathway-based predictive model, to conventional pathway analysis approaches, such as ORA [25], GSEA [26], or integrated pathway analysis"
- [discussion] molecular vs pathway-level comparison challenge: "it remains a challenge to compare molecular vs. pathway-level models, as it is unknown how many molecules in a pathway are differentially abundant"
