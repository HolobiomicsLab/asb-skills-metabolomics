---
name: hdf5-spectral-dataset-serialization
description: Use when you need to store or exchange large mined MS/MS collections
  (millions of tandem mass spectra with LC–MS/MS metadata) in a compact binary format
  optimized for deep-learning training or downstream clustering/deduplication.
when_to_use_negative:
- Your data are already available in a labeled spectral library format (e.g., MoNA,
  NIST20) and do not require repository-scale mining, clustering or compact storage.
- You need to preserve the full unmodified raw spectral detail (e.g., sub-1.0 m/z
  resolution or non-binned peak lists) for downstream analysis rather than deep-learning-ready
  representations.
- Your task requires small, fully annotated benchmark sets (e.g., MassSpecGym-style
  evaluation) rather than deduplicated, capped self-supervised training corpora.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3315
tools:
- name: HDF5
  role: Compact HDF5-based binary storage format for the GeMS spectra and selected
    LC–MS/MS metadata, designed for deep learning
  repo: https://github.com/pluskal-lab/DreaMS
- name: Locality-sensitive hashing (LSH)
  role: Random-projection LSH (64 hyperplanes) used to cluster similar spectra and
    address redundancy before serializing the capped variants
  repo: https://github.com/pluskal-lab/DreaMS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:1c07409dc7e3e155ac0244d952f9bbee73558a600f2eefa7b999467e67c490e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - HDF5
  - Locality-sensitive hashing (LSH)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# HDF5 spectral dataset serialization

## Summary

Serialize repository-scale MS/MS collections (GeMS spectra mined from MassIVE GNPS, including selected LC–MS/MS metadata) into a compact HDF5-based binary format explicitly designed for deep learning, producing nine capped GeMS variants (GeMS-A1/A10/A1000, B1/B10/B1000, C1/C10/C1000).

## When to use

Use when you need to store or exchange large mined MS/MS collections (millions of tandem mass spectra with LC–MS/MS metadata) in a compact binary format optimized for deep-learning training or downstream clustering/deduplication.

## When NOT to use

- Your data are already available in a labeled spectral library format (e.g., MoNA, NIST20) and do not require repository-scale mining, clustering or compact storage.
- You need to preserve the full unmodified raw spectral detail (e.g., sub-1.0 m/z resolution or non-binned peak lists) for downstream analysis rather than deep-learning-ready representations.
- Your task requires small, fully annotated benchmark sets (e.g., MassSpecGym-style evaluation) rather than deduplicated, capped self-supervised training corpora.

## Inputs

- Quality-filtered GeMS-A, GeMS-B, GeMS-C tandem mass spectrum subsets mined from the MassIVE GNPS repository
- Selected LC–MS/MS metadata associated with each spectrum
- LSH clustering assignments (64 random-projection hyperplanes, 1.0 m/z binned spectra)

## Outputs

- Nine HDF5 files named GeMS_<variant><cap>.hdf5 (e.g., GeMS_A1.hdf5, GeMS_A10.hdf5, ..., GeMS_C1000.hdf5)
- Cluster-capped representative spectra with cluster sizes limited to 1, 10 or 1,000 per LSH cluster (GeMS-C1 yields ~24 million spectra)

## How to apply

Load the quality-filtered GeMS-A, GeMS-B and GeMS-C spectrum subsets and, if deduplicating, first bin each spectrum on a 1.0 m/z grid, then apply random-projection locality-sensitive hashing with 64 hyperplanes to approximate cosine similarity in linear time and group similar spectra into LSH clusters. For each of the three quality subsets, cap each LSH cluster by randomly sampling at most 1, 10 and 1,000 spectra per cluster, which yields nine variants (GeMS-A1, GeMS-A10, GeMS-A1000, GeMS-B1, GeMS-B10, GeMS-B1000, GeMS-C1, GeMS-C10, GeMS-C1000). Store the spectra together with selected LC–MS/MS metadata in the compact HDF5-based binary format designed for deep learning, writing one output file per variant named GeMS_<variant><cap>.hdf5 (e.g., GeMS_A1.hdf5, GeMS_A10.hdf5, ..., GeMS_C1000.hdf5). Random sampling within clusters (rather than taking the first k spectra) is the key design choice to avoid biasing the capped variant toward particular acquisition contexts.

## Related tools

- **HDF5** (Compact HDF5-based binary storage format for the GeMS spectra and selected LC–MS/MS metadata, designed for deep learning) — https://github.com/pluskal-lab/DreaMS
- **Locality-sensitive hashing (LSH)** (Random-projection LSH (64 hyperplanes) used to cluster similar spectra and address redundancy before serializing the capped variants) — https://github.com/pluskal-lab/DreaMS

## Evaluation signals

- Exactly nine output files are produced, named GeMS_<variant><cap>.hdf5 for all combinations of {A, B, C} × {1, 10, 1000}.
- Cluster caps are respected: no LSH cluster in a given variant exceeds its cap (1, 10 or 1,000 spectra), enforced by random sampling within each cluster.
- GeMS-C1 contains ~24 million representative spectra, matching the paper-reported GeMS-C size of ~24 million spectra before deduplication.
- Each HDF5 file stores both the spectra (binned on the 1.0 m/z grid) and the selected LC–MS/MS metadata fields.
- Serialization round-trips cleanly: reloading a GeMS_*.hdf5 file yields the same spectra and metadata as the pre-serialization capped clusters.

## Limitations

- Capping is by random sampling within LSH clusters, so repeated serializations with different random seeds will not produce identical variants.
- The pipeline focused on tandem mass spectra only, disregarding MS1 isotopic patterns and adduct distributions, and did not incorporate spectra beyond positive ionization mode or singly charged precursor ions.
- Even after capping, only a subset of available mass spectra was used for training in the original study, so the serialized variants are not exhaustive of the repository.

## Evidence

- [abstract] Compact HDF5 storage of GeMS spectra with metadata: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] LSH clustering to address redundancy: "we addressed redundancy in GeMS by clustering similar spectra using locality-sensitive hashing (LSH)"
- [abstract] Cluster size caps via random sampling: [evidence span withheld — anchor status: not_found]
- [abstract] GeMS mined from MassIVE GNPS: "our GNPS Experimental Mass Spectra (GeMS) dataset mined from the MassIVE GNPS repository"
- [abstract] Quality control subsets GeMS-A/B/C: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [discussion] Scope: only tandem mass spectra considered: "our method is focused solely on tandem mass spectra, disregarding other important features such as MS1 isotopic patterns or adduct distributions"
