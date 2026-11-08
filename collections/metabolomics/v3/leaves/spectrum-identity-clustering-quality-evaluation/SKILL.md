---
name: spectrum-identity-clustering-quality-evaluation
description: Use when GLEAMS embeddings of a large MS/MS dataset (e.g., MassIVE-KB
  peak files) have been hierarchically clustered and you must quantify clustering
  quality and label-propagation yield.
when_to_use_negative:
- You have not yet clustered the embeddings — this skill evaluates an existing clustering,
  it does not produce one (use gleams cluster first).
- Your spectra lack ground-truth peptide identifications (e.g., MSGF+ PSMs at 1% FDR),
  so incorrectly clustered fraction and completeness cannot be estimated.
- You are working with a small dataset where the repository-scale metrics (76% clustered,
  1.16% incorrect, 0.837 completeness) are not a meaningful reference.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GLEAMS
  role: Produces the 32-dimensional embeddings (gleams embed) and hierarchical clustering
    (gleams cluster) whose quality this skill evaluates
  repo: https://github.com/bittremieux/GLEAMS
- name: MSGF+
  role: Source of the 185 million PSMs at 1% FDR used as reference identifications
    for computing cluster purity and completeness
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:8faca2ebf63e3eab55cfbd5265327a3e7a6bd24d3cb0c27ae64b2f02e7e491f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - GLEAMS
  - MSGF+
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spectrum-identity-clustering-quality-evaluation

## Summary

Evaluate GLEAMS spectrum clustering quality on repository-scale MassIVE-KB datasets by measuring clustered fraction, incorrectly clustered fraction (cluster purity error), and completeness, then propagate peptide labels within pure clusters to quantify newly annotated spectra.

## When to use

Use when GLEAMS embeddings of a large MS/MS dataset (e.g., MassIVE-KB peak files) have been hierarchically clustered and you must quantify clustering quality and label-propagation yield.

## When NOT to use

- You have not yet clustered the embeddings — this skill evaluates an existing clustering, it does not produce one (use gleams cluster first).
- Your spectra lack ground-truth peptide identifications (e.g., MSGF+ PSMs at 1% FDR), so incorrectly clustered fraction and completeness cannot be estimated.
- You are working with a small dataset where the repository-scale metrics (76% clustered, 1.16% incorrect, 0.837 completeness) are not a meaningful reference.

## Inputs

- GLEAMS_cluster.npy NumPy file of cluster labels per embedding (-1 indicates noise)
- GLEAMS_embed.parquet metadata file with precursor mass, filename, scan number, and peptide identifications
- 32-dimensional GLEAMS embeddings (GLEAMS_embed.npy) for 669 million MassIVE-KB spectra
- MassIVE-KB metadata TSV with MSGF+ PSMs at 1% FDR used as ground-truth identifications for purity/completeness estimation

## Outputs

- Clustered fraction (fraction of spectra in clusters, ~76% / 511M of 669M)
- Incorrectly clustered fraction (fraction of spectra clustered with a different peptide, ~1.16%)
- Completeness (fraction of same-peptide spectra clustered together, ~0.837)
- Count of newly propagated PSMs from label propagation in pure clusters (~82 million)

## How to apply

After running 'gleams embed' to produce 32-dimensional embeddings and 'gleams cluster' with hierarchical average linkage, apply a Euclidean distance threshold (0.35 for the full-scale 669-million-spectrum MassIVE-KB run) and postprocess clusters by splitting any cluster containing inconsistent precursor masses. Compute clustered fraction (spectra assigned to clusters of size >= 2; expected ~76%: 511 million of 669 million), incorrectly clustered fraction (spectra clustered with spectra of a different identified peptide; expected ~1.16%), and completeness (fraction of identically-identified spectra grouped together; expected ~0.837). Then propagate peptide identifications within pure clusters—clusters where all identified members share the same peptide—to unassigned member spectra and count newly propagated PSMs (~82 million at full scale). Judge success by comparing all three metrics against the reported values and confirming the propagated-PSM count is of the same order.

## Related tools

- **GLEAMS** (Produces the 32-dimensional embeddings (gleams embed) and hierarchical clustering (gleams cluster) whose quality this skill evaluates) — https://github.com/bittremieux/GLEAMS
- **MSGF+** (Source of the 185 million PSMs at 1% FDR used as reference identifications for computing cluster purity and completeness)

## Examples

```
gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.35
```

## Evaluation signals

- Clustered fraction falls near the reported 76% (511 million of 669 million MassIVE-KB spectra clustered).
- Incorrectly clustered fraction is approximately 1.16%, i.e., very few spectra share a cluster with a different identified peptide.
- Completeness is approximately 0.837, i.e., most spectra of the same peptide end up in the same cluster.
- Label propagation over pure clusters yields roughly 82 million newly assigned PSMs at repository scale.
- Postprocessing splits clusters with inconsistent precursor masses, so every final pure cluster has a consistent precursor mass.

## Limitations

- Requires a CUDA-enabled GPU and Linux (Python 3.8) to run GLEAMS embedding before evaluation is possible.
- The 0.35 distance threshold was tuned for the full MassIVE-KB corpus; other datasets may need a different --distance_threshold (the README example uses 0.3).
- Quality metrics depend on reference identifications; spectra lacking MSGF+ PSMs at 1% FDR cannot contribute to purity/completeness estimates.
- Git LFS bandwidth limits may block model-weight download; weights must be fetched manually from the v0.3 release (gleams_82c0124b.hdf5).

## Evidence

- [readme] Clustering all 669 million MassIVE-KB spectra with GLEAMS embeddings using hierarchical average linkage at Euclidean distance threshold 0.35 clustered 511 million spectra (76%) with 1.16% incorrectly clustered spectra and 0.837 completeness.: [evidence span withheld — anchor status: not_found]
- [readme] README clustering command with distance threshold.: "gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.3"
- [readme] Cluster labels output description.: "The output will be written to the `GLEAMS_cluster.npy` NumPy file with cluster labels per embedding (`-1` indicates noise, minimum cluster size 2)."
- [intro] Embedding description from the article index.: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space"
- [methods] Postprocessing by precursor mass and propagation of identifications.: [evidence span withheld — anchor status: not_found]
- [methods] Reference identifications used for validation.: "All 185 million PSMs at 1% FDR obtained using MSGF+"
