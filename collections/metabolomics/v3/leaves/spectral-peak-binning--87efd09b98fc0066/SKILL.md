---
name: spectral-peak-binning
description: Use when variable-length MS/MS peak lists (e.g. from GeMS-A/B/C quality-filtered
  subsets) must be converted into fixed-dimension vectors before approximate similarity
  computations such as LSH random projections.
when_to_use_negative:
- Input spectra are already stored as fixed-dimension vectors in the compact HDF5
  GeMS format and only need LSH clustering or cluster capping.
- The downstream task requires high mass accuracy (e.g. 10-ppm precursor m/z filtering
  for spectral library retrieval), where collapsing peaks into 1.0 m/z bins would
  destroy the needed precision.
- Constructing the DreaMS Atlas approximate 3-NN graph, which operates on DreaMS embeddings
  via PyNNDescent rather than on binned peak vectors.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DreaMS
  role: Repository implementing the GeMS mining pipeline in which peak binning on
    a 1.0 m/z grid precedes LSH clustering and cluster capping.
  repo: https://github.com/pluskal-lab/DreaMS
- name: HDF5
  role: Storage format for the GeMS spectra (including binned/clustered variants)
    in the compact binary format designed for deep learning.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:dcb7c1b4065142da321db2e13a1107686ac829aa8e4623aa11f64bf6154b05fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - HDF5
  - Locality-sensitive hashing (LSH)
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-peak-binning

## Summary

Bins tandem MS/MS peaks onto a 1.0 m/z grid to convert variable-length peak lists into fixed-dimension sparse intensity vectors, the prerequisite step for random-projection LSH clustering that removes redundancy in the GeMS datasets.

## When to use

Use when variable-length MS/MS peak lists (e.g. from GeMS-A/B/C quality-filtered subsets) must be converted into fixed-dimension vectors before approximate similarity computations such as LSH random projections.

## When NOT to use

- Input spectra are already stored as fixed-dimension vectors in the compact HDF5 GeMS format and only need LSH clustering or cluster capping.
- The downstream task requires high mass accuracy (e.g. 10-ppm precursor m/z filtering for spectral library retrieval), where collapsing peaks into 1.0 m/z bins would destroy the needed precision.
- Constructing the DreaMS Atlas approximate 3-NN graph, which operates on DreaMS embeddings via PyNNDescent rather than on binned peak vectors.

## Inputs

- Quality-filtered MS/MS spectra from GeMS-A, GeMS-B or GeMS-C subsets in the compact HDF5-based binary format (peak m/z and intensity lists plus selected LC–MS/MS metadata)
- Binning specification: 1.0 m/z grid as defined in the LSH clustering setup

## Outputs

- One fixed-length sparse intensity vector per spectrum, binned on the 1.0 m/z m/z grid
- Binned spectra suitable as input to 64-hyperplane random-projection LSH clustering

## How to apply

Load the quality-filtered GeMS-A, GeMS-B and GeMS-C spectra from the compact HDF5-based binary format and bin each spectrum on a 1.0 m/z grid as specified in the LSH clustering setup: assign every peak's intensity to the bin covering its m/z, aggregating intensities that fall in the same bin. The result is a fixed-length sparse vector per spectrum whose dimensionality is set by the precursor/fragment m/z range divided by the 1.0 m/z bin width. These binned vectors are the input representation over which random-projection locality-sensitive hashing with 64 hyperplanes approximates cosine similarity in linear time to group similar spectra into LSH clusters. Downstream, each cluster is capped by random sampling (1, 10 or 1,000 spectra per cluster for GeMS-A/B/C) to produce the nine GeMS variants serialized back into HDF5. Binning is deliberately coarse: it trades fine mass resolution for a cheap, uniform vector space that makes linear-time LSH feasible at repository scale (hundreds of millions of MassIVE GNPS spectra).

## Related tools

- **DreaMS** (Repository implementing the GeMS mining pipeline in which peak binning on a 1.0 m/z grid precedes LSH clustering and cluster capping.) — https://github.com/pluskal-lab/DreaMS
- **HDF5** (Storage format for the GeMS spectra (including binned/clustered variants) in the compact binary format designed for deep learning.)

## Evaluation signals

- Every output spectrum vector is aligned to the 1.0 m/z grid: bin boundaries fall at multiples of 1.0 and no peak contributes to more than one bin.
- Vector dimensionality equals the covered m/z span divided by the 1.0 bin width, and is identical across all binned spectra so LSH projections are well-defined.
- Binned vectors feed LSH with 64 hyperplanes and yield clusters that, after capping at 1, 10 and 1,000 spectra per cluster, produce exactly nine GeMS variants (A1/A10/A1000, B1/B10/B1000, C1/C10/C1000).
- GeMS-C1 contains approximately 24 million representative spectra, matching the paper-reported GeMS-C size.
- All nine variants serialize back into valid HDF5 files (e.g. GeMS_A1.hdf5 ... GeMS_C1000.hdf5) that load without loss of LC–MS/MS metadata.

## Limitations

- The 1.0 m/z grid is a coarse representation chosen for linear-time LSH; it blurs the instrument m/z accuracy distinctions that the GeMS quality-control pipeline (GeMS-A/B/C) is based on.
- Binning is specified for the LSH clustering setup specifically; tasks such as fine-tuned spectral similarity prediction or library retrieval operate on full-resolution spectra, not binned vectors.
- Even after clustering-based redundancy reduction, only a subset of available mass spectra was used for training, so binning+LSH does not exhaust the repository-scale data.

## Evidence

- [methods] Binning precedes LSH clustering: [evidence span withheld — anchor status: not_found]
- [methods] LSH parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Redundancy rationale: "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [abstract] Storage format: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
