---
name: spearman-correlation-edge-filtering
description: Use when building omics co-occurrence or multi-omics networks from a
  feature (species/metabolite/gene) abundance table in MetaNet, and you need defensible
  correlation thresholds instead of an arbitrary similarity cutoff before downstream
  topology, module, or keystone analyses.
when_to_use_negative:
- You only want raw pairwise similarity for exploratory heatmaps or clustering without
  building a filtered graph — c_net_calculate alone suffices and c_net_build's pruning
  is unnecessary.
- The relationship of interest is non-linear or indirect; the article's discussion
  notes correlation-based network construction may overlook such associations.
- Your data are already a pre-built graph or an edge list — threshold selection applies
  to abundance matrices before network construction, not to existing networks.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_1317
- http://edamontology.org/topic_0091
tools:
- name: MetaNet
  role: Provides c_net_calculate() (Spearman correlation with p-value adjustment)
    and c_net_build() (threshold-based edge filtering and singleton removal) used
    to construct and prune the co-occurrence network.
  repo: https://github.com/Asa12138/MetaNet
- name: R
  role: Runtime environment; MetaNet is an R package requiring R >= 4.0 and built
    on igraph.
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: igraph
  role: Underlying graph data structure on which MetaNet's filtered network objects
    and downstream topology functions operate.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:a126cf17d2ca3fe06660b026a46ebb6d4776977499bbf88190c70824aa7c0b5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Asa12138/MetaNet
  tools:
  - MetaNet
  - R
  - ggraph
  - igraph
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_1317
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spearman-correlation-edge-filtering

## Summary

Prunes a MetaNet co-occurrence network to statistically robust edges by keeping only Spearman correlations with |ρ| above a chosen cutoff (e.g. 0.6–0.65) and Benjamini–Hochberg-adjusted p-values below 0.05, so the retained graph reflects strong, significant co-occurrence rather than arbitrary similarity.

## When to use

Use when building omics co-occurrence or multi-omics networks from a feature (species/metabolite/gene) abundance table in MetaNet, and you need defensible correlation thresholds instead of an arbitrary similarity cutoff before downstream topology, module, or keystone analyses.

## When NOT to use

- You only want raw pairwise similarity for exploratory heatmaps or clustering without building a filtered graph — c_net_calculate alone suffices and c_net_build's pruning is unnecessary.
- The relationship of interest is non-linear or indirect; the article's discussion notes correlation-based network construction may overlook such associations.
- Your data are already a pre-built graph or an edge list — threshold selection applies to abundance matrices before network construction, not to existing networks.

## Inputs

- Feature-by-sample abundance matrix (e.g. MetaNet `micro` species table, or MetaNet `otutab` test data), transposed so features are columns
- Chosen Spearman r threshold (e.g. 0.6 or 0.65) and p-value threshold (0.05)
- p-value multiple-testing correction method setting (Benjamini–Hochberg) supplied to c_net_calculate

## Outputs

- A MetaNet/igraph network object containing only edges with |Spearman ρ| > threshold and BH-adjusted p < 0.05
- Singleton-pruned vertex set (delete_single = TRUE), e.g. 871 species nodes retained from 914 in Case 1
- Full Spearman correlation and adjusted-p matrices usable for subsequent topology or module analyses

## How to apply

Transpose the feature table (species as columns, samples as rows) and run `c_net_calculate()` with method = 'spearman' to obtain the pairwise correlation coefficient matrix together with p-values. Apply Benjamini–Hochberg multiple-testing correction to the p-value matrix, then pass the result to `c_net_build()` with explicit thresholds — r_threshold = 0.6 (the Case 1 co-occurrence cutoff) or 0.65 in the quick-start example — p_threshold = 0.05, and use_p_adj = TRUE so the adjusted, not the raw, p-value drives edge retention. Set delete_single = TRUE to prune singleton nodes left over after filtering; in the Case 1 underwater exposome study this kept |ρ|>0.6 significant edges across 914 prevalent species and yielded an 871-node network. The rationale, per the article's introduction, is that subjective thresholding can bias network topology and downstream biological interpretation, so the cutoff and the use of p-adjustment should be stated explicitly and reproducibly.

## Related tools

- **MetaNet** (Provides c_net_calculate() (Spearman correlation with p-value adjustment) and c_net_build() (threshold-based edge filtering and singleton removal) used to construct and prune the co-occurrence network.) — https://github.com/Asa12138/MetaNet
- **R** (Runtime environment; MetaNet is an R package requiring R >= 4.0 and built on igraph.) — https://cran.r-project.org/web/packages/MetaNet
- **igraph** (Underlying graph data structure on which MetaNet's filtered network objects and downstream topology functions operate.)

## Examples

```
library(MetaNet); cor <- c_net_calculate(totu, method='spearman', p.adjust.method='BH'); net <- c_net_build(cor, r_threshold=0.6, p_threshold=0.05, use_p_adj=TRUE, delete_single=TRUE)
```

## Evaluation signals

- Network edge set contains no edges with |Spearman ρ| ≤ 0.6 (or the chosen threshold) or with BH-adjusted p ≥ 0.05.
- Vertex count after delete_single = TRUE matches the expected pruned size (e.g. 871 of 914 species in Case 1); a large discrepancy flags threshold or filtering errors.
- Degree distribution of the filtered network can be fit by a power law (fit_power), consistent with scale-free topology reported for Case 1.
- Warning-free run: MetaNet emits 'Have not do p-value adjustment!' when use_p_adj is not honored — absence of this warning confirms adjusted p-values were used.
- Re-running the pipeline on the same table reproduces the identical node and edge set, confirming reproducible thresholding.

## Limitations

- Correlation-based construction may overlook complex, non-linear, or indirect associations, as noted in the article's discussion.
- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology and downstream biological interpretation if not justified and reported.
- Lack of standardized protocols for similarity metrics, thresholding, and null model selection across the field makes cross-study comparison of filtered networks difficult.

## Evidence

- [methods] Case 1 global microbial co-occurrence network: [evidence span withheld — anchor status: not_found]
- [intro] Threshold selection caveat: [evidence span withheld — anchor status: not_found]
- [discussion] Non-linear association caveat: [evidence span withheld — anchor status: not_found]
