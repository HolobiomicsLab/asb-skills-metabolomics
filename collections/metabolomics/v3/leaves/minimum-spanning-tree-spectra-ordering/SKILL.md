---
name: minimum-spanning-tree-spectra-ordering
description: Use when you have a molecular network component's MS/MS spectra (GNPS2
  task/component or USIs) and need a deterministic ordering for transitive multiple-spectrum
  alignment in MMSA.
when_to_use_negative:
- You only need pairwise spectrum comparison or standard GNPS molecular networking
  without multiple-spectrum consensus alignment
- Input is already a consensus or aligned peak set rather than raw component MS/MS
  spectra
- The component has fewer than two spectra so no MST ordering is required
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NetworkX 3.3
  role: Constructs the minimum spanning tree over component spectra weighted by 1
    − modified cosine similarity, then provides topological sorting of nodes for MMSA
    processing order
- name: MMSA
  role: Consumes the MST ordering to run transitive pairwise alignment and compute
    non-transitively aligned consensus peak sets
- name: GNPS2
  role: Source of molecular-network component spectra via task ID and component number
- name: Python Dash
  role: Web interface where 'Process and Save JSON' persists the MST-derived alignment
    and the visualization is exposed
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:c50a3c4d0630a5fef1a25b879bcd829c76f82c8366758aac1d28b03b5304af83
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GNPS2
  - MMSA
  - NetworkX
  - NetworkX 3.3
  - Python Dash
  - metabolomics resolver
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# minimum-spanning-tree-spectra-ordering

## Summary

Orders MS/MS spectra within a molecular-networking component by building a minimum spanning tree (MST) with NetworkX 3.3, weighting edges as 1 minus pairwise modified cosine similarity, then topologically sorting nodes to fix the processing order for MMSA spectral alignment.

## When to use

Use when you have a molecular network component's MS/MS spectra (GNPS2 task/component or USIs) and need a deterministic ordering for transitive multiple-spectrum alignment in MMSA.

## When NOT to use

- You only need pairwise spectrum comparison or standard GNPS molecular networking without multiple-spectrum consensus alignment
- Input is already a consensus or aligned peak set rather than raw component MS/MS spectra
- The component has fewer than two spectra so no MST ordering is required

## Inputs

- MS/MS spectra from a molecular network component (via GNPS2 task ID + component number, USI list of the form mzspec:GNPS2:TASK-<task-id>-nf_output/clustering/specs_ms.mgf:scan:<scan-number>, or FBMN task ID)
- Preprocessed spectra (top-6 peaks in ±25 m/z window, precursor ±17 m/z exclusion, sqrt-transformed normalized intensities)
- Pairwise modified cosine similarity scores for all spectrum pairs

## Outputs

- Minimum spanning tree over spectra with edge weights of 1 − modified cosine similarity
- Topologically sorted processing order of spectra for transitive alignment
- Alignment JSON from 'Process and Save JSON' feeding the interactive peak-alignment visualization

## How to apply

After MMSA preprocessing (top-6 peaks within ±25 m/z excluding peaks within ±17 m/z of the precursor, square-root transform, intensity normalization), compute a pairwise modified cosine similarity score for every spectrum pair. Build a minimum spanning tree over the spectra using NetworkX 3.3, with each edge weighted by the pairwise modified cosine similarity score subtracted from 1, so that more-similar spectra sit on shorter paths. Apply topological sorting to process the nodes in a specific order before running the transitive pairwise alignment, then derive the non-transitively aligned consensus peak sets. The MST structure connects nodes along shortest paths based on edge weights, guaranteeing that each transitive step aligns the most-similar spectrum pair first; click 'Process and Save JSON' in the Dash interface to persist the alignment output.

## Related tools

- **NetworkX 3.3** (Constructs the minimum spanning tree over component spectra weighted by 1 − modified cosine similarity, then provides topological sorting of nodes for MMSA processing order)
- **MMSA** (Consumes the MST ordering to run transitive pairwise alignment and compute non-transitively aligned consensus peak sets)
- **GNPS2** (Source of molecular-network component spectra via task ID and component number)
- **Python Dash** (Web interface where 'Process and Save JSON' persists the MST-derived alignment and the visualization is exposed)

## Examples

```
import networkx as nx; G = nx.minimum_spanning_tree(nx.complete_graph(N)); order = list(nx.topological_sort(G))
```

## Evaluation signals

- Edge weights in the exported graph equal 1 minus the pairwise modified cosine similarity for each spectrum pair
- Topological sort yields a unique deterministic node order before transitive alignment runs
- Reproduced alignment JSON for the rhamnolipid family contains the consensus peaks m/z 135.017, 211.130, 357.189 and the shifted acetylated-rhamnose peak m/z 399.199
- SVG export of the aligned-spectra visualization can be generated from the MST ordering
- C12-containing spectra correctly fail to align with C10-derived peak sets (m/z 239.162 and 385.219 absent), matching reported MMSA behavior

## Limitations

- Greedy transitive alignment via the MST can miss smaller peak alignments, e.g. y2 peaks in the albicidin family are not fully captured
- Shifted C12 peaks (m/z 239.162, 385.219) fail to align with C10-derived peak sets, leaving C12-containing rhamnolipid spectra unaligned
- Phantom peaks can appear in C12 and R(OAc) consensus spectra arising from the greedy alignment
- The README shows no public repository URL; only the local directory NetworkFamily_MultipleAlignment_Website is named, and no DOI or Zenodo archive is given

## Evidence

- [results] weighting edges by pairwise modified cosine similarity scores subtracted from 1: [evidence span withheld — anchor status: not_found]
- [results] MMSA failed to capture the shifted peaks for the C12 HAA (m/z 239.162) and rhamnose bound to C12 (m/z 385.219), leaving the C12-containing rhamnolipid spectra unaligned: [evidence span withheld — anchor status: not_found]
