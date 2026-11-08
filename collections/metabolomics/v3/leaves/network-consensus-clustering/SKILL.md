---
name: network-consensus-clustering
description: Use when a DNEA object contains estimated condition-specific networks
  (post-getNetworks) and you need to partition metabolites into highly interconnected
  subnetworks for downstream NetGSA enrichment testing.
when_to_use_negative:
- The input is raw peak-intensity data that has not yet passed BICtune(), stabilitySelection(),
  and getNetworks() — clustering requires estimated (and ideally filtered) partial-correlation
  networks.
- The goal is pathway-database-driven enrichment rather than data-driven modules —
  pathway analysis cannot map unknown, exogenous, or lipid/secondary metabolites,
  but this skill defines modules from the network itself.
- You need a single-group network decomposition via a different tool (e.g. CorrelationCalculator
  builds single-condition networks) rather than the DNEA joint two-condition workflow.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: DNEA R package
  role: Provides clusterNet() and CCsummary() for iterative consensus clustering of
    the estimated networks; results populate the consensus_clustering slot
  repo: https://github.com/Karnovsky-Lab/DNEA
- name: igraph
  role: 'Supplies the seven clustering algorithms applied iteratively: edge_betweenness,
    fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen'
- name: R
  role: Runtime environment; set.seed() must be called natively because clusterNet()
    does not use the BiocParallel framework
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:a1bb7465e5b27a720b8dfdc79d6afb9bc2cc23f6d4f1d026e0a176cf3a249226
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Karnovsky-Lab/DNEA
  tools:
  - DNEA R package
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# network-consensus-clustering

## Summary

Identifies metabolic modules (subnetworks) within an estimated partial-correlation network by taking the consensus of seven igraph clustering algorithms, iterated until subnetwork membership agreement reaches a tau threshold (default 50%, i.e. 4 of 7 algorithms). This is step 3 of the DNEA workflow, populating the consensus_clustering slot of the DNEA object.

## When to use

Use when a DNEA object contains estimated condition-specific networks (post-getNetworks) and you need to partition metabolites into highly interconnected subnetworks for downstream NetGSA enrichment testing.

## When NOT to use

- The input is raw peak-intensity data that has not yet passed BICtune(), stabilitySelection(), and getNetworks() — clustering requires estimated (and ideally filtered) partial-correlation networks.
- The goal is pathway-database-driven enrichment rather than data-driven modules — pathway analysis cannot map unknown, exogenous, or lipid/secondary metabolites, but this skill defines modules from the network itself.
- You need a single-group network decomposition via a different tool (e.g. CorrelationCalculator builds single-condition networks) rather than the DNEA joint two-condition workflow.

## Inputs

- DNEA object with estimated networks (output of getNetworks(), optionally filtered via filterNetworks(), e.g. pcor = 0.166)
- tau agreement threshold (0.5–1; default 0.5 = 4 of 7 algorithms)
- max_iterations (e.g. 5)
- native R seed set via set.seed() (e.g. 417)

## Outputs

- DNEA object with populated consensus_clustering slot
- CCsummary() clustering summary table (number of subnetworks, metabolites clustered vs independent)
- Subnetwork membership assignments appended to the node list
- Saved consensus clustering results file (e.g. consensus_clustering_results.rds)

## How to apply

Run clusterNet() on the processed DNEA object (e.g. the packaged 'dnw' TEDDY dataset with 134 metabolites across two experimental groups) after network estimation and optional filterNetworks() edge filtering. Set the native R seed with set.seed(417) first, because clusterNet() does not use the BiocParallel framework and several clustering methods rely on random number generation. The function iteratively applies seven igraph algorithms — edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen — until agreement on subnetwork membership is reached; tau (0.5–1) sets the percentage of algorithms that must agree (default 0.5 = 4 of 7), and max_iterations caps the refinement loop (e.g. 5). Retrieve the clustering summary with CCsummary(), extract subnetwork membership appended to the node list, and save the populated consensus clustering results (e.g. as consensus_clustering_results.rds). With tau = 0.5 on the TEDDY vignette data, 132 of 134 metabolites clustered into 13 subnetworks with 2 left as independent metabolites.

## Related tools

- **DNEA R package** (Provides clusterNet() and CCsummary() for iterative consensus clustering of the estimated networks; results populate the consensus_clustering slot) — https://github.com/Karnovsky-Lab/DNEA
- **igraph** (Supplies the seven clustering algorithms applied iteratively: edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen)
- **R** (Runtime environment; set.seed() must be called natively because clusterNet() does not use the BiocParallel framework)

## Examples

```
set.seed(417); TEDDYdat <- clusterNet(object = TEDDYdat_filtered, tau = 0.5, max_iterations = 5); CCsummary(TEDDYdat); saveRDS(TEDDYdat, 'consensus_clustering_results.rds')
```

## Evaluation signals

- CCsummary() table matches the vignette reference values for tau = 0.5: 13 subnetworks containing 132/134 metabolites, with 2 metabolites left as independent
- consensus_clustering slot of the DNEA object is populated and subnetwork memberships are appended to the node list
- Each metabolite's subnetwork assignment is supported by at least tau × 7 algorithms (default 4 of 7)
- Re-running with the same set.seed(417) reproduces identical subnetwork membership (RNG-dependent, non-BiocParallel)
- Saved RDS (consensus_clustering_results.rds) round-trips and contains memberships consistent with the in-memory object

## Limitations

- clusterNet() does not use the BiocParallel framework; reproducibility depends on manually setting the native R seed because several igraph clustering methods rely on random number generation
- tau is adjustable between 0.5 and 1 — lower thresholds admit noisier modules, higher thresholds may leave many metabolites as independent (2 of 134 remained unclustered even at tau = 0.5)
- Consensus clustering is the third of four DNEA workflow steps and depends on the quality of the estimated networks; sparse networks (few samples) yield fewer edges and may degrade module recovery
- The iterative agreement loop is capped by max_iterations (e.g. 5); convergence on membership agreement is not guaranteed within the cap

## Evidence

- [methods] clusterNet() identifies metabolic modules by consensus among clustering algorithms: the agreement threshold — the percentage of algorithms that must agree on a subnetwork's membership — defaults to 50% (4 of 7 algorithms), adjustable via tau (0.5–1): [evidence span withheld — anchor status: not_found]
- [methods] Seven igraph algorithms applied iteratively: [evidence span withheld — anchor status: not_found]
- [methods] RNG / BiocParallel caveat for clusterNet(): [evidence span withheld — anchor status: not_found]
- [methods] Vignette validation values for TEDDY: [evidence span withheld — anchor status: not_found]
- [intro] Consensus clustering as a workflow step after network estimation: "The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks"
- [readme] Networks can be clustered to define modules for enrichment: "Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data and subsequently test them for enrichment across experimental conditions"
