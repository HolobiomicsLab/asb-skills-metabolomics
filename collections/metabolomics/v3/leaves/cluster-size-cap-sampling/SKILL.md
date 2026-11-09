---
name: cluster-size-cap-sampling
description: Use when a large spectrum collection (e.g. GeMS mined from MassIVE GNPS)
  contains redundant near-duplicate spectra and you need size-limited representative
  subsets for training or evaluation.
when_to_use_negative:
- The GeMS variants are already deduplicated and you need raw, unfiltered spectrum
  counts instead of representative subsets
- You need an exact similarity structure or nearest-neighbor graph (e.g. the DreaMS
  Atlas 3-NN graph built with PyNNDescent) rather than linear-time approximate cosine-similarity
  grouping via LSH
- Every redundant spectrum must be retained, since capping by random sampling discards
  non-sampled members of each cluster
edam_operation: http://edamontology.org/operation_3432
edam_topics: []
tools:
- name: HDF5
  role: Serialization format for storing the quality-filtered input subsets and the
    nine capped GeMS variants in the compact HDF5-based binary format designed for
    deep learning
- name: Locality-sensitive hashing (LSH)
  role: Random-projection LSH with 64 hyperplanes approximating cosine similarity
    in linear time; produces the clusters whose sizes are capped
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:d69f0761703d2666baaeece5f36521d29d61a710fafc65f9d5118e399733b9eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  - Locality-sensitive hashing (LSH)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cluster-size-cap-sampling

## Summary

Caps the size of LSH-derived similarity clusters by randomly sampling a fixed maximum number of spectra per cluster, producing deduplicated representative variants of the GeMS MS/MS corpus (e.g. GeMS-A1/A10/A1000) stored in the compact HDF5 binary format.

## When to use

Use when a large spectrum collection (e.g. GeMS mined from MassIVE GNPS) contains redundant near-duplicate spectra and you need size-limited representative subsets for training or evaluation.

## When NOT to use

- The GeMS variants are already deduplicated and you need raw, unfiltered spectrum counts instead of representative subsets
- You need an exact similarity structure or nearest-neighbor graph (e.g. the DreaMS Atlas 3-NN graph built with PyNNDescent) rather than linear-time approximate cosine-similarity grouping via LSH
- Every redundant spectrum must be retained, since capping by random sampling discards non-sampled members of each cluster

## Inputs

- Quality-filtered GeMS-A, GeMS-B and GeMS-C MS/MS spectrum subsets in the compact HDF5-based binary format
- Spectra binned on a 1.0 m/z grid (LSH clustering setup)
- LSH cluster assignments from random-projection LSH with 64 hyperplanes

## Outputs

- Nine capped GeMS variants: GeMS_A1.hdf5, GeMS_A10.hdf5, GeMS_A1000.hdf5, GeMS_B1.hdf5, GeMS_B10.hdf5, GeMS_B1000.hdf5, GeMS_C1.hdf5, GeMS_C10.hdf5, GeMS_C1000.hdf5
- Per-cluster representative spectra sampled uniformly at random up to the cap (1, 10, or 1,000 spectra per cluster)

## How to apply

Start from the quality-filtered GeMS-A, GeMS-B and GeMS-C subsets stored in the compact HDF5-based binary format, and bin each spectrum on a 1.0 m/z grid as specified in the LSH clustering setup. Apply random-projection locality-sensitive hashing with 64 hyperplanes, which approximates cosine similarity in linear time, to group similar spectra into LSH clusters. For each subset, cap every cluster to a maximum of 1, 10, or 1,000 spectra by random sampling within the cluster, yielding three variants per subset. Serialize the nine capped variants (GeMS-A1, GeMS-A10, GeMS-A1000, GeMS-B1, GeMS-B10, GeMS-B1000, GeMS-C1, GeMS-C10, GeMS-C1000) back into HDF5 files named GeMS_<variant><cap>.hdf5 (e.g. GeMS_A1.hdf5, GeMS_C1000.hdf5). The rationale is that redundant spectra add no training signal, so a small random sample per LSH cluster preserves chemical diversity while controlling dataset scale.

## Related tools

- **HDF5** (Serialization format for storing the quality-filtered input subsets and the nine capped GeMS variants in the compact HDF5-based binary format designed for deep learning)
- **Locality-sensitive hashing (LSH)** (Random-projection LSH with 64 hyperplanes approximating cosine similarity in linear time; produces the clusters whose sizes are capped)

## Evaluation signals

- Exactly nine variants are produced (GeMS-A1/A10/A1000, GeMS-B1/B10/B1000, GeMS-C1/C10/C1000), each saved as GeMS_<variant><cap>.hdf5
- After random sampling, no cluster in the GeMS-A1/A10/A1000, GeMS-B1/B10/B1000 or GeMS-C1/C10/C1000 variants exceeds its cap of 1, 10 or 1,000 spectra respectively
- GeMS-C1 contains approximately 24 million representative spectra, matching the paper-reported GeMS-C size of ~24 million spectra before deduplication
- Input and output files are in the compact HDF5-based binary format and spectra were binned on the 1.0 m/z grid with LSH run using 64 hyperplanes

## Limitations

- Random sampling within each cluster discards non-selected spectra, so each capped variant trades redundancy reduction for data loss relative to the parent GeMS subset
- LSH is an approximate method, so very similar spectra may fall into different clusters and survive capping in more than one variant cluster
- The article notes that only a subset of available mass spectra was used for training, so capped variants are not a substitute for the full mined corpus

## Evidence

- [abstract] Redundancy addressed by LSH clustering: "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [abstract] Compact HDF5 storage format: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Mining pipeline producing GeMS subsets: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
