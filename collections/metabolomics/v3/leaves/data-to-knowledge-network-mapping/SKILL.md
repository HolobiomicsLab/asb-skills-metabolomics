---
name: data-to-knowledge-network-mapping
description: Use when you have a GNN-curated reaction network plus MS1 feature table
  and MS2 spectra, and need to constrain feature-feature edges by modified dot-product
  MS2 similarity (≥ 0.5) before annotation propagation.
when_to_use_negative:
- Input data lacks MS2 spectra for features — modified dot-product similarity between
  reaction-linked pairs cannot be computed
- The data-constrained MRN is already built (filtered pair table exists as table_ms2_network_filter)
  and the goal is downstream annotation propagation via getMetAnno
- You are running the older single-layer MetDNA2/KGMN workflow rather than the MetDNA3
  two-layer topology
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetDNA3
  role: Core algorithm package (MrnAnnoAlgo3) providing getMs1Data4Mrn3, getMs2Data4Mrn3,
    getTableSeedLong4Mrn3, getMrnNeighbor, getMatchParam, and getMetAnno used in the
    filtering and mapping workflow
  repo: https://github.com/ZhuMetLab/MrnAnnoAlgo3
- name: SpectraTools
  role: Computes the modified dot-product MS2 spectral similarity scores used to filter
    reaction-linked feature pairs
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: igraph
  role: Constructs and manipulates the feature network / data-constrained MRN graph
    structure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-63536-6
    title: Knowledge and data-driven two-layer networking for accurate metabolite
      annotation in untargeted metabolomics
  source_content_hash: sha256:08922b561790a467988177b1f83b6c29aa0a297c2930bd2c84f5b2e1c17034f6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  repo_url: https://github.com/ZhuMetLab/MrnAnnoAlgo3
  tools:
  - MetDNA3
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# data-to-knowledge-network-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters a knowledge-layer metabolic reaction network (MRN) by experimental MS2 spectral similarity between reaction-linked feature pairs, producing a data-constrained MRN used as the scaffold for MetDNA3's two-layer annotation propagation.

## When to use

Use when you have a GNN-curated reaction network plus MS1 feature table and MS2 spectra, and need to constrain feature-feature edges by modified dot-product MS2 similarity (≥ 0.5) before annotation propagation.

## When NOT to use

- Input data lacks MS2 spectra for features — modified dot-product similarity between reaction-linked pairs cannot be computed
- The data-constrained MRN is already built (filtered pair table exists as table_ms2_network_filter) and the goal is downstream annotation propagation via getMetAnno
- You are running the older single-layer MetDNA2/KGMN workflow rather than the MetDNA3 two-layer topology

## Inputs

- MS1 peak/feature table (.csv; first three columns 'name', 'mz', 'rt')
- MS2 spectra files (.mgf or .msp)
- Seed metabolite annotation table (getTableSeedLong4Mrn3)
- GNN-curated knowledge-layer metabolic reaction network (reaction-linked feature pairs)

## Outputs

- Data-constrained MRN edge table (data_constrained_mrn_edges.csv) containing 2,993 metabolites and 55,674 reaction pairs for NIST urine HILIC-MS(+)
- Filtered pair table passed to getMetAnno as table_ms2_network_filter

## How to apply

Load the MS1 feature table, MS2 spectra, and seed annotation table (e.g., NIST urine HILIC-MS(+) demo data via getMs1Data4Mrn3, getMs2Data4Mrn3, getTableSeedLong4Mrn3). Retrieve reaction-linked feature pairs from the GNN-curated knowledge-layer MRN with getMrnNeighbor. Compute the modified dot-product MS2 spectral similarity between each reaction-linked pair using getMatchParam with SpectraTools, then retain only edges with similarity score ≥ 0.5 — this is the third pre-mapping constraint, applied after sequential MS1 m/z matching and reaction relationship mapping. Pass the filtered pair table into getMetAnno as table_ms2_network_filter to map the data-constrained network back onto the knowledge layer. The rationale is that experimental MS2 similarity eliminates unwanted nodes from the reaction-mapped feature network, reducing false positives from the GNN-predicted reaction pairs before recursive annotation propagation.

## Related tools

- **MetDNA3** (Core algorithm package (MrnAnnoAlgo3) providing getMs1Data4Mrn3, getMs2Data4Mrn3, getTableSeedLong4Mrn3, getMrnNeighbor, getMatchParam, and getMetAnno used in the filtering and mapping workflow) — https://github.com/ZhuMetLab/MrnAnnoAlgo3
- **SpectraTools** (Computes the modified dot-product MS2 spectral similarity scores used to filter reaction-linked feature pairs) — https://github.com/ZhuMetLab/SpectraTools
- **igraph** (Constructs and manipulates the feature network / data-constrained MRN graph structure)

## Examples

```
devtools::install_github("ZhuMetLab/MrnAnnoAlgo3")
```

## Evaluation signals

- For NIST urine HILIC-MS(+), the exported data-constrained MRN must contain exactly 2,993 metabolites and 55,674 reaction pairs, matching the Source Data file for Fig. 2c–d
- Every retained edge has modified dot-product MS2 similarity score ≥ 0.5
- The filtered pair table is passed as the table_ms2_network_filter argument of getMetAnno, confirming the constraint was applied before annotation propagation
- The edge table data_constrained_mrn_edges.csv exports with a well-formed edge list schema consumable by igraph

## Limitations

- Direct GNN prediction of reaction pairs may include a considerable number of false positives, which motivates but does not fully resolve network noise
- No FDR is reported for the curated MRN due to the lack of clearly defined non-RPs and inapplicability of GNN performance metrics to the filtered candidate set
- MS2 spectra often lack sufficient fragment information and cannot distinguish isomeric metabolites, so MS2-similarity-based filtering may be optimistic

## Evidence
