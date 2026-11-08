---
name: metabolomics-correlation-network-construction
description: Use when building metabolite-metabolite association networks from peak-intensity
  tables, or when explaining why marginal-correlation networks are denser than partial-correlation
  networks on the same data.
when_to_use_negative:
- The analysis goal is pathway enrichment over endogenous metabolites with good KEGG
  annotation rather than data-driven association networks
- The dataset has fewer than ~500 samples per group and you intend to use DNEA's BIC
  tuning protocol, which the paper reports requires a large dataset
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: DNEA R package
  role: Estimates joint regularized partial correlation (GLASSO/GGM) networks from
    two experimental groups and provides filterNetworks()/getNetworkFiles() for edge
    filtering and export
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: Metabox2
  role: Constructs the full pairwise Pearson (marginal) correlation network used as
    the dense-network comparison at |r| >= 0.2
- name: Cytoscape v3.10.0
  role: Visualizes the network by importing the edge list generated with getNetworkFiles()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:5e1d1512817bfc71a9cef03bb6dc0fe78aa7e776adb7accf1aaf9809f57fcde9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape v3.10.0
  - DNEA R package
  - Metabox2
  - R
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# metabolomics-correlation-network-construction

## Summary

Constructs metabolite-metabolite association networks from peak-intensity data, contrasting marginal (Pearson) correlation networks, e.g. from MetaBox2, with regularized partial correlation (GLASSO/GGM) networks from DNEA, to explain why marginal networks are denser.

## When to use

Use when building metabolite-metabolite association networks from peak-intensity tables, or when explaining why marginal-correlation networks are denser than partial-correlation networks on the same data.

## When NOT to use

- The analysis goal is pathway enrichment over endogenous metabolites with good KEGG annotation rather than data-driven association networks
- The dataset has fewer than ~500 samples per group and you intend to use DNEA's BIC tuning protocol, which the paper reports requires a large dataset

## Inputs

- Metabolomics peak-intensity or concentration feature table (samples x metabolites), e.g. TEDDY IA-arm data from Metabolomics Workbench ST001386
- Edge-strength threshold (e.g. absolute correlation/partial correlation >= 0.2)
- Missingness tolerance (exclude metabolites missing in > 30% of samples)

## Outputs

- Filtered edge list (Metabolite A, Metabolite B, Pearson r) as CSV, e.g. metabox2_ia_edge_list.csv with 576 edges
- DNEA partial correlation network edge list (172 edges at |pcor| >= 0.2), exportable via getNetworkFiles() for Cytoscape
- Edge-count density comparison across network types at a shared threshold

## How to apply

Load the metabolomics peak-intensity data (e.g. the TEDDY IA-arm data from Metabolomics Workbench ST001386, project PR000950) and preprocess by excluding metabolites missing in more than 30% of samples and median-imputing the remaining missing values. For a marginal network, run MetaBox2's correlation-based construction and retain edges with absolute Pearson r >= 0.2; for a partial correlation network, run DNEA, which uses the GLASSO algorithm with a BIC-tuned lambda penalty and stability selection, then filter with filterNetworks() at the partial correlation threshold (0.2 was used in the paper's comparison). Rationale: Pearson correlation includes edges for both direct and indirect associations, whereas partial correlations control for all other metabolites, so filtered marginal networks (576 edges in the IA comparison) come out much denser than DNEA networks (172 edges) at the same |r| >= 0.2 cutoff. Export edge lists as (Metabolite A, Metabolite B, coefficient) CSV files and compare edge counts at identical thresholds to judge the density difference.

## Related tools

- **DNEA R package** (Estimates joint regularized partial correlation (GLASSO/GGM) networks from two experimental groups and provides filterNetworks()/getNetworkFiles() for edge filtering and export) — http://www.github.com/Karnovsky-Lab/DNEA/
- **Metabox2** (Constructs the full pairwise Pearson (marginal) correlation network used as the dense-network comparison at |r| >= 0.2)
- **Cytoscape v3.10.0** (Visualizes the network by importing the edge list generated with getNetworkFiles())

## Examples

```
TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.2)
```

## Evaluation signals

- The filtered MetaBox2 IA edge list contains exactly 576 edges, markedly denser than DNEA's 172 edges at the same |r| >= 0.2 cutoff
- Preprocessing invariant: no retained metabolite is missing in more than 30% of samples; remaining missing values are median-imputed
- Both networks are filtered at the identical edge-strength threshold (absolute correlation coefficient >= 0.2) before any density comparison
- The DNEA edge list imports into Cytoscape v3.10.0 via the getNetworkFiles() output without errors
- DNEA lambda is optimized via the BIC tuning protocol and stability selection (e.g. 1000 resampling reps) is run before network estimation

## Limitations

- Marginal-correlation networks are very dense due to inclusion of edges representing both direct and indirect correlations; the latter can confound identification of biologically relevant interactions
- The BIC tuning protocol for DNEA's lambda requires a large dataset (~500 or more samples per group); smaller datasets should use the lambda approximation of Eq. 3
- DNEA requires regularization when the number of metabolites exceeds the number of biological samples (p >> n), and network density depends heavily on subsampling (T1D network: 26,888 edges at 100% of samples vs 1,354 at 5%)

## Evidence

- [discussion] The DNEA, Filigree and Metabox2 networks were filtered to include edges with an absolute value of correlation coefficient greater than or equal to 0.2.: "The DNEA, Filigree and Metabox2 networks were filtered to include edges with an absolute value of correlation coefficient greater than or equal to 0.2."
- [discussion] Marginal-correlation networks include indirect correlations: "they tend to be very dense due to the inclusion of edges representing both direct and indirect correlations. The latter can confound the results"
- [methods] Missingness filter of 30%: "The data were filtered to exclude metabolites that were missing in more than 30% of the samples."
- [readme] GLASSO-based partial correlation estimation: "The package takes as input the raw peak intensity / concentrations for a set of features from two experimental conditions and uses the GLASSO algorithm to estimate metabolite-metabolite partial"
- [methods] filterNetworks threshold parameter in practice: "TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.166)"
