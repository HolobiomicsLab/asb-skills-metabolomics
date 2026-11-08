---
name: clustering-quality-metric-computation
description: Use when you have cluster labels assigned over millions of MS/MS spectra
  together with peptide identifications and need to quantify clustered fraction, incorrect
  clustering, and completeness for tool comparison.
when_to_use_negative:
- Spectra lack peptide ground-truth identifications, so the incorrectly-clustered
  and completeness metrics cannot be computed
- The goal is de novo clustering quality assessment on small datasets where benchmark-scale
  metrics are unnecessary; the evaluation setup targets ~84M spectra in 24 datasets
- Input spectra have not yet been clustered — embed and cluster first (e.g. 'gleams
  embed' then 'gleams cluster') before computing metrics
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GLEAMS
  role: Produces 32-dimensional spectrum embeddings and hierarchical cluster labels
    with a 0.3 distance threshold whose quality is then measured
  repo: https://github.com/bittremieux/GLEAMS
- name: MSGF+
  role: Source of the 185 million PSMs at 1% FDR used to derive MassIVE-KB peptide
    ground-truth labels for the metric computation
- name: ANN-SoLo
  role: Used for an open modification search of GLEAMS cluster centroids in supplementary
    analyses
- name: Python
  role: Scripting environment (Python 3.8 required) for computing the fraction clustered,
    incorrectly clustered, and completeness metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:2322e2d97628cccce331cdbf66f58cc9462d423c98ac8d45205078593088eddb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - MSGF+
  - Python
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# clustering-quality-metric-computation

## Summary

Compute per-tool cluster-quality metrics (fraction of clustered spectra, fraction incorrectly clustered, and cluster completeness against peptide ground-truth labels) to benchmark MS/MS spectrum clustering tools such as GLEAMS, falcon, MaRaCluster, MS-Cluster, and spectra-cluster.

## When to use

Use when you have cluster labels assigned over millions of MS/MS spectra together with peptide identifications and need to quantify clustered fraction, incorrect clustering, and completeness for tool comparison.

## When NOT to use

- Spectra lack peptide ground-truth identifications, so the incorrectly-clustered and completeness metrics cannot be computed
- The goal is de novo clustering quality assessment on small datasets where benchmark-scale metrics are unnecessary; the evaluation setup targets ~84M spectra in 24 datasets
- Input spectra have not yet been clustered — embed and cluster first (e.g. 'gleams embed' then 'gleams cluster') before computing metrics

## Inputs

- MGF peak-list files of three ~28M-spectrum test-set folds (~84 million MS/MS spectra from 24 test-set datasets)
- Cluster label files per spectrum per tool (e.g. GLEAMS_cluster.npy with -1 indicating noise)
- Peptide ground-truth identifications from MassIVE-KB (e.g. metadata TSV with 'filename' and 'scan' columns)

## Outputs

- Per-tool results table with fraction of clustered spectra, fraction incorrectly clustered, and cluster completeness
- Cross-tool cluster-quality comparison table benchmarking GLEAMS versus falcon, MaRaCluster, MS-Cluster, and spectra-cluster

## How to apply

Run each clustering tool (GLEAMS, falcon, MaRaCluster, MS-Cluster, spectra-cluster) sequentially over the three ~28M-spectrum MGF folds of the 84M-spectrum MassIVE-KB test set, sweeping each tool's stated parameter grid (e.g. GLEAMS's 0.3 embedding distance threshold). Assign cluster labels to all ~84 million spectra per tool, marking spectra in clusters smaller than the minimum size as noise (GLEAMS uses -1 as the noise label with minimum cluster size 2). Using the peptide ground-truth labels from the MassIVE-KB identifications, compute three metrics per tool: the fraction of clustered spectra, the fraction incorrectly clustered (spectra grouped with spectra of a different peptide), and cluster completeness relative to the ground truth. Aggregate the three metrics per tool into a single results table and compare against the paper's reported cluster-quality values.

## Related tools

- **GLEAMS** (Produces 32-dimensional spectrum embeddings and hierarchical cluster labels with a 0.3 distance threshold whose quality is then measured) — https://github.com/bittremieux/GLEAMS
- **MSGF+** (Source of the 185 million PSMs at 1% FDR used to derive MassIVE-KB peptide ground-truth labels for the metric computation)
- **ANN-SoLo** (Used for an open modification search of GLEAMS cluster centroids in supplementary analyses)
- **Python** (Scripting environment (Python 3.8 required) for computing the fraction clustered, incorrectly clustered, and completeness metrics)

## Examples

```
gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.3
```

## Evaluation signals

- Per-tool fraction of clustered spectra reproduces the paper's reported cluster-quality comparison
- Per-tool fraction incorrectly clustered (spectra grouped with a different peptide) matches reported values
- Per-tool cluster completeness relative to MassIVE-KB peptide ground-truth labels matches reported values
- Noise handling is consistent: spectra in clusters smaller than the minimum size are labeled noise (-1 for GLEAMS, minimum cluster size 2)
- Counts reconcile: the sum of clustered and noise spectra equals the ~84M total input spectra across the three folds

## Limitations

- GLEAMS requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU
- Model weights can exceed Git LFS bandwidth, requiring manual download of the GLEAMS weights (gleams_82c0124b.hdf5) from the release
- Ground-truth labels depend on MassIVE-KB identifications at 1% FDR, so misassignments in PSMs propagate into the incorrectly-clustered metric

## Evidence

- [methods] The five clustering tools were evaluated on 84 million MS/MS spectra from 24 test-set datasets, split into three randomly generated folds of approximately 28 million spectra each and exported to MGF files.: [evidence span withheld — anchor status: not_found]
- [readme] gleams cluster distance threshold: "gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.3"
- [readme] cluster output labels: "The output will be written to the `GLEAMS_cluster.npy` NumPy file with cluster labels per embedding (`-1` indicates noise, minimum cluster size 2)."
- [intro] embedding space description: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space in which spectra generated by the same peptide are close together."
- [methods] MSGF+ PSMs: "All 185 million PSMs at 1% FDR obtained using MSGF+"
