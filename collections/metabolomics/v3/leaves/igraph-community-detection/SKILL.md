---
name: igraph-community-detection
description: Use when a partial-correlation network estimated from two-condition metabolomics
  data (e.g., TEDDY-style GLASSO networks) needs to be partitioned into metabolic
  modules via consensus clustering before subnetwork enrichment testing.
when_to_use_negative:
- The goal is pathway enrichment of well-annotated endogenous metabolites that map
  to KEGG/pathway databases — data-driven module detection is designed for datasets
  with exogenous, secondary, or unknown metabolites
- The network is a single-group or marginal-correlation network where indirect correlations
  confound the identification of biologically relevant interactions
- The network has not yet been estimated/filtered (clusterNet() expects a DNEA object
  with networks from getNetworks(), optionally pruned by filterNetworks())
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DNEA R package
  role: Provides clusterNet() and CCsummary() for consensus clustering and summary
    reporting over the estimated networks
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: igraph
  role: Supplies the seven community-detection algorithms (edge_betweenness, fast_greedy,
    infomap, label_prop, louvain, walktrap, leading_eigen) applied iteratively by
    clusterNet()
- name: R
  role: Runtime environment; native set.seed() is required since clusterNet() bypasses
    BiocParallel random-number handling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:32b727ba3594e0b640ccf7dd7111b05c73be7fbdbac8a6250a223cc52ce51a41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# igraph-community-detection

## Summary

Identifies metabolic modules (subnetworks) in an estimated metabolite network by consensus clustering over seven igraph community-detection algorithms, as implemented in the DNEA R package's clusterNet() function. Agreement across algorithms (tau threshold) determines final subnetwork membership, which is appended to the node list for downstream NetGSA enrichment testing.

## When to use

Use when a partial-correlation network estimated from two-condition metabolomics data (e.g., TEDDY-style GLASSO networks) needs to be partitioned into metabolic modules via consensus clustering before subnetwork enrichment testing.

## When NOT to use

- The goal is pathway enrichment of well-annotated endogenous metabolites that map to KEGG/pathway databases — data-driven module detection is designed for datasets with exogenous, secondary, or unknown metabolites
- The network is a single-group or marginal-correlation network where indirect correlations confound the identification of biologically relevant interactions
- The network has not yet been estimated/filtered (clusterNet() expects a DNEA object with networks from getNetworks(), optionally pruned by filterNetworks())

## Inputs

- DNEA object with estimated (and optionally filtered, e.g., pcor >= 0.166) networks for two experimental conditions
- tau agreement threshold (0.5–1, default 0.5)
- max_iterations cap on consensus iterations (e.g., 5)
- R random seed for reproducibility (set.seed(417) in the vignette)

## Outputs

- DNEA object with populated consensus_clustering slot
- Subnetwork (metabolic module) membership assignments appended to the node list
- Clustering summary from CCsummary() (e.g., number of subnetworks and clustered vs. unclustered metabolites)

## How to apply

Run clusterNet() on a processed DNEA object (e.g., the packaged 'dnw' TEDDY dataset with 134 metabolites across two experimental groups) after network estimation via getNetworks(). Set the native R seed (set.seed(417)) first because clusterNet() does not use the BiocParallel framework and several of the seven clustering methods rely on random number generation. The function iteratively applies seven igraph algorithms — edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, and leading_eigen — until agreement on subnetwork membership is reached, capped by max_iterations (e.g., 5). The agreement threshold tau is the percentage of algorithms that must agree on a subnetwork's membership; it defaults to 0.5 (4 of 7 algorithms) and is adjustable in the range 0.5–1, with higher tau demanding stricter consensus. Results populate the consensus_clustering slot, subnetwork membership is appended to the node list, and CCsummary() reports the clustering outcome for validation.

## Related tools

- **DNEA R package** (Provides clusterNet() and CCsummary() for consensus clustering and summary reporting over the estimated networks) — http://www.github.com/Karnovsky-Lab/DNEA/
- **igraph** (Supplies the seven community-detection algorithms (edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen) applied iteratively by clusterNet())
- **R** (Runtime environment; native set.seed() is required since clusterNet() bypasses BiocParallel random-number handling)

## Examples

```
set.seed(417); TEDDYdat <- clusterNet(object = TEDDYdat, tau = 0.5, max_iterations = 5); CCsummary(TEDDYdat)
```

## Evaluation signals

- CCsummary() at tau = 0.5 on the TEDDY vignette data reports 13 subnetworks containing 132/134 metabolites, with 2 metabolites left as independent
- The consensus_clustering slot of the returned DNEA object is populated (non-empty)
- Subnetwork membership assignments are appended to the node list — every clustered metabolite carries a module label
- Re-running with the same seed (set.seed(417)) and parameters reproduces identical subnetwork membership, since several clustering algorithms depend on random number generation
- Raising tau above 0.5 toward 1 shrinks or splits modules, reflecting the stricter percentage-of-algorithms agreement requirement

## Limitations

- clusterNet() does not use the BiocParallel framework, so reproducibility depends on setting the native R seed manually
- The default tau = 0.5 requires only 4 of 7 algorithms to agree, so module boundaries may vary with tau; the result at a given tau is validated against vignette-documented values (13 subnetworks, 132/134 metabolites clustered)
- Consensus clustering is run on the estimated/filtered network; unfiltered dense networks (e.g., including indirect marginal correlations) can confound module identification — filterNetworks() with a partial-correlation threshold is recommended for interpretation

## Evidence

- [other] clusterNet() identifies metabolic modules by consensus among clustering algorithms: the agreement threshold — the percentage of algorithms that must agree on a subnetwork's membership — defaults to 50% (4 of 7 algorithms), adjustable via tau (0.5–1): [evidence span withheld — anchor status: not_found]
- [other] Seven igraph algorithms applied iteratively until agreement: [evidence span withheld — anchor status: not_found]
- [other] Validation against vignette values: [evidence span withheld — anchor status: not_found]
- [intro] Consensus clustering follows network estimation in the DNEA workflow: "The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks"
- [readme] Clustering enables module-based enrichment testing: "Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data and subsequently test them for enrichment across experimental conditions"
