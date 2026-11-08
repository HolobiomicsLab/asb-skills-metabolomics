---
name: power-law-degree-distribution-fitting
description: Use when a built microbial or multi-omics co-occurrence network (e.g.,
  Spearman |ρ|>0.6, BH-adjusted edges) must be checked for scale-free topology by
  fitting its degree distribution to a power law.
when_to_use_negative:
- Networks built from curated reference databases (e.g., STRING, KEGG hybrid networks)
  rather than correlation-based construction, where a scale-free degree assumption
  may not hold
- Datasets too small or over-filtered, where the degree distribution after thresholding
  (e.g., >10% prevalence plus |ρ|>0.6) is too sparse to support a meaningful power-law
  fit
- When the goal is module/hub role classification (zp_analyse/zp_plot) rather than
  verifying global topology
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_1317
tools:
- name: MetaNet
  role: Provides fit_power for degree-distribution power-law fitting and net_par for
    global topological indices on the built co-occurrence network
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: igraph
  role: Underlying graph engine MetaNet builds on; supplies the degree vector of the
    co-occurrence network used in the power-law fit
- name: R
  role: Runtime environment (R >= 4.0) in which the MetaNet topology-fitting workflow
    is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:3dc67104ed61f2f5563f69c7e3e850a49ae4361a87646eaacecaba1b73f4987b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaNet
  - R
  - ggraph
  - igraph
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_1317
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# power-law-degree-distribution-fitting

## Summary

Verify that a constructed omics co-occurrence network is scale-free by fitting a power law to its degree distribution with MetaNet's fit_power, alongside global topological indices computed via net_par.

## When to use

Use when a built microbial or multi-omics co-occurrence network (e.g., Spearman |ρ|>0.6, BH-adjusted edges) must be checked for scale-free topology by fitting its degree distribution to a power law.

## When NOT to use

- Networks built from curated reference databases (e.g., STRING, KEGG hybrid networks) rather than correlation-based construction, where a scale-free degree assumption may not hold
- Datasets too small or over-filtered, where the degree distribution after thresholding (e.g., >10% prevalence plus |ρ|>0.6) is too sparse to support a meaningful power-law fit
- When the goal is module/hub role classification (zp_analyse/zp_plot) rather than verifying global topology

## Inputs

- species/feature abundance table (e.g., 914 prevalent taxa from the Case 1 underwater exposome dataset)
- MetaNet correlation matrix from c_net_calculate (Spearman coefficients with BH-adjusted p-values)
- built co-occurrence network object (metanet/igraph) with |ρ|>0.6, p<0.05 edges

## Outputs

- power-law fit of the degree distribution (scale-free topology verdict)
- global topological indices from net_par
- summary statistics table such as case1_network_summary.csv (871 species, 6 modules, 13 module hubs, 19 connectors, power-law fit)

## How to apply

First build the network in R with MetaNet: compute pairwise Spearman correlations on the filtered abundance table (species present in >10% of samples) using c_net_calculate with BH p-adjustment, then retain significant edges with c_net_build at r_threshold = 0.6, p_threshold = 0.05, use_p_adj = TRUE, delete_single = TRUE. Extract the node degree vector from the resulting metanet/igraph object and pass it to MetaNet's fit_power to fit a power-law model to the degree distribution. Compute global topological indices with net_par to complement the fit. Judge success against the Case 1 benchmark: the reproduced 871-node underwater exposome co-occurrence network resolved six modules (13 module hubs, 19 connectors) and its degree distribution followed a power-law, indicating scale-free properties; a poor power-law fit flags the network for expert review.

## Related tools

- **MetaNet** (Provides fit_power for degree-distribution power-law fitting and net_par for global topological indices on the built co-occurrence network) — https://cran.r-project.org/web/packages/MetaNet
- **igraph** (Underlying graph engine MetaNet builds on; supplies the degree vector of the co-occurrence network used in the power-law fit)
- **R** (Runtime environment (R >= 4.0) in which the MetaNet topology-fitting workflow is executed)

## Examples

```
net <- c_net_build(c_net_calculate(totu), r_threshold = 0.6, p_threshold = 0.05, use_p_adj = TRUE, delete_single = TRUE); fit_power(degree(net))
```

## Evaluation signals

- The fitted degree distribution follows a power law, matching the Case 1 finding of scale-free properties for the 871-node global microbial co-occurrence network
- Network scale invariants hold before fitting: 914 prevalent taxa retained after the >10% prevalence filter, 871 nodes after thresholding at |ρ|>0.6 with BH-adjusted p<0.05 and singletons deleted
- net_par global topological indices are computed on the same 871-node, 6-module network used for the fit, so node/edge counts are consistent across fit and indices
- Reproduced module statistics (13 module hubs, 19 connectors from Zi/P classification) are coherent with the topology on which the power law is fit
- Deviation from the expected power-law (scale-free) fit flags the result for expert review per the task validation rule

## Limitations

- Correlation-based network construction may overlook complex, non-linear, or indirect associations, so the degree distribution reflects only linear co-occurrence structure
- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing network topology and the resulting power-law fit
- No version numbers or changelog for MetaNet are stated in the source material, complicating exact reproducibility of the fit

## Evidence

- [other] Verify scale-free topology by fitting a power law to the degree distribution with fit_power and compute global topological indices with net_par: [evidence span withheld — anchor status: not_found]
