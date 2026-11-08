---
name: precursor-mass-difference-analysis
description: Use when a large corpus of unidentified spectra has been clustered (e.g.,
  GLEAMS clusters with medoid indexes) and you need to annotate representative spectra
  via open modification search and derive the mass-difference profile.
when_to_use_negative:
- The spectra already have confident database-search identifications (no unidentified-cluster
  annotation is needed)
- The dataset is small and not clustered — direct closed or open searching of individual
  spectra is more appropriate than medoid-search-plus-propagation
- The target library lacks the organism/instrument type of the query spectra (e.g.,
  non-Human HCD data against the Human HCD library)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GLEAMS
  role: Embeds spectra into a 32-dimensional space and clusters them; provides cluster
    labels (GLEAMS_cluster.npy) and medoid indexes (GLEAMS_cluster_medoids.npy) used
    to select medoid spectra and propagate PSMs
  repo: https://github.com/bittremieux/GLEAMS
- name: ANN-SoLo
  role: Performs open modification searching of the selected medoid spectra against
    the MassIVE-KB (v1) Human HCD spectral library at 1% FDR
- name: conda
  role: Creates the Python 3.8 environment with compiler tools and GPU runtime required
    to run GLEAMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:6d00952b3b54e4729adf3e12d9af3c134cf156901d720b3d10732f99a3f73e49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - Python
  - conda
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# precursor-mass-difference-analysis

## Summary

Annotate previously unidentified MS/MS spectra via open modification searching of spectrum-cluster medoids, then catalog the precursor mass differences of the resulting PSMs to characterize post-translational modifications and other mass shifts.

## When to use

Use when a large corpus of unidentified spectra has been clustered (e.g., GLEAMS clusters with medoid indexes) and you need to annotate representative spectra via open modification search and derive the mass-difference profile.

## When NOT to use

- The spectra already have confident database-search identifications (no unidentified-cluster annotation is needed)
- The dataset is small and not clustered — direct closed or open searching of individual spectra is more appropriate than medoid-search-plus-propagation
- The target library lacks the organism/instrument type of the query spectra (e.g., non-Human HCD data against the Human HCD library)

## Inputs

- GLEAMS_cluster.npy (NumPy array of cluster labels per spectrum; -1 = noise)
- GLEAMS_cluster_medoids.npy (NumPy array of indexes of cluster representative spectra)
- MS/MS spectra in peak files (e.g., mzML)
- MassIVE-KB (v1) Human HCD spectral library

## Outputs

- 1% FDR-filtered PSMs from ANN-SoLo open modification search (~5.3 million medoid identifications)
- Propagated PSMs assigned to cluster members (~44 million)
- Top 500 precursor mass differences from the open modification search of GLEAMS cluster centroids

## How to apply

First select medoid spectra from the clusters of previously unidentified spectra using GLEAMS_cluster_medoids.npy, excluding noise points labeled -1 in GLEAMS_cluster.npy. Run ANN-SoLo (v0.3.3) open modification search on the selected medoid spectra against the MassIVE-KB (v1) Human HCD spectral library, and filter the results at 1% FDR to retain confident peptide-spectrum matches. Propagate each medoid's peptide identification to all member spectra of its cluster via the GLEAMS cluster labels. Finally, compile the precursor mass differences of the open search hits (e.g., the top 500 precursor mass differences from the GLEAMS cluster centroids) to survey modification landscape.

## Related tools

- **GLEAMS** (Embeds spectra into a 32-dimensional space and clusters them; provides cluster labels (GLEAMS_cluster.npy) and medoid indexes (GLEAMS_cluster_medoids.npy) used to select medoid spectra and propagate PSMs) — https://github.com/bittremieux/GLEAMS
- **ANN-SoLo** (Performs open modification searching of the selected medoid spectra against the MassIVE-KB (v1) Human HCD spectral library at 1% FDR)
- **conda** (Creates the Python 3.8 environment with compiler tools and GPU runtime required to run GLEAMS)

## Examples

```
gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.3
```

## Evaluation signals

- The direct open modification search of the 45 million cluster medoid spectra yields ~5.3 million PSMs at 1% FDR
- Propagating medoid peptide labels via cluster membership produces ~44 million additional PSMs, consistent with reported counts
- Noise points (cluster label -1) are excluded from medoid selection and propagation
- The final mass-difference table contains the top 500 precursor mass differences from the GLEAMS cluster centroids
- ANN-SoLo version and FDR threshold (1%) are recorded and applied consistently across the search results

## Limitations

- Identification is limited to medoid spectra, so annotations depend on cluster quality; spectra assigned to noise (-1) receive no propagated PSMs
- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU
- The ANN-SoLo search is restricted to the MassIVE-KB (v1) Human HCD library, so modifications outside that library's precursor/spectrum space may be missed
- Model weights download can fail with a Git LFS data-quota error, requiring manual download from the release page

## Evidence

- [supplementary] Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids.: "Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids."
- [readme] The output will be written to the `GLEAMS_cluster.npy` NumPy file with cluster labels per embedding (`-1` indicates noise, minimum cluster size 2).: "The output will be written to the `GLEAMS_cluster.npy` NumPy file with cluster labels per embedding (`-1` indicates noise, minimum cluster size 2)."
- [readme] This will perform hierarchical clustering on the embeddings with the given distance threshold.: "This will perform hierarchical clustering on the embeddings with the given distance threshold."
- [methods] GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"
