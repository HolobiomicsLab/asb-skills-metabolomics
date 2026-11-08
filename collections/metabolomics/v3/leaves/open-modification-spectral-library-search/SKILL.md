---
name: open-modification-spectral-library-search
description: Use when large sets of unidentified spectra have been grouped into clusters
  (e.g. GLEAMS clusters) and you want to annotate them by open modification searching
  of cluster medoids against a spectral library such as MassIVE-KB, accepting unknown
  precursor mass shifts.
when_to_use_negative:
- When spectra are already identified by a standard (closed-mass) database or spectral
  library search — open modification searching is intended for previously unidentified
  spectra with unanticipated mass shifts.
- When individual unidentified spectra lack cluster context — the propagation step
  depends on GLEAMS cluster labels and medoids from GLEAMS_cluster.npy / GLEAMS_cluster_medoids.npy.
- When strict identification of a known, narrow set of modifications is required and
  unanticipated mass shifts are unacceptable as evidence.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: ANN-SoLo
  role: Performs the open modification spectral library search of GLEAMS cluster medoid
    spectra (v0.3.3), with FDR filtering and precursor mass difference extraction.
- name: GLEAMS
  role: Produces the spectrum embeddings, cluster labels, and medoid indexes (GLEAMS_cluster.npy,
    GLEAMS_cluster_medoids.npy) that define which spectra are searched and how identifications
    are propagated.
  repo: https://github.com/bittremieux/GLEAMS
- name: Python
  role: Runtime environment (Python 3.8 required) for installing GLEAMS, loading NumPy
    cluster files, and post-processing search results.
- name: conda
  role: Creates the environment with compiler tools and GPU runtime needed to run
    GLEAMS.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:d490c333dbfc0d932d18d0d2612ff6f87d4527d505610b8aa91be84270ef0f39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - Python
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# open-modification-spectral-library-search

## Summary

Annotates previously unidentified MS/MS spectra by searching them against a spectral library with tolerance for unanticipated precursor mass shifts (open modification search), then propagates hits to related spectra. Applied here to GLEAMS cluster medoids with ANN-SoLo to lift 45 million medoid identifications to ~49 million total PSMs.

## When to use

Use when large sets of unidentified spectra have been grouped into clusters (e.g. GLEAMS clusters) and you want to annotate them by open modification searching of cluster medoids against a spectral library such as MassIVE-KB, accepting unknown precursor mass shifts.

## When NOT to use

- When spectra are already identified by a standard (closed-mass) database or spectral library search — open modification searching is intended for previously unidentified spectra with unanticipated mass shifts.
- When individual unidentified spectra lack cluster context — the propagation step depends on GLEAMS cluster labels and medoids from GLEAMS_cluster.npy / GLEAMS_cluster_medoids.npy.
- When strict identification of a known, narrow set of modifications is required and unanticipated mass shifts are unacceptable as evidence.

## Inputs

- GLEAMS_cluster.npy — NumPy array of cluster labels per spectrum (-1 indicates noise, minimum cluster size 2)
- GLEAMS_cluster_medoids.npy — NumPy array of indexes of the cluster representative spectra (medoids)
- MS/MS spectra in peak files (e.g. mzML) for the cluster medoids
- MassIVE-KB (v1) Human HCD spectral library

## Outputs

- Peptide-spectrum matches (PSMs) for cluster medoids at 1% FDR (~5.3 million PSMs)
- Propagated PSMs assigned to cluster members (~44 million PSMs)
- Top 500 precursor mass differences from the open modification search of GLEAMS cluster centroids

## How to apply

Load the cluster labels from GLEAMS_cluster.npy and the medoid indexes from GLEAMS_cluster_medoids.npy, then select the representative (medoid) spectra for clusters of previously unidentified spectra, excluding noise points labeled -1. Run ANN-SoLo (v0.3.3) open modification search on the selected medoid spectra against the MassIVE-KB (v1) Human HCD spectral library, which tolerates unanticipated precursor mass differences rather than restricting to the library precursor mass. Filter the ANN-SoLo search results at 1% FDR to retain confident peptide-spectrum matches, and compile the top 500 precursor mass differences from the search of GLEAMS cluster centroids as a summary of modification signatures. Propagate each medoid's PSM identification to all member spectra of its cluster using the GLEAMS cluster labels, yielding roughly 5.3 million direct and 44 million propagated identifications from 45 million medoid spectra.

## Related tools

- **ANN-SoLo** (Performs the open modification spectral library search of GLEAMS cluster medoid spectra (v0.3.3), with FDR filtering and precursor mass difference extraction.)
- **GLEAMS** (Produces the spectrum embeddings, cluster labels, and medoid indexes (GLEAMS_cluster.npy, GLEAMS_cluster_medoids.npy) that define which spectra are searched and how identifications are propagated.) — https://github.com/bittremieux/GLEAMS
- **Python** (Runtime environment (Python 3.8 required) for installing GLEAMS, loading NumPy cluster files, and post-processing search results.)
- **conda** (Creates the environment with compiler tools and GPU runtime needed to run GLEAMS.)

## Examples

```
gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.3
```

## Evaluation signals

- Direct PSM count is approximately 5.3 million and propagated PSM count is approximately 44 million, consistent with the reported figures.
- All reported PSMs pass 1% FDR filtering of the ANN-SoLo search results.
- Noise spectra (cluster label -1) are excluded from medoid selection; every medoid searched corresponds to a cluster of size >= 2.
- The top 500 precursor mass differences from the open search of GLEAMS cluster centroids are compiled and can be reviewed for known modification signatures.
- Each propagated PSM traces back to a medoid identification whose cluster label matches the member spectrum's label in GLEAMS_cluster.npy.

## Limitations

- Identifications are inherited from a single medoid per cluster, so heterogeneity within a cluster maps all members to the medoid's peptide assignment.
- Noise points labeled -1 by GLEAMS clustering are excluded and therefore receive no propagated identifications.
- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU, and the Git LFS model weights download can exceed data quota limits, requiring manual weight download.

## Evidence

- [supplementary] Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids.: "Top 500 precursor mass differences from the ANN-SoLo open modification search of GLEAMS cluster centroids."
- [readme] The output will be written to the GLEAMS_cluster.npy NumPy file with cluster labels per embedding (-1 indicates noise, minimum cluster size 2). Additionally, a file GLEAMS_cluster_medoids.npy will be created containing indexes of the cluster representative spectra (medoids).: "The output will be written to the `GLEAMS_cluster.npy` NumPy file with cluster labels per embedding (`-1` indicates noise, minimum cluster size 2).
Additionally, a file `GLEAMS_cluster_medoids.npy`"
- [readme] GLEAMS was trained on 30 million PSMs from the MassIVE-KB (v1) dataset.: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset](https://massive.ucsd.edu/ProteoSAFe/static/massive-kb-libraries.jsp)."
- [readme] GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU.: "GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU."
- [methods] All 185 million PSMs at 1% FDR obtained using MSGF+: "All 185 million PSMs at 1% FDR obtained using MSGF+"
