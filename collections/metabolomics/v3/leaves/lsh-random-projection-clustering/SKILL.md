---
name: lsh-random-projection-clustering
description: Use when a large mass spectra collection (e.g., millions of LC-MS/MS
  spectra from GeMS/MassIVE GNPS) contains redundant similar spectra and you need
  near-duplicate clusters in linear time before capping or subsampling.
when_to_use_negative:
- The dataset is small enough that exact pairwise cosine similarity (e.g., spectral
  entropy or dot-product similarity from matchms tooling) is tractable; LSH approximation
  is only needed at repository scale.
- The goal is a k-nearest-neighbor similarity graph rather than redundancy-clustering
  — the article used approximate 3-NN graph construction (PyNNDescent) for the DreaMS
  Atlas, not LSH.
- Spectra are already deduplicated or are library spectra where retaining all variants
  of similar spectra is desired for supervised training.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HDF5
  role: Compact binary format used to store the GeMS spectra (with selected LC–MS/MS
    metadata) as both input to and output of the LSH clustering pipeline
- name: Locality-sensitive hashing (LSH)
  role: Random-projection LSH with 64 hyperplanes used to cluster similar spectra
    by approximating cosine similarity in linear time
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:53a715111b153e8715b945011c05e2bf7888efae6fd461eff0c0fcf1e4d8d808
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  - Locality-sensitive hashing (LSH)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lsh-random-projection-clustering

## Summary

Redundancy reduction for repository-scale tandem MS/MS collections by clustering spectra with random-projection locality-sensitive hashing (LSH), which approximates cosine similarity in linear time, then capping cluster sizes by random sampling to produce deduplicated dataset variants.

## When to use

Use when a large mass spectra collection (e.g., millions of LC-MS/MS spectra from GeMS/MassIVE GNPS) contains redundant similar spectra and you need near-duplicate clusters in linear time before capping or subsampling.

## When NOT to use

- The dataset is small enough that exact pairwise cosine similarity (e.g., spectral entropy or dot-product similarity from matchms tooling) is tractable; LSH approximation is only needed at repository scale.
- The goal is a k-nearest-neighbor similarity graph rather than redundancy-clustering — the article used approximate 3-NN graph construction (PyNNDescent) for the DreaMS Atlas, not LSH.
- Spectra are already deduplicated or are library spectra where retaining all variants of similar spectra is desired for supervised training.

## Inputs

- Quality-filtered GeMS-A, GeMS-B, and GeMS-C spectrum subsets stored in the compact HDF5-based binary format
- LSH configuration: 1.0 m/z binning grid and 64 random-projection hyperplanes

## Outputs

- Nine HDF5 files of capped LSH clusters: GeMS_A1.hdf5, GeMS_A10.hdf5, GeMS_A1000.hdf5, GeMS_B1.hdf5, GeMS_B10.hdf5, GeMS_B1000.hdf5, GeMS_C1.hdf5, GeMS_C10.hdf5, GeMS_C1000.hdf5

## How to apply

Load the quality-filtered spectra subsets (GeMS-A, GeMS-B, GeMS-C) from the compact HDF5-based binary format. Bin each spectrum on a 1.0 m/z grid as specified in the LSH clustering setup, converting variable-length peak lists to fixed-length vectors. Apply random-projection LSH with 64 hyperplanes to group spectra whose binned vectors have high cosine similarity into LSH clusters; the projection trick makes this linear-time rather than quadratic in the number of spectra. For each GeMS variant, cap each cluster to a maximum of 1, 10, or 1,000 spectra by random sampling within the cluster, yielding nine variants (GeMS-A1/A10/A1000, GeMS-B1/B10/B1000, GeMS-C1/C10/C1000). Serialize the capped clusters back to the HDF5 binary format, one file per variant named GeMS_<variant><cap>.hdf5.

## Related tools

- **HDF5** (Compact binary format used to store the GeMS spectra (with selected LC–MS/MS metadata) as both input to and output of the LSH clustering pipeline)
- **Locality-sensitive hashing (LSH)** (Random-projection LSH with 64 hyperplanes used to cluster similar spectra by approximating cosine similarity in linear time)

## Examples

```
python cluster_lsh.py --input GeMS_C.hdf5 --bins 1.0 --hyperplanes 64 --cap 1000 --output GeMS_C1000.hdf5
```

## Evaluation signals

- Nine clustered variants are produced with correct cluster caps: max cluster sizes of 1, 10, and 1,000 spectra per cluster for each of GeMS-A, GeMS-B, GeMS-C.
- GeMS-C1 yields approximately 24 million representative spectra, matching the paper-reported GeMS-C size of ~24 million spectra before deduplication.
- All nine output files serialize back into the compact HDF5-based binary format with selected LC–MS/MS metadata preserved.
- Spectra were binned on the 1.0 m/z grid prior to projection, so the binned vectors match the dimensionality implied by the instrument m/z range.

## Limitations

- LSH only approximates cosine similarity, so cluster membership is not an exact nearest-neighbor partition; similar spectra can fall into different buckets.
- Random sampling within clusters discards spectra, so capped variants are representative subsets rather than exhaustive de-duplications.
- The reported paper-trained models used only a subset of available mass spectra; capped variants constrain training diversity further.

## Evidence

- [abstract] LSH clustering to address redundancy, limiting cluster sizes by random sampling (e.g. 10 or 1,000 spectra per cluster): "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [abstract] HDF5 storage of GeMS spectra: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] Mining pipeline including LSH clustering step: "Our mining pipeline consists of five main steps (Fig. 2a). First, we collected 250,000 LC–MS/MS experiments"
- [discussion] Trained models used only a subset of available spectra: "we trained our model using only a subset of available mass spectra"
