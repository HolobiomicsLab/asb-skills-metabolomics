---
name: peptide-ground-truth-label-mapping
description: Use when evaluating spectrum clustering results on data with known peptide
  identifications, needing per-cluster peptide label assignment to compute incorrectly-clustered
  fraction and completeness.
when_to_use_negative:
- Input spectra have no peptide identifications available (e.g. non-MassIVE-KB data),
  so no ground-truth labels can be joined
- The goal is de novo clustering quality without reference identifications — this
  skill only measures agreement with known PSMs
- Working on the 185M-PSM full search set rather than the labeled test folds — this
  skill targets the 84M-spectrum test-set evaluation
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: GLEAMS
  role: Produces the cluster labels (GLEAMS_cluster.npy) whose members are annotated
    with peptide ground-truth labels
  repo: https://github.com/bittremieux/GLEAMS
- name: MSGF+
  role: Source of the 185 million PSMs at 1% FDR underlying the MassIVE-KB ground-truth
    peptide identifications
- name: Python
  role: Joins cluster labels to peptide identifications and computes clustered-spectra
    fraction, incorrect-clustering fraction, and completeness
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-022-01496-1
    title: A learned embedding for efficient joint analysis of millions of mass spectra
  source_content_hash: sha256:e41a998c8322e0adfd436df0732f455adaa2ec4d86d1bb501456cef1aa3d49ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GLEAMS
  tools:
  - ANN-SoLo
  - GLEAMS
  - MSGF+
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# peptide-ground-truth-label-mapping

## Summary

Maps clustered MS/MS spectra to peptide ground-truth labels derived from MassIVE-KB identifications (PSMs at 1% FDR obtained with MSGF+) so that clustering quality can be quantified as fraction incorrectly clustered and cluster completeness.

## When to use

Use when evaluating spectrum clustering results on data with known peptide identifications, needing per-cluster peptide label assignment to compute incorrectly-clustered fraction and completeness.

## When NOT to use

- Input spectra have no peptide identifications available (e.g. non-MassIVE-KB data), so no ground-truth labels can be joined
- The goal is de novo clustering quality without reference identifications — this skill only measures agreement with known PSMs
- Working on the 185M-PSM full search set rather than the labeled test folds — this skill targets the 84M-spectrum test-set evaluation

## Inputs

- Cluster label file per spectrum, e.g. GLEAMS_cluster.npy (-1 = noise, minimum cluster size 2)
- MassIVE-KB metadata TSV with peptide identifications ('filename' and 'scan' columns) from the Human HCD Spectral Library / All Candidate library spectra download
- MGF peak-list files of the three ~28M-spectrum test-set folds (24 test-set datasets)

## Outputs

- Per-cluster peptide ground-truth label assignment
- Fraction of clustered spectra, fraction incorrectly clustered (spectra grouped with a different peptide), and cluster completeness per tool
- Aggregated cluster-quality results table across the three ~28M-spectrum folds

## How to apply

Take the peptide identifications from the MassIVE-KB resource — the metadata TSV of candidate library spectra containing the 'filename' and 'scan' columns — and map each spectrum to its peptide by joining cluster membership against those identification records. Assign cluster labels produced by the clustering tool (e.g. GLEAMS outputs GLEAMS_cluster.npy where -1 denotes noise), then treat spectra grouped in the same cluster but bearing different peptide labels as incorrectly clustered. Compute three metrics per tool: the fraction of clustered spectra, the fraction incorrectly clustered, and cluster completeness relative to the peptide ground-truth labels. For the benchmark scale described, this is done over ~84 million test spectra from 24 datasets split into three random folds of ~28 million spectra each, aggregated into one results table per tool.

## Related tools

- **GLEAMS** (Produces the cluster labels (GLEAMS_cluster.npy) whose members are annotated with peptide ground-truth labels) — https://github.com/bittremieux/GLEAMS
- **MSGF+** (Source of the 185 million PSMs at 1% FDR underlying the MassIVE-KB ground-truth peptide identifications)
- **Python** (Joins cluster labels to peptide identifications and computes clustered-spectra fraction, incorrect-clustering fraction, and completeness)

## Examples

```
gleams cluster --embed_name GLEAMS_embed --cluster_name GLEAMS_cluster --distance_threshold 0.3
```

## Evaluation signals

- Every clustered spectrum (label != -1) resolves to a non-empty peptide ground-truth label after the MassIVE-KB filename/scan join
- Fraction incorrectly clustered equals the share of within-cluster spectra pairs with discordant peptide labels and matches the paper's reported values
- Cluster completeness (spectra of the same peptide recovered in one cluster) reproduces the reported cluster-quality comparison
- Noise spectra match clusters smaller than the minimum cluster size (2 for GLEAMS)
- Aggregated metrics are consistent across the three ~28M-spectrum folds

## Limitations

- Ground truth is limited to identified spectra; only PSMs at 1% FDR from MSGF+ searches are available as labels
- MassIVE-KB labels are a subset of identifications (maximum top 100 PSMs for 2.1 million unique precursors), so label coverage is partial
- Requires Python 3.8, a Linux operating system, and a CUDA-enabled GPU to run the GLEAMS pipeline feeding the evaluation

## Evidence

- [intro] GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space: "GLEAMS encodes mass spectra as vectors of features and feeds them to a neural network to embed them into a 32-dimensional space"
- [methods] All 185 million PSMs at 1% FDR obtained using MSGF+: "All 185 million PSMs at 1% FDR obtained using MSGF+"
- [readme] GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]: "GLEAMS was trained on 30 million PSMs from the [MassIVE-KB (v1) dataset]"
- [readme] This will give you a zipped TSV file with the metadata and peptide identifications for all 30 million PSMs.: "This will give you a zipped TSV file with the metadata and peptide identifications for all 30 million PSMs."
- [readme] These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource: "These 30 million PSMs are a subset of all spectrum identifications that were obtained during compilation of the MassIVE-KB resource"
