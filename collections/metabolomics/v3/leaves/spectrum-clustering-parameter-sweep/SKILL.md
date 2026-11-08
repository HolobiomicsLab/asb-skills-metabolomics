---
name: spectrum-clustering-parameter-sweep
description: Use when comparing spectrum-clustering tools on large-scale MS/MS peak-list
  data and you need per-tool cluster-quality metrics (clustered fraction, incorrect
  clustering, completeness) reproduced against a paper's reported comparison.
when_to_use_negative:
- The goal is to run GLEAMS itself for novel clustering rather than to benchmark/compare
  clustering tools against reported cluster-quality metrics.
- Input spectra lack peptide ground-truth identifications (e.g. MassIVE-KB labels),
  so incorrectly-clustered fraction and completeness cannot be computed.
- The data is small enough that clustering-tool performance at millions-of-spectra
  scale is not the question.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GLEAMS
  role: Clustering tool under evaluation; embeds MS/MS spectra into a 32-dimensional
    space and clusters them with a distance threshold (e.g. 0.3)
  repo: https://github.com/bittremieux/GLEAMS
- name: MSGF+
  role: Produced the 185 million PSMs at 1% FDR underlying the MassIVE-KB identifications
    used as ground truth
- name: ANN-SoLo
  role: Open modification search of GLEAMS cluster centroids (supplementary analysis
    of cluster quality)
- name: Python
  role: Runtime for GLEAMS (Python 3.8, Linux, CUDA-enabled GPU) used to run embedding
    and clustering commands
  repo: https://github.com/bittremieux/GLEAMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:f88074447401f4f75f1807524f0e9b8abd30c517c583f3e328918d09ea797f7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - MSGF+
  - Python
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# spectrum-clustering-parameter-sweep

## Summary

Benchmark cluster quality of MS/MS spectrum clustering tools (GLEAMS, falcon, MaRaCluster, MS-Cluster, spectra-cluster) by sweeping each tool's parameters over ~84M spectra in three ~28M-spectrum MGF folds and scoring clustered fraction, incorrectly-clustered fraction, and cluster completeness against MassIVE-KB peptide ground truth.

## When to use

Use when comparing spectrum-clustering tools on large-scale MS/MS peak-list data and you need per-tool cluster-quality metrics (clustered fraction, incorrect clustering, completeness) reproduced against a paper's reported comparison.

## When NOT to use

- The goal is to run GLEAMS itself for novel clustering rather than to benchmark/compare clustering tools against reported cluster-quality metrics.
- Input spectra lack peptide ground-truth identifications (e.g. MassIVE-KB labels), so incorrectly-clustered fraction and completeness cannot be computed.
- The data is small enough that clustering-tool performance at millions-of-spectra scale is not the question.

## Inputs

- Three test-set folds of ~28 million MS/MS spectra each (~84M total from 24 test-set datasets), derived from MassIVE-KB data
- MassIVE-KB identifications used as peptide ground-truth labels for the spectra
- MGF peak-list files exported from the test-set folds
- GLEAMS model weights (e.g. gleams_82c0124b.hdf5)

## Outputs

- Cluster labels per spectrum for each of the five tools (noise spectra marked, e.g. label -1)
- Per-tool metrics: fraction of clustered spectra, fraction incorrectly clustered, cluster completeness
- Aggregated results table comparing cluster-quality metrics across GLEAMS, falcon, MaRaCluster, MS-Cluster, and spectra-cluster

## How to apply

Retrieve the three randomly generated test-set folds of ~28 million spectra each (84 million MS/MS spectra total from 24 test-set datasets) from the MassIVE-KB data and export them as MGF peak-list files. Run the five clustering tools sequentially over the MGF folds, sweeping each tool's stated parameter grid — for GLEAMS this includes the embedding distance threshold (e.g. `--distance_threshold 0.3` after `gleams embed` on peak files). Assign cluster labels to all ~84 million spectra per tool, marking spectra in clusters smaller than the minimum size (e.g. minimum cluster size 2 for GLEAMS, label -1) as noise. Using MassIVE-KB identifications as peptide ground-truth labels, compute per tool the fraction of clustered spectra, the fraction incorrectly clustered (spectra grouped with spectra of a different peptide), and cluster completeness. Aggregate the three metrics per tool into a single results table and compare against the paper's reported cluster-quality values.

## Related tools

- **GLEAMS** (Clustering tool under evaluation; embeds MS/MS spectra into a 32-dimensional space and clusters them with a distance threshold (e.g. 0.3)) — https://github.com/bittremieux/GLEAMS
- **MSGF+** (Produced the 185 million PSMs at 1% FDR underlying the MassIVE-KB identifications used as ground truth)
- **ANN-SoLo** (Open modification search of GLEAMS cluster centroids (supplementary analysis of cluster quality))
- **Python** (Runtime for GLEAMS (Python 3.8, Linux, CUDA-enabled GPU) used to run embedding and clustering commands) — https://github.com/bittremieux/GLEAMS

## Examples

```
gleams embed *.mzML --embed_name GLEAMS_embed && gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.3
```

## Evaluation signals

- Per-tool fraction of clustered spectra reproduces the paper's reported values.
- Per-tool fraction of incorrectly clustered spectra (spectra grouped with spectra of a different peptide) matches the reported comparison.
- Cluster completeness relative to MassIVE-KB peptide ground-truth labels matches the reported values.
- Noise handling is consistent: spectra in clusters below the minimum cluster size are marked as noise (GLEAMS label -1, minimum cluster size 2).
- Spectrum counts reconcile: ~84 million spectra across the three ~28M-spectrum MGF folds are all assigned cluster labels per tool.

## Limitations

- The full test set (~84M spectra in three folds) is very large; the associated training data pipeline notes spectra are 'not readily available as a single download' and must be assembled from MassIVE-KB metadata and the MassIVE FTP server.
- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU; installing can hit Git LFS data-quota errors when downloading model weights, requiring manual download of the .hdf5 weights file.
- Ground truth derives from MassIVE-KB PSMs at 1% FDR (MSGF+), so incorrectly-clustered and completeness metrics are bounded by identification quality.

## Evidence

- [methods] The five clustering tools were evaluated on 84 million MS/MS spectra from 24 test-set datasets, split into three randomly generated folds of approximately 28 million spectra each and exported to MGF files.: [evidence span withheld — anchor status: not_found]
- [readme] GLEAMS provides the `gleams embed` command to convert MS/MS spectra in peak files to 32-dimensional embeddings.: [evidence span withheld — anchor status: external_doc]
- [readme] This will perform hierarchical clustering on the embeddings with the given distance threshold. The output will be written to the `GLEAMS_cluster.npy` NumPy file with cluster labels per embedding (`-1` indicates noise, minimum cluster size 2).: "This will perform hierarchical clustering on the embeddings with the given distance threshold."
- [intro] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together.: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together."
- [methods] All 185 million PSMs at 1% FDR obtained using MSGF+: "All 185 million PSMs at 1% FDR obtained using MSGF+"
