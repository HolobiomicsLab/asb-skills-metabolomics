---
name: partial-correlation-edge-filtering
description: Use when a DNEA (or Filigree) GLASSO network has been estimated for two
  experimental groups and the raw edge list is too dense for interpretation — edges
  must be filtered by partial correlation strength or percentile.
when_to_use_negative:
- The input is a raw peak-intensity feature table that has not yet been through BICtune/stabilitySelection/getNetworks
  — there are no edges to filter yet.
- Pathway-database enrichment (e.g. FELLA/KEGG) is the goal — FELLA required KEGG
  IDs and covered only 19 of 174 TEDDY metabolites; data-driven network filtering
  is not a substitute for pathway mapping.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: DNEA R package
  role: Provides filterNetworks() to filter estimated networks by partial correlation
    or percentile edge strength; edgeList() enumerates filtered edges
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: R
  role: Runtime environment in which the DNEA filtering and edge counting are executed
- name: Filigree
  role: Prior Java implementation whose default networks are filtered with the same
    |pcor| >= 0.2 criterion for cross-tool comparison
- name: Cytoscape v3.10.0
  role: Visualizes the filtered edge list exported via getNetworkFiles()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:8cbbdce01c366b8ac0a70ad7151795b21d12576f8fee322cd235223d0d48968c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape v3.10.0
  - DNEA R package
  - Filigree
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# partial-correlation-edge-filtering

## Summary

Filtering GLASSO-estimated differential metabolite networks with DNEA's filterNetworks() to retain only edges meeting a partial correlation strength threshold (e.g. |pcor| >= 0.2), controlling network density before counting or enrichment analysis.

## When to use

Use when a DNEA (or Filigree) GLASSO network has been estimated for two experimental groups and the raw edge list is too dense for interpretation — edges must be filtered by partial correlation strength or percentile.

## When NOT to use

- The input is a raw peak-intensity feature table that has not yet been through BICtune/stabilitySelection/getNetworks — there are no edges to filter yet.
- Pathway-database enrichment (e.g. FELLA/KEGG) is the goal — FELLA required KEGG IDs and covered only 19 of 174 TEDDY metabolites; data-driven network filtering is not a substitute for pathway mapping.

## Inputs

- DNEA object with estimated networks (output of getNetworks(), e.g. TEDDYdat)
- Partial correlation threshold (e.g. pcor = 0.166 or 0.2) or edge-strength percentile
- Unnormalized peak intensity data (e.g. Metabolomics Workbench ST001386 TEDDY IA last-visit data) used to build the network

## Outputs

- Filtered DNEA object / edge list containing only edges with |partial correlation| >= threshold
- Counts of total and condition-specific (case-specific, control-specific, shared) differential edges

## How to apply

After network construction (BICtune -> stabilitySelection -> getNetworks in the DNEA R package), apply filterNetworks() with either a partial correlation cutoff (e.g. pcor = 0.166 in the methods example, or pcor = 0.2 as used in the comparison against Filigree/Metabox2) or a percentile-based threshold on edge strength. The rationale is that unfiltered GGM networks from p >> n metabolomics data are dense and include unstable edges; filtering controls network density and highlights the most biologically relevant connections. Once filtered, count total and condition-specific (differential) edges via edgeList() — e.g. the BIC-tuned IA network yielded 172 total edges with 139 differential, versus 70 total with 41 differential for the approximated-lambda default, after |pcor| >= 0.2 filtering. Choose the cutoff based on dataset size: BIC-tuned lambda for large datasets (~500+ samples/group) or the lambda approximation sqrt(ln(num features)/num samples) for small ones, since the threshold interacts with lambda choice.

## Related tools

- **DNEA R package** (Provides filterNetworks() to filter estimated networks by partial correlation or percentile edge strength; edgeList() enumerates filtered edges) — http://www.github.com/Karnovsky-Lab/DNEA/
- **R** (Runtime environment in which the DNEA filtering and edge counting are executed)
- **Filigree** (Prior Java implementation whose default networks are filtered with the same |pcor| >= 0.2 criterion for cross-tool comparison)
- **Cytoscape v3.10.0** (Visualizes the filtered edge list exported via getNetworkFiles())

## Examples

```
TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.2)
```

## Evaluation signals

- Filtered network on the matched TEDDY IA case-control subset yields 70 total edges with 41 differential edges for the approximated-lambda (Filigree-default) configuration after |pcor| >= 0.2 filtering.
- BIC-tuned configuration (informed=TRUE, interval=1e-3, stabilitySelection nreps=1000 with subSample=TRUE) yields 172 total edges with 139 differential edges after the same filtering.
- Filtered edge list contains 112 case-specific, 70 control-specific, and 108 shared edges in the unfiltered/subsampled comparison (290 total).
- No edge in the output has absolute partial correlation below the chosen threshold (e.g. 0.2 or 0.166).
- Filtering reduces network density relative to the unfiltered GLASSO output, as expected from the discussion of density control.

## Limitations

- The BIC tuning protocol that determines the lambda feeding into network construction requires large datasets (~500 or more samples per group); for smaller datasets the lambda approximation (Eq. 3) should be used instead, which changes the resulting edge counts.
- No software version numbers were provided for DNEA or comparison tools, so exact reproducibility may vary.
- Filtering threshold choice interacts with lambda choice — the 290-edge network (112 T1D-case-specific, 70 control-specific, 108 shared) was produced with an approximated lambda and subsampling, whereas BIC-tuned networks produced different counts.

## Evidence

- [intro] filterNetworks() filters networks based on edge strength: "the filterNetworks() function is available to filter networks based on edge strength (percentile or partial correlation threshold) and highlight the most biologically relevant connections"
- [discussion] Filtering to |pcor| >= 0.2: "The DNEA, Filigree and Metabox2 networks were filtered to include edges with an absolute value of correlation coefficient greater than or equal to 0.2."
- [methods] Methods filterNetworks call: "TEDDYdat_filtered <- filterNetworks(TEDDYdat, pcor = 0.166)"
- [discussion] Density-control rationale: "the implementation of network filtering through the filterNetworks() function that aids interpretation by controlling the network density."
- [discussion] BIC tuning sample-size requirement: "Utilizing the BIC protocol for network construction increases the sensitivity of DNEA, however, a large dataset is required (~ 500 or more samples per group)."
