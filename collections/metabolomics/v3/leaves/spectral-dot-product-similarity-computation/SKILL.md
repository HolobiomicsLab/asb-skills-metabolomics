---
name: spectral-dot-product-similarity-computation
description: Use when a large MS/MS training corpus (e.g., ~200,000 downsampled MassIVE-KB-derived
  PSM spectra) must be reduced to a small, diversity-covering reference subset, and
  spectra-to-spectra similarity must be quantified classically (normalized dot product)
  rather than via a learned embedding.
when_to_use_negative:
- When spectra have already been converted to 32-dimensional GLEAMS embeddings — downstream
  grouping should instead use hierarchical clustering on embeddings (gleams cluster
  with --distance_threshold 0.3), not raw dot-product similarity
- When the goal is a full spectral library search against identified peptides (use
  ANN-SoLo-style open modification search) rather than unsupervised reference-set
  selection
- When the input spectra lack fragment peak lists (e.g., only metadata/PSM tables
  from the MassIVE-KB TSV are available) — spectra must be retrieved from the MassIVE
  FTP server by filename and scan number first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GLEAMS
  role: 'Downstream consumer: the 500 selected reference spectra provide the similarity-based
    features that the GLEAMS encoder uses when embedding spectra into its 32-dimensional
    space'
  repo: https://github.com/bittremieux/GLEAMS
- name: ANN-SoLo
  role: Open spectral library search tool whose fragment peak-matching style the normalized
    dot product similarity computation parallels; used on GLEAMS cluster centroids
    in the same study
- name: Python
  role: Implementation environment for computing the pairwise similarity matrix and
    running the apricot submodular selection (GLEAMS requires Python 3.8)
- name: apricot
  role: Performs the greedy submodular facility-location selection of 500 representative
    reference spectra from the precomputed similarity matrix
- name: conda
  role: Environment setup for GLEAMS/Python dependencies
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:1c02ef4ab8b6491cdcf1cb006fc76d044d9090fac2c3b6f28017a5b40c17e6c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - Python
  - apricot
  - conda
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-dot-product-similarity-computation

## Summary

Compute a pairwise spectral similarity matrix over MS/MS spectra using the normalized dot product with a 0.05 Da fragment m/z peak-matching tolerance, so that a small representative reference set (e.g., GLEAMS' 500 reference spectra) can be selected from a large training corpus via submodular facility-location selection (apricot).

## When to use

Use when a large MS/MS training corpus (e.g., ~200,000 downsampled MassIVE-KB-derived PSM spectra) must be reduced to a small, diversity-covering reference subset, and spectra-to-spectra similarity must be quantified classically (normalized dot product) rather than via a learned embedding.

## When NOT to use

- When spectra have already been converted to 32-dimensional GLEAMS embeddings — downstream grouping should instead use hierarchical clustering on embeddings (gleams cluster with --distance_threshold 0.3), not raw dot-product similarity
- When the goal is a full spectral library search against identified peptides (use ANN-SoLo-style open modification search) rather than unsupervised reference-set selection
- When the input spectra lack fragment peak lists (e.g., only metadata/PSM tables from the MassIVE-KB TSV are available) — spectra must be retrieved from the MassIVE FTP server by filename and scan number first

## Inputs

- ~200,000 downsampled training MS/MS spectra from the MassIVE-KB (v1) Human HCD Spectral Library-derived training set (30 million PSMs metadata TSV plus peak files from the MassIVE FTP server)
- Peak lists for each MS/MS spectrum (fragment m/z values) extracted from mzML/peak files
- Pairwise spectral similarity matrix (n x n, normalized dot product, 0.05 Da fragment m/z tolerance)

## Outputs

- 500 selected reference spectra exported to a serialized file for use by the GLEAMS encoder
- Pairwise similarity matrix over the training spectra (intermediate output)

## How to apply

Load the downsampled training MS/MS spectra from the MassIVE-KB-derived training set (metadata and scan numbers retrieved from MassIVE; spectra from the MassIVE FTP server, as described in the GLEAMS README). Compute the pairwise similarity matrix between all spectra using the normalized dot product, matching fragment peaks within a 0.05 m/z tolerance. Feed the precomputed similarity matrix to apricot's facility-location function, which greedily maximizes coverage of the corpus, to select 500 representative reference spectra. Export the selected candidates to a serialized file so the GLEAMS encoder can use them to derive similarity-based features. The 0.05 Da tolerance mirrors the peak-matching granularity of spectral library search tools like ANN-SoLo that operate on the same kind of peak data.

## Related tools

- **GLEAMS** (Downstream consumer: the 500 selected reference spectra provide the similarity-based features that the GLEAMS encoder uses when embedding spectra into its 32-dimensional space) — https://github.com/bittremieux/GLEAMS
- **ANN-SoLo** (Open spectral library search tool whose fragment peak-matching style the normalized dot product similarity computation parallels; used on GLEAMS cluster centroids in the same study)
- **Python** (Implementation environment for computing the pairwise similarity matrix and running the apricot submodular selection (GLEAMS requires Python 3.8))
- **apricot** (Performs the greedy submodular facility-location selection of 500 representative reference spectra from the precomputed similarity matrix)
- **conda** (Environment setup for GLEAMS/Python dependencies)

## Examples

```
from apricot import FacilityLocationSelection; refs = FacilityLocationSelection(500, metric='precomputed').fit_transform(sim)
```

## Evaluation signals

- Exactly 500 reference spectra are exported to the serialized file for the GLEAMS encoder
- The similarity matrix is symmetric with normalized dot-product values in [0, 1], computed with a 0.05 m/z fragment peak-matching tolerance
- The apricot facility-location greedy objective (marginal gain) increases monotonically over selection rounds and plateaus as coverage of the ~200,000-spectrum corpus saturates
- The selected reference set spans diverse peptide/precursor space of the MassIVE-KB training PSMs rather than clustering on a few abundant precursors
- GLEAMS training with the exported reference spectra reproduces 32-dimensional embeddings in which spectra of the same peptide lie close together

## Limitations

- The 0.05 Da fragment m/z tolerance is a hard matching threshold; spectra with systematic fragment shifts or chimeric peaks may be scored poorly by the normalized dot product
- The GLEAMS training data cannot be downloaded as a single file: the 30 million PSMs from MassIVE-KB (v1) must be assembled from a metadata TSV plus per-task mzTab files and spectra from the MassIVE FTP server
- The full GLEAMS pipeline (of which this similarity computation is part) requires Python 3.8, Linux, and a CUDA-enabled GPU; Git LFS bandwidth limits may force manual download of model weights (gleams_82c0124b.hdf5)

## Evidence

- [readme] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together.: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together."
- [methods] command to convert MS/MS spectra in peak files to 32-dimensional embeddings: "command to convert MS/MS spectra in peak files to 32-dimensional embeddings"
- [readme] This will perform hierarchical clustering on the embeddings with the given distance threshold.: "This will perform hierarchical clustering on the embeddings with the given distance threshold."
- [full_text] The 500 reference spectra were selected by computing a pairwise similarity matrix (normalized dot product with fragment m/z tolerance 0.05 m/z) and using it to perform submodular selection with the facility location function.: [evidence span withheld — anchor status: not_found]
