---
name: hdf5-large-scale-graph-serialization
description: Decision support for persisting tens of millions of MS/MS spectra or
  nearest-neighbor graph structures (nodes, edges, similarity scores) when flat text
  or MGF files become impractical.
when_to_use_negative:
- When the data volume is small (e.g. a single MGF file or a few thousand spectra)
  and plain MGF/MSP or CSV suffices.
- When the consumer is a tool that expects mzML/MGF and cannot read HDF5-based structures.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0121
tools:
- name: HDF5
  role: underlying storage container for the compact binary format designed for deep
    learning
- name: PyNNDescent
  role: produces the 3-NN graph whose edges and similarity scores are serialized into
    the Atlas HDF5 file
  repo: https://github.com/lmcinnes/pynndescent
- name: DreaMS
  role: generates the embeddings stored per spectrum and the Atlas node annotations
  repo: https://github.com/pluskal-lab/DreaMS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:753624d4036489cef54671ebc2b98f98b6ba1e8940dc8ea9ba46570d83fcf7d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lmcinnes/pynndescent
  tools:
  - DreaMS
  - HDF5
  - MoNA
  - NIST20
  - PyNNDescent
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hdf5-large-scale-graph-serialization

## Summary

Serialize repository-scale molecular networks and spectrum collections into a compact HDF5-based binary format designed for deep learning, as done for the 76-million-spectrum GeMS-C collection and the 34-million-node DreaMS Atlas.

## When to use

Decision support for persisting tens of millions of MS/MS spectra or nearest-neighbor graph structures (nodes, edges, similarity scores) when flat text or MGF files become impractical.

## When NOT to use

- When the data volume is small (e.g. a single MGF file or a few thousand spectra) and plain MGF/MSP or CSV suffices.
- When the consumer is a tool that expects mzML/MGF and cannot read HDF5-based structures.

## Inputs

- DreaMS embeddings (1,024-dim real-valued vectors) for GeMS-C spectra and merged MoNA/NIST20 library spectra
- 3-NN graph edges with cosine similarity scores from PyNNDescent
- LC–MS/MS metadata (precursor m/z, retention order) per spectrum

## Outputs

- Compact HDF5-based binary file containing spectra, embeddings, metadata
- DreaMS Atlas molecular network stored as nodes, edges, and similarity scores in HDF5

## How to apply

Write DreaMS embeddings (1,024-dimensional real-valued vectors) and selected LC–MS/MS metadata for each GeMS-C spectrum into an HDF5-based binary format purpose-built for deep learning pipelines. When serializing the DreaMS Atlas, store nodes (collapsed from spectra merged at similarity > 0.9 via BFS over the 3-NN graph built with PyNNDescent), edges, and edge similarity scores in the same compact format. The format enables random access to individual spectra and subgraphs during pre-training and Atlas queries without loading the full 201-million-spectrum collection. Keep the serialization step after quality filtering (GeMS-A/B/C) and LSH clustering so only curated spectra occupy disk.

## Related tools

- **HDF5** (underlying storage container for the compact binary format designed for deep learning)
- **PyNNDescent** (produces the 3-NN graph whose edges and similarity scores are serialized into the Atlas HDF5 file) — https://github.com/lmcinnes/pynndescent
- **DreaMS** (generates the embeddings stored per spectrum and the Atlas node annotations) — https://github.com/pluskal-lab/DreaMS

## Examples

```
python -c "import h5py; f=h5py.File('atlas.hdf5','r'); print(len(f['nodes']))"
```

## Evaluation signals

- Node count of the serialized Atlas should be approximately 34 million.
- About 67% of stored edges must carry similarity > 0.8.
- 99.7% of nodes should form a single connected component when loaded back.
- Median shortest path from a random node to the nearest MoNA/NIST20 library node should be six edges.
- Random access to an individual spectrum must return its 1,024-dim embedding without reading the whole file.

## Limitations

- Only spectra surviving GeMS-A/B/C quality filtering and LSH deduplication are serialized; redundant spectra are dropped or capped per cluster.
- The DreaMS Atlas and GeMS releases rely on this custom format, so downstream users must adopt the same HDF5 reader conventions.

## Evidence

- [abstract] HDF5 storage of GeMS spectra: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Atlas construction as 3-NN graph: "The DreaMS Atlas is constructed as an approximate three-nearest-neighbor (3-NN) graph based on GeMS mass spectra"
- [abstract] Atlas connectivity statistic: "99.7% of the nodes form a single connected component"
- [discussion] PyNNDescent role: "the PyNNDescent implementation of NN-Descent by McInnes et al. (https://github.com/lmcinnes/pynndescent)"
