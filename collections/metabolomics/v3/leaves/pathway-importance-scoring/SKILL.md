---
name: pathway-importance-scoring
description: Use when you have fitted a pathway-level multi-omics model (PathIntegrate
  MB-PLS/Single-View or DIABLO on ssPA-transformed data) and need ranked pathway importance
  statistics with permutation-tested significance or detection AUCs.
when_to_use_negative:
- When you need molecule-level (not pathway-level) feature importance — this skill
  ranks multi-omics pathways, not individual molecules
- When the pathway coverage of your omics data is too low for a pathway-level analysis
  (a stated bottleneck, particularly for metabolomics)
- When samples lack measurements across all omics views — PathIntegrate requires all
  input omics measured on the same individuals, so such samples must be discarded
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: PathIntegrate
  role: Fits the pathway-based multi-view (MB-PLS) and single-view supervised models
    from which MB-VIP/VIP pathway importance scores are computed
  repo: https://github.com/cwieder/PathIntegrate
- name: py-ssPA
  role: Generates the pathway-level score matrices that the importance statistics
    are computed on; ssPA methods expose fit/transform for SciKitLearn pipelines
  repo: https://github.com/cwieder/py-ssPA
- name: DIABLO
  role: 'Benchmark model: pathway relevance scores derived from RGCCA component loadings
    and, for the sparse L1-norm variant, bootstrap non-zero-loading statistics'
- name: SciKitLearn
  role: Underlying model optimisation and evaluation framework for PathIntegrate Single-View
    and pipeline integration
- name: Python
  role: Implementation language for the scoring and permutation-testing workflow (PathIntegrate_scripts
    for benchmarking)
  repo: https://github.com/cwieder/PathIntegrate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:92dfef9d31f6b768f3b07dd53814dbf7b0cf0c874f71095d1ff8a3a1545cc2fd
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pathway-importance-scoring

## Summary

Computes and validates pathway-level importance statistics from fitted multi-omics predictive models (MB-VIP/VIP for PathIntegrate MB-PLS/Single-View, RGCCA loadings and bootstrap non-zero loadings for DIABLO), using permutation-based null distributions to score pathway detection by confusion-matrix AUC.

## When to use

Use when you have fitted a pathway-level multi-omics model (PathIntegrate MB-PLS/Single-View or DIABLO on ssPA-transformed data) and need ranked pathway importance statistics with permutation-tested significance or detection AUCs.

## When NOT to use

- When you need molecule-level (not pathway-level) feature importance — this skill ranks multi-omics pathways, not individual molecules
- When the pathway coverage of your omics data is too low for a pathway-level analysis (a stated bottleneck, particularly for metabolomics)
- When samples lack measurements across all omics views — PathIntegrate requires all input omics measured on the same individuals, so such samples must be discarded

## Inputs

- Pathway-transformed multi-omics score matrices (ssPA output from molecular abundance data)
- Sample outcome labels (e.g. COPD status, COVID-19 mild vs severe)
- Pathway database definitions (KEGG or Reactome GMT for metabolites/proteins)

## Outputs

- Pathway importance scores (MB-VIP for PathIntegrate Multi-View, VIP for Single-View)
- Permutation-based null distributions and BH-FDR-corrected pathway detection confusion matrices
- Per-method confusion-matrix AUC table (e.g. pathway_detection_auc.csv)

## How to apply

Transform molecular-level omics matrices to pathway-level scores with single-sample pathway analysis (ssPA), then fit the predictive model: PathIntegrate Multi-View uses an MB-PLS model on a shared pathway-based latent space, yielding MB-VIP scores; PathIntegrate Single-View yields VIP scores from a SciKitLearn-compatible model. For pathway detection assessment, build confusion matrices from permutation-based null distributions (10,000 permutations) with Benjamini-Hochberg FDR correction. For the DIABLO benchmark, derive pathway relevance scores from RGCCA component loadings under the same detection criterion, and for the sparse L1-norm DIABLO variant use the bootstrap non-zero-loading statistic. Compare methods by confusion-matrix AUC per effect size; the expected behaviour is detection AUC >= 0.90 at effect size 0.25 (low signal-to-noise), with sparse DIABLO improving slightly over standard DIABLO at high effect sizes. Write per-method, per-effect-size AUCs to a results table (e.g. pathway_detection_auc.csv).

## Related tools

- **PathIntegrate** (Fits the pathway-based multi-view (MB-PLS) and single-view supervised models from which MB-VIP/VIP pathway importance scores are computed) — https://github.com/cwieder/PathIntegrate
- **py-ssPA** (Generates the pathway-level score matrices that the importance statistics are computed on; ssPA methods expose fit/transform for SciKitLearn pipelines) — https://github.com/cwieder/py-ssPA
- **DIABLO** (Benchmark model: pathway relevance scores derived from RGCCA component loadings and, for the sparse L1-norm variant, bootstrap non-zero-loading statistics)
- **SciKitLearn** (Underlying model optimisation and evaluation framework for PathIntegrate Single-View and pipeline integration)
- **Python** (Implementation language for the scoring and permutation-testing workflow (PathIntegrate_scripts for benchmarking)) — https://github.com/cwieder/PathIntegrate

## Examples

```
kpca_scores = sspa.sspa_kpca(reactome_pathways, min_entity=2).fit_transform(covid_data_processed.iloc[:, :-2])
```

## Evaluation signals

- Detection confusion-matrix AUC >= 0.90 for the enriched target pathway at effect size 0.25 (low effect size, high noise)
- Pathway null distributions built from exactly 10,000 permutations with Benjamini-Hochberg FDR correction
- Sparse DIABLO shows a slight improvement over standard DIABLO at high effect sizes; PathIntegrate and DIABLO pathway models remain comparable
- Importance statistics are computed on pathway-transformed matrices (ssPA), not raw molecular features — verify the pipeline output dimensions match the number of pathways
- Per-method AUC values recorded for every tested effect size in the results CSV

## Limitations

- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics
- Reliance on current pathway annotation knowledge; un-annotated molecules cause information loss
- Detection power for low effect sizes depends on the grouping of molecules into pathways; the AUC >= 0.90 result was established on semi-synthetic simulations, not validated real benchmarks
- A full comparison to conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) was beyond the scope of the work

## Evidence

- [abstract] Pathway detection performance via permutation testing: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [intro] MB-VIP / VIP from Multi-View and Single-View frameworks: "PathIntegrate consists of two supervised learning frameworks for pathway-based multi-omics integration: PathIntegrate Single-View, which produces a multi-omics pathway-transformed dataset"
- [intro] SciKitLearn compatibility of the scoring pipeline: "designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation."
- [supplementary] Permutation nulls on semi-synthetic COPDgene data: "Pathway transformation enhances sensitivity to low signal-to-noise signals (COPDgene semi synthetic data)"
- [readme] Ranked pathway importance as a model output: "Model outputs include multi-omics pathways ranked by their contribution to the outcome prediction, the contribution of each omics layer, and the importance of each molecule in a pathway."
- [readme] ssPA fit/transform for pathway scoring: [evidence span withheld — anchor status: external_doc]
