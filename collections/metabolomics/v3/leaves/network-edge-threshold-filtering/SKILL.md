---
name: network-edge-threshold-filtering
description: Use when a GGM/marginal-correlation network over metabolomics data is
  too dense to interpret, and edges must be filtered by an absolute correlation cutoff
  (e.g., |r| or |pcor| >= 0.2).
when_to_use_negative:
- When the input is raw peak-intensity data rather than an already-estimated network
  — data preprocessing (excluding metabolites missing in >30% of samples, median imputation)
  and network estimation must precede filtering
- When the dataset has fewer than ~500 samples per group and the lambda approximation
  (Eq. 3) rather than BIC tuning is used — the resulting unfiltered networks may not
  be comparable to paper edge counts
- When filtering is desired by statistical significance rather than edge strength
  — filterNetworks() supports only percentile or correlation-threshold based filtering
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DNEA R package
  role: Provides filterNetworks() to filter estimated networks by edge strength (percentile
    or partial correlation threshold); filtered edge lists are exported via getNetworkFiles()
    for Cytoscape visualization
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: Metabox2
  role: Produces marginal (Pearson) correlation networks whose edge lists (e.g., metabox2_ia_edge_list.csv)
    are filtered at |r| >= 0.2 for the density comparison against DNEA
- name: Cytoscape v3.10.0
  role: Visualizes the filtered networks by importing edge lists generated with getNetworkFiles()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:7230cef4148dde55062926b44b9e3730d977a367506f492a66d3cd0b0b4c4301
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape v3.10.0
  - DNEA R package
  - Metabox2
  - R
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# network-edge-threshold-filtering

## Summary

Prune estimated metabolite-metabolite networks by thresholding edge strength (absolute partial or marginal correlation coefficient), using the DNEA package's filterNetworks() function to control network density and retain the most biologically relevant connections.

## When to use

Use when a GGM/marginal-correlation network over metabolomics data is too dense to interpret, and edges must be filtered by an absolute correlation cutoff (e.g., |r| or |pcor| >= 0.2).

## When NOT to use

- When the input is raw peak-intensity data rather than an already-estimated network — data preprocessing (excluding metabolites missing in >30% of samples, median imputation) and network estimation must precede filtering
- When the dataset has fewer than ~500 samples per group and the lambda approximation (Eq. 3) rather than BIC tuning is used — the resulting unfiltered networks may not be comparable to paper edge counts
- When filtering is desired by statistical significance rather than edge strength — filterNetworks() supports only percentile or correlation-threshold based filtering

## Inputs

- Estimated metabolite-metabolite network object from DNEA (e.g., TEDDYdat after getNetworks()) containing edge lists with partial correlation weights
- Threshold parameter: an absolute partial correlation value (e.g., pcor = 0.2 or pcor = 0.166) or a percentile of edge strength
- Optionally, an edge list CSV (e.g., metabox2_ia_edge_list.csv with columns Metabolite A, Metabolite B, Pearson r) to filter with the same |r| >= 0.2 cutoff

## Outputs

- Filtered network object (e.g., TEDDYdat_filtered) retaining only edges meeting the absolute correlation cutoff
- Exported edge list files via getNetworkFiles() for Cytoscape v3.10.0 visualization
- Edge counts (e.g., 576 MetaBox2 Pearson edges vs. 172 DNEA partial-correlation edges at |r| >= 0.2 on the TEDDY IA arm) for cross-tool density comparison

## How to apply

After network estimation (e.g., DNEA's GLASSO-based partial correlation networks from TEDDY IA/T1D metabolomics), apply the filterNetworks() function, which supports filtering either by a percentile of edge strength or by an absolute correlation threshold. In the paper's cross-tool comparison (DNEA vs. Filigree vs. MetaBox2), all three networks were filtered to keep edges with absolute correlation coefficient >= 0.2 so that edge counts were comparable across tools; in the methods vignette, filterNetworks() is demonstrated with pcor = 0.166. The rationale is that marginal-correlation networks are dense because they include edges representing both direct and indirect correlations, the latter confounding interpretation, whereas thresholding partial correlations controls density. Edge counts before and after filtering serve as the key diagnostic of filtering effect and comparability.

## Related tools

- **DNEA R package** (Provides filterNetworks() to filter estimated networks by edge strength (percentile or partial correlation threshold); filtered edge lists are exported via getNetworkFiles() for Cytoscape visualization) — http://www.github.com/Karnovsky-Lab/DNEA/
- **Metabox2** (Produces marginal (Pearson) correlation networks whose edge lists (e.g., metabox2_ia_edge_list.csv) are filtered at |r| >= 0.2 for the density comparison against DNEA)
- **Cytoscape v3.10.0** (Visualizes the filtered networks by importing edge lists generated with getNetworkFiles())

## Examples

```
TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.2)
```

## Evaluation signals

- The filtered network contains exactly the edges whose absolute correlation coefficient meets the cutoff (e.g., >= 0.2 for cross-tool comparison, or pcor = 0.166 as in the vignette)
- Cross-tool density check: at the same |r| >= 0.2 cutoff on the TEDDY IA arm, the MetaBox2 Pearson edge list contains 576 edges vs. 172 edges in the DNEA partial-correlation network — a marked density difference confirming marginal networks retain indirect associations
- filterNetworks() output preserves the original metabolite nodes while removing below-threshold edges (filtering affects edges, not features)
- The filtered edge list exported via getNetworkFiles() imports cleanly into Cytoscape v3.10.0 with correlation weights as edge attributes

## Limitations

- Marginal-correlation networks (e.g., Metabox2) are dense due to inclusion of both direct and indirect correlations; indirect edges can confound interpretation, so identical absolute thresholds yield very different densities across methods
- filterNetworks() supports only edge-strength-based criteria (percentile or partial correlation threshold); choosing the threshold value (e.g., 0.2 vs. 0.166) is a user decision that directly changes network density
- Network sparsity is sensitive to sample size — in TEDDY subsampling experiments, total edges dropped from 26,888 at 100% of samples to 1,354 at 5% (T1D arm), so a fixed cutoff interacts strongly with input sample size

## Evidence

- [intro] the filterNetworks() function is available to filter networks based on edge strength (percentile or partial correlation threshold) and highlight the most biologically relevant connections: "the filterNetworks() function is available to filter networks based on edge strength (percentile or partial correlation threshold) and highlight the most biologically relevant connections"
- [discussion] The DNEA, Filigree and Metabox2 networks were filtered to include edges with an absolute value of correlation coefficient greater than or equal to 0.2.: "The DNEA, Filigree and Metabox2 networks were filtered to include edges with an absolute value of correlation coefficient greater than or equal to 0.2."
- [methods] TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.166): "TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.166)"
- [discussion] they tend to be very dense due to the inclusion of edges representing both direct and indirect correlations. The latter can confound the results: "they tend to be very dense due to the inclusion of edges representing both direct and indirect correlations. The latter can confound the results"
