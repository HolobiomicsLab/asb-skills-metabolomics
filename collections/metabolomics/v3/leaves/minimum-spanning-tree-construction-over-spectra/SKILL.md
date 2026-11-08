---
name: minimum-spanning-tree-construction-over-spectra
description: Reference for constructing an MST over MS/MS spectra whose pairwise modified
  cosine similarity scores are known, when a deterministic ordering of spectra is
  needed for transitive peak alignment within a molecular network component.
when_to_use_negative:
- Input is already a single pairwise spectrum-spectrum alignment result and no multi-spectrum
  consensus is needed
- Spectra lack precursor metadata or peak filtering context (±25 m/z window, ±17 m/z
  precursor exclusion), so the modified cosine similarity matrix cannot be produced
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: NetworkX
  role: Implements the MST construction from the pairwise similarity-derived edge
    weights (version 3.3 used in the article)
- name: MMSA
  role: Web tool whose alignment pipeline consumes the MST and pairwise similarity
    scores to produce transitive peak alignments
  repo: github.com/github.com__Wang-Bioinformatics-Lab__NetworkFamily_MultipleAlignment_Website
- name: GNPS2
  role: Source of molecular network tasks whose component spectra are fetched and
    aligned
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.5c00237
    title: Multiple Spectrum Alignment for Molecular Networking Exploration and Discovery
  source_content_hash: sha256:29836498feabbe76b4ee8038025d422f7c48a3cc226cde4a6cf21a4f1e6c5f3f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/github.com/github.com__Wang-Bioinformatics-Lab__NetworkFamily_MultipleAlignment_Website
  tools:
  - GNPS2
  - MMSA
  - NetworkX
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# minimum-spanning-tree-construction-over-spectra

## Summary

Builds a minimum spanning tree (MST) over MS/MS spectra within a molecular network component, using NetworkX 3.3 and greedy spanning tree logic with edges weighted by modified cosine similarity subtracted from 1, to drive the transitive spectral alignment in MMSA.

## When to use

Reference for constructing an MST over MS/MS spectra whose pairwise modified cosine similarity scores are known, when a deterministic ordering of spectra is needed for transitive peak alignment within a molecular network component.

## When NOT to use

- Input is already a single pairwise spectrum-spectrum alignment result and no multi-spectrum consensus is needed
- Spectra lack precursor metadata or peak filtering context (±25 m/z window, ±17 m/z precursor exclusion), so the modified cosine similarity matrix cannot be produced

## Inputs

- Set of MS/MS spectra within a single molecular network component (e.g. GNPS2 task output, specs_ms.mgf scans referenced by USI)
- Pairwise modified cosine similarity matrix produced by the _cosine_fast algorithm

## Outputs

- Minimum spanning tree over spectra nodes with edge weights = 1 − modified cosine similarity
- Topologically sorted node ordering used to drive transitive alignment
- Transitively aligned peak sets (m/z and intensity per set) and non-transitively aligned peak sets, serialized to a JSON file

## How to apply

Preprocess each MS/MS spectrum by filtering peaks (top 6 most intense peaks within a ±25 m/z window, excluding peaks within ±17 m/z of the precursor ion), applying a square root transformation to intensities, and normalizing. Score all spectrum pairs pairwise with the _cosine_fast modified cosine algorithm to obtain a similarity matrix. Convert pairwise scores to edge weights by subtracting each similarity score from 1 so that more similar spectra yield lower (cheaper) edge weights. Construct the MST over the spectra as nodes using NetworkX 3.3 with a greedy spanning tree approach, since the goal is to connect spectra along the shortest paths based on similarity-derived edge weights. Then apply topological sorting to process MST nodes in a specific order, which determines the sequence in which spectrum pairs undergo transitive alignment (the _cosine_fast transitive alignment producing matched peak sets) and the non-transitively aligned sets are computed afterwards. Serialize the MST structure and aligned peak sets (m/z and intensity per set) to JSON.

## Related tools

- **NetworkX** (Implements the MST construction from the pairwise similarity-derived edge weights (version 3.3 used in the article))
- **MMSA** (Web tool whose alignment pipeline consumes the MST and pairwise similarity scores to produce transitive peak alignments) — github.com/github.com__Wang-Bioinformatics-Lab__NetworkFamily_MultipleAlignment_Website
- **GNPS2** (Source of molecular network tasks whose component spectra are fetched and aligned)

## Examples

```
networkx.minimum_spanning_tree(G, weight='weight')  # edge weight = 1 - modified cosine similarity
```

## Evaluation signals

- Verify the output JSON parses and contains the MST/alignment structure with peak sets annotated with m/z and intensity information for each spectrum in the component
- Check that the MST connects all spectra in the component (spanning property) and that edge weights equal 1 minus the modified cosine similarity
- Confirm that topological sorting produced an ordering under which transitive alignment of spectrum pairs proceeded without cycles
- Inspect the aligned peak sets for chemically meaningful shared fragments (e.g. rhamnose fragment at m/z 135.017 in the rhamnolipid family) to sanity-check alignment quality

## Limitations

- The greedy multiple alignment does not fully capture smaller peak alignments, such as y2 peaks in the albicidin family
- MMSA did not capture shifted peaks corresponding to the C12 HAA (m/z 239.162) or rhamnose bound to C12 (m/z 385.219), showing that low-similarity or shifted-peak alignments may be missed
- Peak filtering (top 6 peaks within ±25 m/z, excluding ±17 m/z of precursor) discards lower-intensity peaks that could contribute to alignment

## Evidence

- [results] Edge weights derived from pairwise modified cosine similarity subtracted from 1: [evidence span withheld — anchor status: not_found]
