---
name: graphml-network-export
description: Use when an NP3 MS Workflow analysis has produced pairwise ionization-variant
  or spectral-similarity links that must be persisted as a reproducible GraphML network
  for Cytoscape inspection and [M+H]+ representative-ion reporting.
when_to_use_negative:
- Input is a plain feature/quantification table with no pairwise ionization-variant
  or spectral-similarity relationships to represent as a graph.
- The downstream visualization tool cannot load GraphML — the export convention here
  assumes Cytoscape-compatible output.
- The task requires annotated library identifications or bioactivity correlation scores
  (BCC) rather than network topology export.
edam_operation: http://edamontology.org/operation_3927
tools:
- name: NP3 MS Workflow
  role: Generates the IVAMN pairwise variant links and PageRank [M+H]+ selections
    that are serialized to GraphML; enforces the reproducibility ordering convention
    on export
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Cytoscape
  role: Graph visualization tool used to open and inspect the exported GraphML network
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:ad96134bb59da798f271001656ae4054396a00fbf0f21c4c4a9b32ec31a293d4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielatrivella/NP3_MS_Workflow
  repo_url: https://github.com/danielatrivella/NP3_MS_Workflow
  tools:
  - Cytoscape
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# graphml-network-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Writes NP3 MS Workflow molecular networks (e.g., the IVAMN used to group in-source/adduct ionization variants of a precursor) as well-formed GraphML files with [M+H]+ PageRank selections encoded as node/edge attributes, loadable in Cytoscape.

## When to use

Use when an NP3 MS Workflow analysis has produced pairwise ionization-variant or spectral-similarity links that must be persisted as a reproducible GraphML network for Cytoscape inspection and [M+H]+ representative-ion reporting.

## When NOT to use

- Input is a plain feature/quantification table with no pairwise ionization-variant or spectral-similarity relationships to represent as a graph.
- The downstream visualization tool cannot load GraphML — the export convention here assumes Cytoscape-compatible output.
- The task requires annotated library identifications or bioactivity correlation scores (BCC) rather than network topology export.

## Inputs

- Per-sample MGF files of clean consensus spectra from NP3 MS Workflow steps 3–5 (clustered with the NP3-shifted cosine function)
- Quantification table of precursor ions for the LC–MS/MS data set (e.g., MSV000080502 Euphorbia dendroides extract)
- Ionization rules table encoding expected positive-mode mass shifts for in-source fragments, adducts, and dimers
- Pairwise ionization-variant links from the Annotation/Merging steps 7–8

## Outputs

- GraphML network file (e.g., ivamn_msv000080502.graphml) with [M+H]+ PageRank selections encoded as node/edge attributes
- Components written largest-first and edges sorted by source node ID for reproducibility

## How to apply

Build the network graph from the pairwise links produced by NP3 MS Workflow's Annotation/Merging steps 7–8, where consensus spectra whose m/z differences and fragmentation relationships match an entry in the ionization rules table are connected as putative ionization variants. Run PageRank on the IVAMN to select the [M + H]+ representative ion for each variant group, then write the graph to GraphML encoding the [M+H]+ selections as node/edge attributes. To guarantee reproducibility (per NP3 MS Workflow v1.4.2), write components largest-first and sort edges by source node ID in numerical ordering. The resulting GraphML follows the same convention as the spectral similarity molecular network (SSMN), which can be opened in graph visualization tools such as Cytoscape. Note that if a noise cutoff is applied before export, reset the filtered IVAMN index to prevent matching errors in later filters (bug fixed in v1.5.1 of join_jobs).

## Related tools

- **NP3 MS Workflow** (Generates the IVAMN pairwise variant links and PageRank [M+H]+ selections that are serialized to GraphML; enforces the reproducibility ordering convention on export) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Cytoscape** (Graph visualization tool used to open and inspect the exported GraphML network)

## Evaluation signals

- The GraphML file (e.g., ivamn_msv000080502.graphml) exists and is a well-formed GraphML document loadable in Cytoscape.
- Every ionization-variant cluster contains exactly one PageRank-selected [M + H]+ representative ion.
- Components are written largest-first and edges are sorted by source node ID in numerical ordering, matching the v1.4.2 reproducibility convention.
- Round-trip reproducibility: re-running the export yields byte-identical GraphML, comparable to the NP3 fixed_results equality tests.
- Each edge/variant link corresponds to a match against an entry in the ionization rules table (in-source fragment, adduct, or dimer mass shift).

## Limitations

- Prior to NP3 MS Workflow v1.4.2 the IVAMN component/edge ordering was not reproducible; older outputs may not reproduce byte-identically.
- In join_jobs (before v1.5.1), applying a noise cutoff to a filtered IVAMN without resetting its index caused matching errors in subsequent filters when joining IVAMNs.
- The [M+H]+ deconvolution and IVAMN grouping rely on positive-mode ionization rules; negative-mode or unmodeled ion species may not be grouped.

## Evidence
