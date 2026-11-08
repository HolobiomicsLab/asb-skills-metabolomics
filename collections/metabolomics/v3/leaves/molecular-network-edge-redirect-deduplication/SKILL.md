---
name: molecular-network-edge-redirect-deduplication
description: Use when you have an IIMN graphml with many ion-species nodes sharing
  a common IIN ID (adducts/isotopes/in-source fragments of one compound) and need
  a collapsed molecular-node network without redundant edges.
when_to_use_negative:
- Input is a raw feature quantification table (.csv) or .mgf spectra file rather than
  an already-built IIMN graphml — build the ion identity network first via GNPS FBMN
- The goal is to connect ion species rather than collapse them (e.g., revealing ion-ligand
  complexes) — use the full two-layer IIMN output instead
- Nodes do not share IIN IDs (no MS1-based ion identity grouping was run), so no collapse
  grouping is possible
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GNPS
  role: Source of the IIMN graphml and molecular networking (MS2 cosine similarity)
    edges; library match scores used to pick representative nodes
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: mzmine
  role: Upstream feature finding, MS1-based ion identity grouping (IIN ID assignment)
    and export of the feature quantification table and IIN edges files consumed here
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:eb8a9ea0a3db2e1832caf8a75c131285d9208580cf894415a498871578f0dd38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - mzmine
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# molecular-network-edge-redirect-deduplication

## Summary

Collapses an Ion Identity Molecular Network (IIMN) by merging the multiple ion-species feature nodes of each compound into a single neutral-molecule representative node, redirecting and de-duplicating MS2 similarity edges. Reduces network redundancy so downstream analysis works on molecular nodes rather than per-ion feature nodes.

## When to use

Use when you have an IIMN graphml with many ion-species nodes sharing a common IIN ID (adducts/isotopes/in-source fragments of one compound) and need a collapsed molecular-node network without redundant edges.

## When NOT to use

- Input is a raw feature quantification table (.csv) or .mgf spectra file rather than an already-built IIMN graphml — build the ion identity network first via GNPS FBMN
- The goal is to connect ion species rather than collapse them (e.g., revealing ion-ligand complexes) — use the full two-layer IIMN output instead
- Nodes do not share IIN IDs (no MS1-based ion identity grouping was run), so no collapse grouping is possible

## Inputs

- IIMN graphml file from GNPS feature-based molecular networking (nodes: ion-species features with IIN ID, library match score, per-sample intensity attributes; edges: MS2 cosine-similarity molecular networking edges)
- IIN supplementary edges CSV files (.csv) mapping ion identities to shared IIN IDs

## Outputs

- iimn_collapsed.graphml: collapsed IIN graph with one neutral-molecule (M) representative node per IIN ID, redirected de-duplicated MS2 edges, per-ion identity attributes, and summed per-sample intensities

## How to apply

Load the IIMN graphml produced by the 24-dataset feature-based molecular networking processing with networkx and parse node attributes (IIN ID, library match score, per-sample intensities). Group ion nodes by their shared IIN ID and select a representative node per group: the node with the highest GNPS library match score, falling back to the node with maximum summed abundance when no member has a library match. Redirect every molecular-networking (MS2 cosine similarity) edge of non-representative members to the representative node; when redirected edges duplicate an existing pair, retain only the edge with the highest cosine similarity score. Attach to each representative node the per-ion identity attributes and intensities summed across all merged ion species (including per-sample and summed intensity columns), then write the collapsed graph as iimn_collapsed.graphml preserving remaining node/edge attributes. The rationale follows IIMN's design of collapsing different ion species of the same molecule to remove redundant, disconnected sub-networks.

## Related tools

- **GNPS** (Source of the IIMN graphml and molecular networking (MS2 cosine similarity) edges; library match scores used to pick representative nodes) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **mzmine** (Upstream feature finding, MS1-based ion identity grouping (IIN ID assignment) and export of the feature quantification table and IIN edges files consumed here) — https://github.com/mzmine/mzmine

## Examples

```
import networkx as nx; G = nx.read_graphml('iimn.graphml'); groups = group_by_iin(G); H = collapse_iin(G, groups); nx.write_graphml(H, 'iimn_collapsed.graphml')
```

## Evaluation signals

- Every IIN ID maps to exactly one collapsed (representative) node in the output graph
- All redirected edges keep valid endpoint nodes (no dangling endpoints after collapse)
- For each node pair with duplicated redirected edges, the retained edge is the one with the highest cosine similarity score
- Representative node intensities equal the sum across all merged ion species, matching per-sample and summed intensity columns
- Network complexity drops relative to the input graph (IIMN reported e.g. a 43-feature-node network reduced to four molecular nodes, a 56% reduction)

## Limitations

- Collapse depends on correct upstream MS1 ion identity grouping; edges are only redirected within a shared IIN ID group, so misgrouped ion species are not merged
- Per-node edge multiplicity is lost: only the highest-cosine edge per node pair is retained, discarding lower-scoring alternative edges
- Representative-node choice depends on GNPS library match scores; groups with no library match fall back to summed abundance, which may not reflect the most informative ion species
- The output preserves only remaining node/edge attributes; ion-species-level topology within a group is no longer distinguishable in the collapsed graph

## Evidence

- [abstract] IIMN collapse connects ion species: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [intro] two-layer IIMN connectivity: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics"
- [discussion] IIN edges files exported for network building: [evidence span withheld — anchor status: not_found]
- [intro] IIMN reduces feature redundancy: "IIMN addresses this central bottleneck of unconnected ion adducts in MN and the general problem of feature redundancy in MS-based metabolomics"
