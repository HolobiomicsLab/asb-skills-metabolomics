---
name: consensus-clustering-of-metabolic-networks
description: Use when a two-condition metabolomics/lipidomics dataset has already
  been modeled as glasso partial-correlation networks (DNEAobject with estimated networks)
  and you need to partition the edge-weighted graph into metabolic modules for downstream
  enrichment testing.
when_to_use_negative:
- The data are not from two experimental conditions — DNEA performs joint estimation
  across two groups, so a single-group network cannot be consensus-clustered this
  way (use CorrelationCalculator instead)
- The networks have not yet been estimated (no BICtune/stabilitySelection/getNetworks
  run) — clustering requires the glasso edge weights
- The goal is standard pathway enrichment of endogenous metabolites that map fully
  to KEGG — pathway analysis rather than data-driven module discovery is the appropriate
  tool
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_0091
tools:
- name: DNEA R package
  role: Provides clusterNet() for consensus clustering using 7 igraph algorithms with
    tau and max_iterations parameters, on networks from getNetworks()
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: igraph
  role: Supplies the 7 graph-clustering algorithms whose results are combined into
    a consensus subnetwork membership
- name: netgsa
  role: Consumes the resulting subnetworks (min_size = 5) as data-driven pathways
    for differential enrichment testing
- name: Cytoscape v3.10.0
  role: Visualization of the clustered networks via node/edge lists exported by getNetworkFiles()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:d41367feb062216b6c3944cf3600199774284e157b64347cb6d009ad51d8e278
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - Cytoscape v3.10.0
  - DNEA R package
  - R
  - glasso
  - igraph
  - netgsa
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# consensus-clustering-of-metabolic-networks

## Summary

Consensus clustering of jointly estimated condition-specific metabolite networks (GLASSO partial correlation graphs from the DNEA R package) into highly interconnected metabolic subnetworks/modules, which are then testable for differential enrichment via NetGSA.

## When to use

Use when a two-condition metabolomics/lipidomics dataset has already been modeled as glasso partial-correlation networks (DNEAobject with estimated networks) and you need to partition the edge-weighted graph into metabolic modules for downstream enrichment testing.

## When NOT to use

- The data are not from two experimental conditions — DNEA performs joint estimation across two groups, so a single-group network cannot be consensus-clustered this way (use CorrelationCalculator instead)
- The networks have not yet been estimated (no BICtune/stabilitySelection/getNetworks run) — clustering requires the glasso edge weights
- The goal is standard pathway enrichment of endogenous metabolites that map fully to KEGG — pathway analysis rather than data-driven module discovery is the appropriate tool

## Inputs

- DNEAobject with jointly estimated glasso networks for two conditions (from getNetworks(aprox = TRUE)), e.g. TEDDY case-vs-control metabolomics data (134 metabolites)
- Optional: edge-filtered network via filterNetworks(pcor = 0.166) or a percentile threshold

## Outputs

- Consensus subnetwork (module) membership assignments for clustered metabolites
- Set of subnetworks, of which those with >= 5 metabolites (11 of 13 in the TEDDY demo) are passed to runNetGSA for differential enrichment testing
- Cytoscape-ready node and edge lists via getNetworkFiles()

## How to apply

After network estimation with getNetworks(aprox = TRUE) on the DNEAobject (e.g., TEDDY case-vs-control with 134 metabolites), run clusterNet(tau = 0.5, max_iterations = 5), which applies 7 igraph clustering algorithms and iterates until subnetwork membership agreement (consensus) is reached across the two condition-specific networks. The tau = 0.5 threshold means edges must agree between conditions at a 0.5 probability cutoff before clustering. Output is a subnetwork membership assignment; modules containing at least 5 metabolites (min_size = 5 for runNetGSA) are the units carried forward to differential enrichment testing — in the TEDDY demo this yielded 13 clustered metabolites forming 11 testable subnetworks of >= 5 metabolites. Optionally pre-filter weak edges with filterNetworks(pcor = 0.166) to control network density before clustering.

## Related tools

- **DNEA R package** (Provides clusterNet() for consensus clustering using 7 igraph algorithms with tau and max_iterations parameters, on networks from getNetworks()) — http://www.github.com/Karnovsky-Lab/DNEA/
- **igraph** (Supplies the 7 graph-clustering algorithms whose results are combined into a consensus subnetwork membership)
- **netgsa** (Consumes the resulting subnetworks (min_size = 5) as data-driven pathways for differential enrichment testing)
- **Cytoscape v3.10.0** (Visualization of the clustered networks via node/edge lists exported by getNetworkFiles())

## Examples

```
TEDDYdat <- clusterNet(object = TEDDYdat, tau = 0.5, max_iterations = 5)
```

## Evaluation signals

- In the TEDDY demonstration, consensus clustering produced subnetworks covering 13 metabolites, of which 11 subnetworks contained >= 5 metabolites and were testable for enrichment
- 8 of the 11 testable subnetworks (S1–S8) were significantly enriched at FDR < 0.05, with subnetwork 1 top-ranked by FDR — a sanity-check expectation for reproducing the reference run
- clusterNet must run with tau = 0.5 and max_iterations = 5 and terminate because subnetwork membership agreement was reached within the iteration cap
- Clustered module membership should be consistent between the control- and case-specific networks (edges shared via the joint-estimation model), not condition-fragmented
- Exported node/edge list files from getNetworkFiles() must parse correctly in Cytoscape and reflect the consensus module assignments

## Limitations

- Consensus clustering is only meaningful after regularization; the p >> n setting requires the glasso penalty and stability selection to avoid clustering on unstable edges
- The tau = 0.5 / max_iterations = 5 settings are the demonstrated defaults; no convergence guarantee is stated if agreement is not reached within 5 iterations
- Subnetworks smaller than 5 metabolites cannot be tested by runNetGSA and are effectively discarded from enrichment analysis
- Sparse networks from small sample sizes reduce edge counts and may fragment modules (edge totals dropped as samples decreased in the article's downsampling analysis)

## Evidence

- [intro] The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks: "The network estimation step is followed by consensus clustering of the resulting network to identify highly interconnected subnetworks"
- [other] clusterNet call with parameters: [evidence span withheld — anchor status: not_found]
- [readme] README on clustering and enrichment: "Networks constructed using DNEA can be clustered to identify metabolic modules, or sub networks, within the data and subsequently test them for enrichment across experimental conditions"
- [methods] filtering step before clustering: "TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.166)"
