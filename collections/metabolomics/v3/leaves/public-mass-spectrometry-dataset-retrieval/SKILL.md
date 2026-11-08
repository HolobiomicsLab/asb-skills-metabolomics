---
name: public-mass-spectrometry-dataset-retrieval
description: Use when an analysis must start from a large public untargeted metabolomics
  network (e.g., the ~8.4M-node GNPS/MassIVE molecular network with cosine-similarity
  edges) rather than locally acquired MS/MS files.
when_to_use_negative:
- The analysis uses locally acquired raw MS/MS files (e.g., mzML) that must be processed
  into spectra first — this skill assumes an already-networked public dataset.
- The target dataset is a small study-specific network rather than the global GNPS/MassIVE
  molecular network, where the reported 8.4M-node / 57.4%-singleton reference statistics
  do not apply.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: molecular_communities
  role: Companion repository whose analyses consume the retrieved GNPS/MassIVE network;
    the retrieval step supplies the node/edge tables the MCN pipeline reconstructs
    and evaluates.
  repo: Alexander0/molecular_communities
- name: networkx
  role: Used immediately after retrieval to load the cosine-similarity edge list and
    build the weighted molecular similarity graph over the ~8.4M nodes.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:ea7d0c3738e78d671c469f8f9b9f2d290e2bc1b4a51865cb6a457f0566229b26
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - molecular_communities
  - networkx
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# public-mass-spectrometry-dataset-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves the public GNPS/MassIVE global molecular network dataset — nodes carrying tandem MS spectra and edges carrying cosine similarity scores — as the starting substrate for molecular community networking (MCN) analyses and their reproduction against reported statistics.

## When to use

Use when an analysis must start from a large public untargeted metabolomics network (e.g., the ~8.4M-node GNPS/MassIVE molecular network with cosine-similarity edges) rather than locally acquired MS/MS files.

## When NOT to use

- The analysis uses locally acquired raw MS/MS files (e.g., mzML) that must be processed into spectra first — this skill assumes an already-networked public dataset.
- The target dataset is a small study-specific network rather than the global GNPS/MassIVE molecular network, where the reported 8.4M-node / 57.4%-singleton reference statistics do not apply.

## Inputs

- Public GNPS/MassIVE global molecular network dataset (node table with tandem MS spectra)
- Edge list with cosine similarity scores as edge weights

## Outputs

- Downloaded node and edge tables for the GNPS/MassIVE network
- Weighted networkx graph over ~8.4M molecular nodes with cosine-similarity edge weights
- Baseline statistics of the conventional cosine-0.7 network (e.g., 57.4% singleton fraction) for downstream comparison

## How to apply

Download the public GNPS/MassIVE global molecular network, which consists of molecular nodes each carrying a tandem MS spectrum and edges each carrying a cosine similarity score. Load the edge list and build a weighted molecular similarity graph with networkx, keeping the cosine score as the edge weight so downstream steps (Louvain community detection, maximum-weight spanning trees within communities) can operate on similarity strength. Validate the retrieval by checking node count (~8.4M molecular nodes) and confirming the conventional cosine-0.7 subnetwork reproduces the reported baseline statistics (57.4% unconnected singletons) before applying any new networking method. The rationale is that MCN claims about rescuing 'hidden' connections and reducing singletons from 57.4% to 4.9% are statements about this specific public network, so any reproduction must use the identical node/edge source.

## Related tools

- **molecular_communities** (Companion repository whose analyses consume the retrieved GNPS/MassIVE network; the retrieval step supplies the node/edge tables the MCN pipeline reconstructs and evaluates.) — Alexander0/molecular_communities
- **networkx** (Used immediately after retrieval to load the cosine-similarity edge list and build the weighted molecular similarity graph over the ~8.4M nodes.)

## Examples

```
import networkx as nx; G = nx.read_weighted_edgelist('gnps_massive_edges.tsv')  # weighted molecular similarity graph from retrieved GNPS/MassIVE edge list
```

## Evaluation signals

- Retrieved node count is approximately 8.4M molecular nodes as reported for the global GNPS/MassIVE network.
- Every retained edge carries a cosine similarity score usable as a weight for maximum-weight spanning trees.
- The conventional cosine-0.7 subnetwork derived from the retrieved data reproduces the published 57.4% singleton fraction before any MCN modification.
- After downstream MCN reconstruction on the retrieved data, ~95% (~8M of ~8.4M) of nodes are linked and the singleton fraction drops to 4.9%, with ~3.5M additional nodes connected to an annotated node.
- Reproduced statistics are serialized (e.g., mcn_network_statistics.csv) and diffed against the reported values.

## Limitations

- Conventional molecular networking's arbitrary similarity thresholds can fracture natural metabolite families, so the retrieved conventional network is a baseline to be compared against, not the final structure.
- Claims such as ~3.5M additional annotated-connected nodes are specific to the global GNPS/MassIVE network and will not transfer to smaller or different datasets.
- No changelog was found in the repository documentation, so dataset version drift between retrieval dates is not explicitly documented.

## Evidence

- [full_text] Download the public GNPS/MassIVE global molecular network dataset (nodes with tandem MS spectra and edges with cosine similarity scores).: [evidence span withheld — anchor status: not_found]
- [results] reducing unconnected singleton nodes from 57.4% (conventional) to 4.9% (MCN): [evidence span withheld — anchor status: not_found]
