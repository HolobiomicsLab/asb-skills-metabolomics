---
name: confusion-matrix-auc-benchmark-evaluation
description: Use when semi-synthetic multi-omics data contain a known enriched target
  pathway and you need a quantitative detection benchmark (confusion-matrix AUC) over
  pathway importance statistics across effect sizes.
when_to_use_negative:
- Real (non-synthetic) datasets with no validated ground-truth target pathway — no
  known positives/negatives means a confusion matrix and AUC cannot be defined
- Comparing methods by qualitative pathway ranking or enrichment p-values only, rather
  than a thresholded detection statistic with a null distribution
- Evaluating molecular-level (feature-level) prediction performance instead of pathway-level
  detection of the enriched pathway
tools:
- name: PathIntegrate
  role: Fits Single-View and Multi-View (MB-PLS) models to produce MB-VIP/VIP pathway
    importance scores used as detection statistics
  repo: https://github.com/cwieder/PathIntegrate
- name: DIABLO
  role: Benchmarked multi-omics integration model whose RGCCA component loadings provide
    pathway relevance scores; sparse L1-norm variant supplies the bootstrap non-zero-loading
    statistic
- name: SciKitLearn
  role: Compatible model optimisation/evaluation framework used to fit models and
    compute confusion-matrix AUCs
- name: PathIntegrate_scripts
  role: Benchmarking scripts replicating the semi-synthetic detection-AUC experiments
    on an HPC cluster
  repo: https://github.com/cwieder/PathIntegrate_scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:12a6e724830b57e2177e88536cec34ce2022b3864600837a37e742af3dd0c1ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# confusion-matrix-auc-benchmark-evaluation

## Summary

Evaluates how accurately pathway-based multi-omics models (PathIntegrate MB-VIP/VIP, DIABLO, sparse DIABLO) detect an embedded target pathway by building confusion matrices from pathway importance statistics and computing detection AUCs.

## When to use

Use when semi-synthetic multi-omics data contain a known enriched target pathway and you need a quantitative detection benchmark (confusion-matrix AUC) over pathway importance statistics across effect sizes.

## When NOT to use

- Real (non-synthetic) datasets with no validated ground-truth target pathway — no known positives/negatives means a confusion matrix and AUC cannot be defined
- Comparing methods by qualitative pathway ranking or enrichment p-values only, rather than a thresholded detection statistic with a null distribution
- Evaluating molecular-level (feature-level) prediction performance instead of pathway-level detection of the enriched pathway

## Inputs

- Semi-synthetic multi-omics simulation datasets with an embedded target pathway signal across multiple effect sizes (including effect size = 0.25)
- Pathway importance statistics: MB-VIP/VIP scores (MB-PLS), RGCCA component loadings (DIABLO), bootstrap non-zero-loading statistics (sparse DIABLO)
- Ground-truth labels of the target enriched pathway per simulation replicate

## Outputs

- Permutation-based null distributions (10,000 permutations) of each pathway detection statistic with Benjamini-Hochberg FDR correction
- Pathway detection confusion matrices per method and effect size
- Confusion-matrix AUC table written to pathway_detection_auc.csv

## How to apply

Fit PathIntegrate Multi-View (MB-PLS) and Single-View models and compute MB-VIP/VIP pathway importance scores; fit DIABLO and derive pathway relevance scores from RGCCA component loadings; fit the sparse L1-norm DIABLO variant and compute a bootstrap non-zero-loading statistic as the detection statistic. For each statistic, build a permutation-based null distribution (10,000 permutations) with Benjamini-Hochberg FDR correction to define the detection criterion, then construct confusion matrices of target-pathway detected vs not detected. Compute confusion-matrix AUCs for every method at every effect size, verifying that detection AUC stays >= 0.90 even at the lowest effect size (0.25), with sparse DIABLO slightly improving on standard DIABLO at high effect sizes. Write the results table to pathway_detection_auc.csv.

## Related tools

- **PathIntegrate** (Fits Single-View and Multi-View (MB-PLS) models to produce MB-VIP/VIP pathway importance scores used as detection statistics) — https://github.com/cwieder/PathIntegrate
- **DIABLO** (Benchmarked multi-omics integration model whose RGCCA component loadings provide pathway relevance scores; sparse L1-norm variant supplies the bootstrap non-zero-loading statistic)
- **SciKitLearn** (Compatible model optimisation/evaluation framework used to fit models and compute confusion-matrix AUCs)
- **PathIntegrate_scripts** (Benchmarking scripts replicating the semi-synthetic detection-AUC experiments on an HPC cluster) — https://github.com/cwieder/PathIntegrate_scripts

## Examples

```
from sklearn.metrics import roc_auc_score; auc = roc_auc_score(y_true_pathway_labels, pathway_importance_scores)
```

## Evaluation signals

- Detection AUC for PathIntegrate and DIABLO pathway models is >= 0.90 even at effect size 0.25 (low signal, high noise)
- Null distributions contain exactly 10,000 permutations and thresholds are Benjamini-Hochberg FDR corrected
- Sparse DIABLO shows a slight improvement over standard DIABLO at high effect sizes, matching the reported behaviour
- Output file pathway_detection_auc.csv contains one AUC per method per effect size with no missing entries
- Confusion matrices are built on the same detection criterion across all methods so AUCs are directly comparable

## Limitations

- The benchmark depends on semi-synthetic data; the article notes a key drawback being the lack of appropriate real benchmarking data for multi-omics integration methods
- A valid ideal benchmark requires two or more high-quality omics views, large sample size (n >= 1000), and validated molecular/pathway-level signals
- Pathway coverage and annotation bottlenecks mean un-annotated molecules cannot enter pathway-based models, which affects detection sensitivity

## Evidence

- [results] PathIntegrate and DIABLO pathway models detected the target enriched pathway with high AUC (>=0.90) even in low effect size, high noise scenarios (effect size = 0.25).: [evidence span withheld — anchor status: not_found]
- [abstract] Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [intro] PathIntegrate was benchmarked against DIABLO, a popular multi-omics integration tool with a similar predictive framework: "PathIntegrate was benchmarked against DIABLO [11], a popular multi-omics integration tool with a similar predictive framework, but which does not use pathway transformation."
- [discussion] Ideally, a benchmarking dataset would contain two or more high-quality omics views, a large sample size (n ≥ 1000), and known biological signals at the molecular and pathway level: "Ideally, a benchmarking dataset would contain two or more high-quality omics views, a large sample size (n ≥ 1000), and known biological signals at the molecular and pathway level validated by"
- [readme] This repository contains benchmarking and application scripts for the PathIntegrate manuscript.: [evidence span withheld — anchor status: external_doc]
