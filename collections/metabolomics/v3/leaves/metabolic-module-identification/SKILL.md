---
name: metabolic-module-identification
description: Use when a DNEA object contains estimated condition networks and you
  need to partition metabolites into interconnected sub networks (metabolic modules)
  before enrichment analysis.
when_to_use_negative:
- The network has not yet been estimated or filtered (clusterNet() expects a DNEA
  object past the getNetworks/filterNetworks steps of the workflow)
- The data has only one experimental group — DNEA performs joint estimation across
  two conditions, and CorrelationCalculator is the single-group alternative
- The goal is curated pathway enrichment over well-annotated endogenous metabolites
  rather than data-driven modules (e.g. FELLA/KEGG mapping, which only covered 19
  of 174 TEDDY metabolites)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DNEA R package
  role: Provides clusterNet() and CCsummary(), which run the iterative consensus clustering
    and store results in the consensus_clustering slot
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: igraph
  role: Supplies the seven clustering algorithms (edge_betweenness, fast_greedy, infomap,
    label_prop, louvain, walktrap, leading_eigen) whose membership votes are combined
    at the tau threshold
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:d80fd96332fad1d2b07d3a5a94d77283ba6a64fffbff281129120cc5ecbf015f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# metabolic-module-identification

## Summary

Identifies metabolic modules (sub networks) within DNEA-estimated metabolite networks by consensus clustering across seven igraph clustering algorithms, using a tau agreement threshold. This turns a partial correlation network into data-driven 'pathways' for downstream NetGSA enrichment testing.

## When to use

Use when a DNEA object contains estimated condition networks and you need to partition metabolites into interconnected sub networks (metabolic modules) before enrichment analysis.

## When NOT to use

- The network has not yet been estimated or filtered (clusterNet() expects a DNEA object past the getNetworks/filterNetworks steps of the workflow)
- The data has only one experimental group — DNEA performs joint estimation across two conditions, and CorrelationCalculator is the single-group alternative
- The goal is curated pathway enrichment over well-annotated endogenous metabolites rather than data-driven modules (e.g. FELLA/KEGG mapping, which only covered 19 of 174 TEDDY metabolites)

## Inputs

- DNEA object (DNEAresults) with estimated networks after BICtune, stabilitySelection, getNetworks, and optional filterNetworks
- tau agreement threshold (0.5–1; default 0.5)
- max_iterations parameter (e.g. 5)

## Outputs

- DNEA object with populated consensus_clustering slot
- Subnetwork membership assignments appended to the node list
- Clustering summary table from CCsummary() (e.g. 13 sub networks, 132/134 metabolites clustered)

## How to apply

Run clusterNet() on the processed DNEA object (e.g. the packaged 'dnw' TEDDY dataset with 134 metabolites across two groups). First set the native R seed (set.seed(417)) because clusterNet() does not use the BiocParallel framework and several clustering methods rely on random number generation. clusterNet() iteratively applies seven igraph clustering algorithms — edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen — until subnetwork membership agreement is reached, with the agreement threshold tau defaulting to 0.5 (4 of 7 algorithms must agree; adjustable 0.5–1) and max_iterations bounding the loop. Results populate the consensus_clustering slot and subnetwork membership is appended to the node list; summarize with CCsummary(). With tau = 0.5 and max_iterations = 5 on the TEDDY vignette data, expect 13 sub networks covering 132/134 metabolites, with 2 left as independent metabolites.

## Related tools

- **DNEA R package** (Provides clusterNet() and CCsummary(), which run the iterative consensus clustering and store results in the consensus_clustering slot) — http://www.github.com/Karnovsky-Lab/DNEA/
- **igraph** (Supplies the seven clustering algorithms (edge_betweenness, fast_greedy, infomap, label_prop, louvain, walktrap, leading_eigen) whose membership votes are combined at the tau threshold)

## Examples

```
set.seed(417); dnw <- clusterNet(object = dnw, tau = 0.5, max_iterations = 5); CCsummary(dnw)
```

## Evaluation signals

- CCsummary() reports 13 sub networks containing 132/134 metabolites for the TEDDY vignette dataset at tau = 0.5, with 2 metabolites left as independent
- The consensus_clustering slot of the returned DNEA object is populated (not NULL) after clusterNet()
- Subnetwork membership is appended to the node list of the DNEA object
- set.seed() was called before clusterNet() (e.g. set.seed(417)) since the function does not use BiocParallel and clustering methods rely on RNG — reruns with the same seed should reproduce memberships
- tau value lies within the documented 0.5–1 range, where 0.5 means 4 of 7 algorithms must agree on subnetwork membership

## Limitations

- The agreement threshold tau trades off module granularity: requiring more algorithms to agree (higher tau) leaves more metabolites unclustered as independent metabolites
- clustering depends on random number generation, so seeds must be set explicitly for reproducibility
- Consensus clustering is applied after edge filtering (e.g. filterNetworks with pcor = 0.166); skipping filtering changes the input network density and hence the modules found

## Evidence

- [intro] The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks: "The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks"
- [readme] Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data: "Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data and subsequently test them for enrichment across experimental conditions using the"
- [other] clusterNet() identifies metabolic modules by consensus among clustering algorithms with a tau agreement threshold: [evidence span withheld — anchor status: not_found]
- [intro] Workflow step: network estimation and consensus clustering: "(3) network estimation and consensus clustering"
