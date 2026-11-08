---
name: graphml-graph-manipulation
description: Use when you have an IIMN graphml file whose nodes are redundant ion-species
  features (adducts, isotopes, dimers sharing one IIN ID) and you need a collapsed
  molecular-node graph for downstream analysis or visualization.
when_to_use_negative:
- The graph is already a collapsed molecular network (one node per neutral molecule)
  — re-collapsing would merge unrelated compounds
- The input is a raw feature quantification table or .mgf file that has not been processed
  through the IIMN workflow to produce IIN IDs
- You need to modify the upstream MS1 ion identity grouping (e.g., correlation thresholds)
  rather than post-process the network graph
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: networkx
  role: Python library used to load and parse the IIMN graphml, redirect edges, and
    write the collapsed iimn_collapsed.graphml
- name: GNPS
  role: Environment that produced the IIMN graphml via Feature-Based Molecular Networking;
    supplies GNPS library match scores used to select representative nodes
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: mzmine
  role: Source of the aligned feature list, IIN results, and feature quantification
    table that feed the IIMN graph
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:9f50b98873082397e6a19325964c698b38bdedccaaf7846199955bb5b5b9c6b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCMS-UCSD/GNPS_Workflows
  tools:
  - GNPS
  - mzmine
  - networkx
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# graphml-graph-manipulation

## Summary

Load, transform, and re-serialize molecular network graphs in GraphML format — specifically collapsing ion identity networks (IIN) so that multiple ion-species feature nodes of one compound merge into a single neutral-molecule (M) representative node, with per-ion attributes and summed intensities attached.

## When to use

Use when you have an IIMN graphml file whose nodes are redundant ion-species features (adducts, isotopes, dimers sharing one IIN ID) and you need a collapsed molecular-node graph for downstream analysis or visualization.

## When NOT to use

- The graph is already a collapsed molecular network (one node per neutral molecule) — re-collapsing would merge unrelated compounds
- The input is a raw feature quantification table or .mgf file that has not been processed through the IIMN workflow to produce IIN IDs
- You need to modify the upstream MS1 ion identity grouping (e.g., correlation thresholds) rather than post-process the network graph

## Inputs

- IIMN graphml file (ion identity molecular network) with node attributes: IIN ID, GNPS library match score, per-sample intensities
- Edge set with MS2 cosine-similarity molecular-networking edges
- Feature quantification table (.csv) and IIN supplementary edges files (.csv) exported from MZmine/MS-DIAL/XCMS via GNPS FBMN

## Outputs

- iimn_collapsed.graphml — collapsed IIN graph with one representative molecular node per IIN ID, redirected edges, and summed intensity attributes
- Mapping of each IIN ID to its single collapsed representative node

## How to apply

Load the IIMN graphml from the 24-dataset GNPS/MZmine processing with networkx and parse node attributes (IIN ID, library match score, per-sample intensities). Group ion nodes by their shared IIN ID and pick a representative node: the member with the highest GNPS library match score, falling back to the node with maximum summed abundance when no member has a library match. Redirect every MS2 cosine-similarity molecular-networking edge of non-representative members to the representative; when redirected edges duplicate an existing pair, keep the one with the highest cosine score. Attach per-ion identity attributes and intensities summed across all merged ion species (per-sample and summed columns) to each representative, then write the collapsed graph as iimn_collapsed.graphml preserving remaining node/edge attributes. Rationale: IIMN links ion identities of the same compound by MS1 characteristics and structurally similar compounds by MS2 spectral similarity, so collapsing the MS1 layer reduces the redundancy of disconnected ion-species subnetworks (e.g., a 43-feature-node network reduced to four molecular nodes, a 56% reduction).

## Related tools

- **networkx** (Python library used to load and parse the IIMN graphml, redirect edges, and write the collapsed iimn_collapsed.graphml)
- **GNPS** (Environment that produced the IIMN graphml via Feature-Based Molecular Networking; supplies GNPS library match scores used to select representative nodes) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **mzmine** (Source of the aligned feature list, IIN results, and feature quantification table that feed the IIMN graph) — https://github.com/mzmine/mzmine

## Examples

```
import networkx as nx; G = nx.read_graphml('iimn.graphml'); # group nodes by IIN ID, redirect edges to representatives; nx.write_graphml(G, 'iimn_collapsed.graphml')
```

## Evaluation signals

- Every IIN ID maps to exactly one collapsed node in the output graphml
- All redirected edges have valid endpoint nodes present in the output graph
- For any node pair with multiple candidate duplicate edges, the retained edge has the highest cosine similarity score
- Representative-node intensity attributes equal the sum of per-sample intensities across all merged ion species
- Node count in iimn_collapsed.graphml equals the number of distinct IIN IDs; edges preserve original attribute keys

## Limitations

- When no group member has a GNPS library match (minimum cosine similarity of 0.9 with 4–6 shared fragments), representative selection falls back to maximum summed abundance, which may not be the most informative ion species
- Collapsing discards the one-to-one mapping between feature nodes and ion species, so per-adduct information must be kept via attached per-ion identity attributes
- Edge redirection can create duplicate node pairs; resolution depends on the highest-cosine tie-break rule and original edge attribute provenance is lost for removed duplicates

## Evidence

- [full_text] Collapse of IIN to representative molecular nodes: [evidence span withheld — anchor status: not_found]
- [intro] Two connectivity layers motivate collapse: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds"
- [discussion] Upstream export formats feeding the graph: [evidence span withheld — anchor status: not_found]
- [discussion] Where the workflow runs and is documented: "Detailed documentation and training videos are available online (https://ccms-ucsd.github.io/GNPSDocumentation/fbmn-iin/)"
- [readme] GNPS workflows repository hosting FBMN/IIN: [evidence span withheld — anchor status: external_doc]
