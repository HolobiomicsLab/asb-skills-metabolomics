---
name: spearman-correlation-heatmap-clustering
description: Use when several normalization variants (e.g., tGAM, rGAM, rLOESS, QC-RLSC,
  QC-RSC, SERFF, TIGER) have been applied to the same dataset and you need to visualize
  how similarly they rank metabolites by differential abundance signal.
when_to_use_negative:
- Only one normalization variant is available — there is no pairwise structure to
  correlate or cluster
- The goal is to quantify differential-abundance-metabolite (DAM) overlap percentages
  rather than p-value ranking similarity — that is a distinct set-based comparison,
  not a correlation analysis
- Phenotype/contrast labels (e.g., biological sex) are unavailable, so per-method
  DAM p-values cannot be computed in the first place
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Language used to compute Spearman correlations, 1-minus-correlation Euclidean
    distances, complete-linkage clustering, and the heatmap
  repo: https://github.com/UGent-LIMET/Metanorm
- name: Metanorm R package
  role: Produces the per-method normalized datasets whose downstream p-values feed
    the correlation/heatmap comparison
  repo: https://github.com/UGent-LIMET/Metanorm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c06841
    title: Robust Metabolomics Data Normalization across Scales and Experimental Designs
  source_content_hash: sha256:994c77fac0a54d1e805ed69322fd055faa9619ddaae099ad0583eb9b9bb2128f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/UGent-LIMET/Metanorm
  tools:
  - Metanorm
  - Metanorm R package
  - R
  - SERFF
  - TIGER
  - fANCOVA
  - mgcv
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Spearman correlation heatmap with complete-linkage clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between normalization methods by computing pairwise Spearman correlations of their downstream differential-abundance p-values (e.g., two-sided Wilcoxon rank sum tests at p < 0.05 for biological sex), converting 1-minus-correlation to Euclidean distances, and rendering a heatmap with complete-linkage hierarchical clustering.

## When to use

Use when several normalization variants (e.g., tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER) have been applied to the same dataset and you need to visualize how similarly they rank metabolites by differential abundance signal.

## When NOT to use

- Only one normalization variant is available — there is no pairwise structure to correlate or cluster
- The goal is to quantify differential-abundance-metabolite (DAM) overlap percentages rather than p-value ranking similarity — that is a distinct set-based comparison, not a correlation analysis
- Phenotype/contrast labels (e.g., biological sex) are unavailable, so per-method DAM p-values cannot be computed in the first place

## Inputs

- Matrix of per-metabolite differential abundance p-values, one column per normalization method (from Wilcoxon rank sum tests on each normalized dataset)
- List/labels of normalization methods compared (tGAM, rGAM, rLOESS, QC-RLSC, QC-RSC, SERFF, TIGER)
- Optionally, the underlying normalized intensity matrices for re-deriving p-values

## Outputs

- Pairwise Spearman correlation matrix between normalization methods
- Pairwise Euclidean distances of one-minus-correlation vectors between methods
- Heatmap with complete-linkage hierarchical clustering of methods by p-value similarity
- CSV of comparison metrics (e.g., environage_dam_comparison.csv with overlap percentages and correlation-derived similarities)

## How to apply

After running each normalization (e.g., Metanorm's tGAM/rGAM/rLOESS/QC-RLSC/QC-RSC in batchwise mode, plus SERFF and TIGER outputs) and computing per-metabolite p-values for the contrast of interest (e.g., two-sided Wilcoxon rank sum tests for biological sex in the ENVIRONAGE urine dataset of 393 runs, 269 metabolites, 6 batches), assemble the p-value vectors into a matrix. Compute all pairwise Spearman correlations between methods' p-value vectors; the rank-based Spearman statistic is appropriate because p-value distributions may be non-normal and only concordance of metabolite rankings matters. Transform correlations into dissimilarities via 1 - rho, treat these as Euclidean distances between method vectors, and perform complete-linkage hierarchical clustering to build the heatmap. Complete linkage is the paper-specified choice, and the expected structure is that robust methods (rLOESS, rGAM, tGAM) cluster tightly while QC-RLSC and, to a lesser degree, SERFF and TIGER diverge.

## Related tools

- **R** (Language used to compute Spearman correlations, 1-minus-correlation Euclidean distances, complete-linkage clustering, and the heatmap) — https://github.com/UGent-LIMET/Metanorm
- **Metanorm R package** (Produces the per-method normalized datasets whose downstream p-values feed the correlation/heatmap comparison) — https://github.com/UGent-LIMET/Metanorm

## Examples

```
rho <- cor(pval_mat, method="spearman"); D <- as.dist(1 - rho); hc <- hclust(D, method="complete"); heatmap(rho, Rowv=as.dendrogram(hc), Colv=as.dendrogram(hc))
```

## Evaluation signals

- Spearman correlation matrix is symmetric with diagonal exactly 1 and all values in [-1, 1]
- Derived distance matrix has zero diagonal and non-negative off-diagonal entries
- Dendrogram from complete-linkage clustering places the robust methods (rLOESS, rGAM, tGAM) on a common branch, consistent with their reported 74–88% DAM overlap
- QC-RLSC appears most distant from the robust cluster, consistent with its reported 47–51% DAM overlap
- Heatmap is generated and saved alongside the overlap/correlation table (e.g., environage_dam_comparison.csv)

## Limitations

- p-value similarity reflects ranking concordance only and does not by itself establish which normalization recovers the true signal; the paper warns against proxies like 'an increased number of differentially abundant metabolites' for judging normalization quality
- Spearman correlation of p-values is sensitive to the downstream test choice (e.g., two-sided Wilcoxon rank sum tests at p < 0.05); a different test or significance cutoff changes the clustering
- The article notes that benchmarking multivariate methods such as SERFF and TIGER alongside metabolite-level methods 'would allow one to also benchmark multivariate methods' but was not done within the paper itself, so correlations involving them should be interpreted cautiously

## Evidence

- [results] Robust methods agreement finding: [evidence span withheld — anchor status: not_found]
