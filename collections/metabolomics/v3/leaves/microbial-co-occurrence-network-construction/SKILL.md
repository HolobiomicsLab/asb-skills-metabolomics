---
name: microbial-co-occurrence-network-construction
description: Use when a species/genus abundance table needs to be converted into a
  co-occurrence network with objective edge thresholds, module detection, and hub/connector
  classification.
when_to_use_negative:
- The goal is differential edge comparison across timepoints or groups rather than
  building a single baseline network (use extract_sample_net/c_net_compare instead).
- The data require non-linear or indirect association detection — the article notes
  correlation-based network optimization 'may overlook complex, non-linear, or indirect
  associations'.
- The input is already a network graph and the task is layout, annotation, or visualization
  rather than edge inference.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: MetaNet
  role: core R package providing c_net_calculate, c_net_build, c_net_module, zp_analyse,
    fit_power, net_par, and rmt/RMT_threshold for correlation-based co-occurrence
    network construction and thresholding
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: igraph
  role: underlying graph engine on which MetaNet's core functionality is built
- name: pcutils
  role: companion package required for MetaNet installation and data handling (e.g.
    otutab test data)
  repo: github.com/Asa12138/pcutils
- name: ggraph
  role: network plotting backend referenced alongside MetaNet for flexible visualization
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:635e0eab5cd8b69844199ee42cf5e5e9c504c44d0b1f9dbdd46ed6d5e1282641
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - ggraph
  - igraph
  - pcutils
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8444444444444444
n_claims: 45
---

# microbial-co-occurrence-network-construction

## Summary

Constructs microbial co-occurrence networks from abundance tables using MetaNet's c_net_calculate and c_net_build, with optional random matrix theory (RMT) thresholding to replace arbitrary correlation cutoffs.

## When to use

Use when a species/genus abundance table needs to be converted into a co-occurrence network with objective edge thresholds, module detection, and hub/connector classification.

## When NOT to use

- The goal is differential edge comparison across timepoints or groups rather than building a single baseline network (use extract_sample_net/c_net_compare instead).
- The data require non-linear or indirect association detection — the article notes correlation-based network optimization 'may overlook complex, non-linear, or indirect associations'.
- The input is already a network graph and the task is layout, annotation, or visualization rather than edge inference.

## Inputs

- species/genus-level abundance table (e.g. MetaNet 'micro' otutab, features as columns)
- prevalence filter threshold (e.g. species present in more than 10% of samples)
- correlation threshold parameters (r_threshold, p_threshold, use_p_adj, delete_single)
- RMT sweep parameters (occor.r matrix, min_threshold, max_threshold, step)

## Outputs

- co-occurrence network object (igraph/metanet graph) with significant |rho|>threshold edges
- module membership and keystone role classifications (module hubs, connectors)
- power-law fit of the degree distribution and global topological indices (net_par)
- summary statistics table (e.g. case1_network_summary.csv) and rmt_threshold.txt cutoff

## How to apply

Transpose the abundance table so features are columns and run c_net_calculate with method='spearman' and p.adjust.method='BH' to obtain correlation and adjusted-p matrices. Choose the edge cutoff objectively by running rmt()/RMT_threshold() on the correlation matrix, sweeping candidate cutoffs between min_threshold and max_threshold and selecting the one minimizing eigenvalue-spacing deviation from the random matrix theory expectation; otherwise apply a stated threshold such as r_threshold=0.6 with p_threshold=0.05 and use_p_adj=TRUE. Build the graph with c_net_build(cor, r_threshold, ...), using delete_single=TRUE to prune singletons (the Case 1 network retained 871 of 914 prevalent taxa after filtering to species present in >10% of samples). Detect modules and compute Zi/Pi keystone roles with get_module/c_net_module plus zp_analyse, then verify scale-free topology by fitting a power law to the degree distribution with fit_power and computing global indices with net_par. The Case 1 reference output is a 871-node, 6-module network with 13 module hubs and 19 connectors following a power-law degree distribution.

## Related tools

- **MetaNet** (core R package providing c_net_calculate, c_net_build, c_net_module, zp_analyse, fit_power, net_par, and rmt/RMT_threshold for correlation-based co-occurrence network construction and thresholding) — https://cran.r-project.org/web/packages/MetaNet
- **igraph** (underlying graph engine on which MetaNet's core functionality is built)
- **pcutils** (companion package required for MetaNet installation and data handling (e.g. otutab test data)) — github.com/Asa12138/pcutils
- **ggraph** (network plotting backend referenced alongside MetaNet for flexible visualization)

## Examples

```
cor <- c_net_calculate(totu); net <- c_net_build(cor, r_threshold = 0.6, p_threshold = 0.05, use_p_adj = TRUE, delete_single = TRUE)
```

## Evaluation signals

- Reproducing Case 1 must yield 871 species nodes from the 914 prevalent taxa, organized into six modules with 13 module hubs and 19 connectors; any deviation flags the result for expert review.
- Degree distribution must follow a power-law fit (scale-free topology), verified with fit_power.
- The RMT-derived r_threshold must fall within [min_threshold, max_threshold] and be directly usable as the r_threshold argument of c_net_build.
- Edges must satisfy |rho| > r_threshold with BH-adjusted p < p_threshold (0.05) and singletons removed (delete_single=TRUE).
- Module hubs and connectors are classified by within-module (Zi) and among-module (P) connectivity thresholds from zp_analyse/zp_plot.

## Limitations

- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology — RMT mitigates but does not eliminate modeling choices.
- Correlation-based network optimization may overlook complex, non-linear, or indirect associations.
- Lack of standardized protocols for network construction (similarity metrics, thresholding, null model selection) complicates cross-study comparability.

## Evidence

- [readme] three-function quick start: [evidence span withheld — anchor status: external_doc]
- [readme] no p-value adjustment warning: [evidence span withheld — anchor status: external_doc]
- [intro] subjective thresholding: [evidence span withheld — anchor status: not_found]
- [readme] module detection API: [evidence span withheld — anchor status: external_doc]
- [abstract] Case 1 longitudinal design: [evidence span withheld — anchor status: not_found]
