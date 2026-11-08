---
name: differential-network-module-agreement-thresholding
description: Use when a DNEA object with estimated two-condition partial correlation
  networks (e.g. TEDDY metabolomics, 134 metabolites) must be partitioned into metabolic
  modules and you need to set or adjust the consensus agreement threshold (tau).
when_to_use_negative:
- Input is a single-condition network or one-group metabolomics data — DNEA/clusterNet
  requires two experimental groups for joint estimation before consensus clustering
- Metabolites are well-annotated endogenous compounds that map cleanly to pathway
  databases — conventional pathway enrichment analysis may be preferable to data-driven
  modules
- Networks are unfiltered and very dense — module detection should follow filterNetworks()
  edge-strength filtering (percentile or partial correlation threshold) to control
  network density
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DNEA R package
  role: Provides clusterNet(), CCsummary(), and the consensus_clustering slot that
    implement and report the consensus clustering
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: igraph
  role: Supplies the seven underlying clustering algorithms (edge_betweenness, fast_greedy,
    infomap, label_prop, louvain, walktrap, leading_eigen) iterated by clusterNet()
- name: R
  role: Runtime environment; native set.seed() is required because clusterNet() does
    not use BiocParallel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:7c2dcf2f0fa3afcfdaf7953e62c2837780a45c61ad20002cbda920880a366a13
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

# differential-network-module-agreement-thresholding

## Summary

Identifies metabolic modules (subnetworks) in DNEA-estimated differential metabolite networks by consensus clustering over seven igraph algorithms, where the tau parameter sets the fraction of algorithms (default 0.5, i.e. 4 of 7) that must agree on subnetwork membership.

## When to use

Use when a DNEA object with estimated two-condition partial correlation networks (e.g. TEDDY metabolomics, 134 metabolites) must be partitioned into metabolic modules and you need to set or adjust the consensus agreement threshold (tau).

## When NOT to use

- Input is a single-condition network or one-group metabolomics data — DNEA/clusterNet requires two experimental groups for joint estimation before consensus clustering
- Metabolites are well-annotated endogenous compounds that map cleanly to pathway databases — conventional pathway enrichment analysis may be preferable to data-driven modules
- Networks are unfiltered and very dense — module detection should follow filterNetworks() edge-strength filtering (percentile or partial correlation threshold) to control network density

## Inputs

- DNEA object with estimated networks (e.g. the packaged 'dnw' TEDDY dataset, 134 metabolites across two experimental groups), optionally after filterNetworks()
- tau agreement threshold (default 0.5, range 0.5–1)
- max_iterations (e.g. 5) and a native R seed (e.g. set.seed(417))

## Outputs

- Populated consensus_clustering slot on the DNEA object
- Subnetwork (metabolic module) membership assignments appended to the node list
- Clustering summary table from CCsummary() (e.g. 13 subnetworks, 132/134 metabolites clustered, 2 independent)
- Saved RDS of consensus clustering results (e.g. consensus_clustering_results.rds)

## How to apply

After network estimation and edge filtering (e.g. filterNetworks() with pcor = 0.166), run clusterNet() on the DNEA object; it iteratively applies seven igraph clustering algorithms (edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen) until agreement on subnetwork membership is reached, controlled by tau (0.5–1, the percentage of algorithms that must agree, defaulting to 50% = 4 of 7) and max_iterations. Set the native R seed (e.g. set.seed(417)) before running because clusterNet() does not use the BiocParallel framework and several clustering methods rely on random number generation. Membership assignments are appended to the node list and results are stored in the consensus_clustering slot; summarize with CCsummary(). In the TEDDY vignette at tau = 0.5 and max_iterations = 5, 132 of 134 metabolites clustered into 13 subnetworks with 2 left as independent metabolites.

## Related tools

- **DNEA R package** (Provides clusterNet(), CCsummary(), and the consensus_clustering slot that implement and report the consensus clustering) — http://www.github.com/Karnovsky-Lab/DNEA/
- **igraph** (Supplies the seven underlying clustering algorithms (edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen) iterated by clusterNet())
- **R** (Runtime environment; native set.seed() is required because clusterNet() does not use BiocParallel)

## Examples

```
set.seed(417); TEDDYdat <- clusterNet(object = TEDDYdat, tau = 0.5, max_iterations = 5); CCsummary(TEDDYdat)
```

## Evaluation signals

- CCsummary() reports 13 subnetworks containing 132/134 metabolites (2 independent metabolites) for the TEDDY vignette dataset at tau = 0.5, matching the documented vignette values
- The consensus_clustering slot of the DNEA object is populated and node-list entries carry subnetwork membership labels
- tau is within the documented 0.5–1 range and defaults to 0.5 (4 of 7 algorithms agree) when not overridden
- A fixed native R seed (e.g. set.seed(417)) reproduces the clustering, confirming the random-seed dependency was handled
- Results serialize cleanly to consensus_clustering_results.rds and reload as a valid DNEA object

## Limitations

- clusterNet() does not use the BiocParallel framework, so it cannot be parallelized via BPPARAM and depends on native random number generation — a seed must be set explicitly
- Consensus clustering results depend on the tau threshold; raising tau toward 1 demands stricter algorithm agreement and changes module membership
- Modules are data-driven, so they are not equivalent to curated KEGG pathways; downstream NetGSA enrichment is needed for condition-specific interpretation

## Evidence

- [intro] The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks: "The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks"
- [other] clusterNet() identifies metabolic modules by consensus among clustering algorithms: the agreement threshold — the percentage of algorithms that must agree on a subnetwork's membership — defaults to 50% (4 of 7 algorithms), adjustable via tau (0.5–1): [evidence span withheld — anchor status: not_found]
- [readme] Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data and subsequently test them for enrichment across experimental conditions: "Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data and subsequently test"
