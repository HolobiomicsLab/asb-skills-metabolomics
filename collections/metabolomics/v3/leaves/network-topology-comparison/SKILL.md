---
name: network-topology-comparison
description: Use when you need to validate an alternative spectral similarity metric
  by reproducing a published molecular network and checking that known compound-family
  sub-networks are recovered with matching topology.
when_to_use_negative:
- When the goal is spectral library search for structural analogues rather than network
  reconstruction — use TransExION's library-search mode instead of building a similarity
  graph
- When the reference network has no confidently-assigned compound families, since
  there is no sub-network-level ground truth to validate the topology against
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: TransExION
  role: Computes the pairwise MS/MS spectral similarity scores used as molecular network
    edges
  repo: https://github.com/banhdzui/TransExION
- name: meRgeION
  role: Builds the molecular network from the filtered TransExION similarity matrix
    and clusters spectra into molecular families (sub-networks)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:2ff1917d3163630af60e942aafc48455df7c3564836ff7deb215a048c7ae45a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/banhdzui/TransExION
  tools:
  - TransExION
  - meRgeION
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# network-topology-comparison

## Summary

Builds a molecular network from TransExION pairwise MS/MS spectral similarities using meRgeION and compares its sub-network topology against a reference (SNAP-MS/GNPS) network to confirm that confidently-assigned compound families are recovered.

## When to use

Use when you need to validate an alternative spectral similarity metric by reproducing a published molecular network and checking that known compound-family sub-networks are recovered with matching topology.

## When NOT to use

- When the goal is spectral library search for structural analogues rather than network reconstruction — use TransExION's library-search mode instead of building a similarity graph
- When the reference network has no confidently-assigned compound families, since there is no sub-network-level ground truth to validate the topology against

## Inputs

- MGF file of experimental MS/MS spectra (e.g. 2161 spectra from a 925-sample marine bacteria extract library)
- Pre-trained TransExION model file (e.g. from Zenodo record 8175528)
- meRgeION network-building parameters from the reference manuscript (topK, maximum molecular family size)
- mergedGNPS spectral library for node annotation
- Reference molecular network with confidently-assigned compound-family sub-networks (e.g. SNAP-MS GNPS network, Fig. 10)

## Outputs

- All-pairs TransExION spectral similarity matrix (filtered to edges > 0.6)
- TransExION-meRgeION molecular network with molecular families (sub-networks)
- Per-node analogue annotations (top three library analogues with score > 0.6)
- Topology comparison showing which compound-family sub-networks are recovered versus the reference network

## How to apply

Compute all-pairs spectral similarities with TransExION by first converting the MGF spectra (e.g. the 2161 spectra from the 925-sample marine bacteria extract library) to h5py via ms_convert_mgf2h5py.py, then running ms_test_spectral_similarity.py with every query spectrum paired against every reference spectrum (leave --pairs empty). Filter the similarity matrix keeping only edges above 0.6, without requiring a minimum number of matched peaks. Import the filtered matrix into meRgeION and build the molecular network keeping the original manuscript's parameters, such as topK and maximum molecular family size. Annotate each node by analogue search against the mergedGNPS spectral library, retaining the top three analogues per node with similarity score above 0.6. Finally, compare the resulting network against the reference network (e.g. the original SNAP-MS GNPS network) to confirm all confidently-assigned compound-family sub-networks are captured and that sub-network topology matches the published figure.

## Related tools

- **TransExION** (Computes the pairwise MS/MS spectral similarity scores used as molecular network edges) — https://github.com/banhdzui/TransExION
- **meRgeION** (Builds the molecular network from the filtered TransExION similarity matrix and clusters spectra into molecular families (sub-networks))

## Examples

```
python ms_convert_mgf2h5py.py --mgf bacteria_extracts.mgf --out bacteria_extracts.db; python ms_test_spectral_similarity.py --db bacteria_extracts.db --query bacteria_extracts.db --model GNPS_MassBank.ms.model --output similarity_result.csv
```

## Evaluation signals

- All seven compound-family sub-networks confidently assigned by SNAP-MS in the original paper are present in the TransExION-meRgeION network
- Sub-network topology matches Fig. 10 of the original SNAP-MS GNPS network when compared node-by-node
- Every edge in the imported similarity matrix exceeds the 0.6 threshold, with no minimum matched-peak requirement applied
- Each network node carries analogue annotations from mergedGNPS with up to three analogues per node, all with similarity score > 0.6

## Limitations

- Similarity edges are kept above 0.6 without a minimum matched-peak filter, which may retain low-information edges; parameters must be kept identical to the original manuscript (topK, maximum molecular family size) for a fair topology comparison
- A sub-network captured by TransExION does not guarantee identical node composition or edge weights to the reference SNAP-MS network — only family-level recovery was validated

## Evidence

- [discussion] Using the pair-wise spectral similarities computed by TransExION, we built a molecular network of bacteria extracts: "Using the pair-wise spectral similarities computed by TransExION, we built a molecular network of bacteria extracts"
- [discussion] retrieved sub-networks representing confidently-identified chemical families: "retrieved sub-networks representing confidently-identified chemical families"
- [readme] If the parameter --pairs is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity.: "If the parameter ```--pairs``` is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity."
- [readme] To work with large scale dataset, the input data files are converted into h5py format.: "To work with large scale dataset, the input data files are converted into h5py format."
