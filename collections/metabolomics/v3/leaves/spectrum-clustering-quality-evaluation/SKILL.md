---
name: spectrum-clustering-quality-evaluation
description: Use when comparing spectrum clustering results from multiple tools on
  a labeled MS/MS dataset and you need purity (incorrectly clustered fraction), cluster
  size distribution, completeness, or data-volume reduction at a comparable operating
  point.
when_to_use_negative:
- When you only need to run clustering itself (use the falcon CLI directly); this
  skill is for post-hoc evaluation of already-produced cluster assignments
- When spectra lack peptide identifications or FDR-filtered PSM labels, since incorrectly
  clustered spectra are determined by deviation from the most frequent peptide label
  per cluster, ignoring unidentified spectra
- When evaluating clustering on metabolomics or top-down data, since the evaluation
  settings (precursor charges 2 and 3, bottom-up proteomics preprocessing) are proteomics-specific
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Python 3.8
  role: Used with pandas to load cluster assignments, compute incorrectly-clustered
    fractions, cluster size distributions, and completeness per tool
  repo: https://github.com/bittremieux/falcon_notebooks
- name: falcon
  role: Clustering tool being evaluated; its '-1' noise label is retained and its
    ~1% incorrect operating point is cosine distance threshold 0.05
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Comparison tool evaluated at p-value/clustering threshold -30.0; singletons
    relabeled as noise
- name: MS-Cluster
  role: Comparison tool evaluated at mixture probability 0.00001; singletons relabeled
    as noise
- name: msCRUSH
  role: Comparison tool evaluated at cosine similarity threshold 0.8; singletons relabeled
    as noise
- name: spectra-cluster
  role: Comparison tool evaluated at clustering threshold 0.99999; singletons relabeled
    as noise
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:db8574db4758dbcf857ddf7254e6d1ddefd758d7a81781b1cb34799a006476a2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon_notebooks
  tools:
  - MS-Cluster
  - MaRaCluster
  - Python 3.8
  - falcon
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spectrum-clustering-quality-evaluation

## Summary

A method for evaluating and comparing MS/MS spectrum clustering tools by computing the fraction of incorrectly clustered spectra and characterizing cluster size distributions at a matched operating point (~1% incorrectly clustered spectra), as done for falcon, MS-Cluster, msCRUSH, MaRaCluster, and spectra-cluster.

## When to use

Use when comparing spectrum clustering results from multiple tools on a labeled MS/MS dataset and you need purity (incorrectly clustered fraction), cluster size distribution, completeness, or data-volume reduction at a comparable operating point.

## When NOT to use

- When you only need to run clustering itself (use the falcon CLI directly); this skill is for post-hoc evaluation of already-produced cluster assignments
- When spectra lack peptide identifications or FDR-filtered PSM labels, since incorrectly clustered spectra are determined by deviation from the most frequent peptide label per cluster, ignoring unidentified spectra
- When evaluating clustering on metabolomics or top-down data, since the evaluation settings (precursor charges 2 and 3, bottom-up proteomics preprocessing) are proteomics-specific

## Inputs

- Clustering result files (CSV of spectrum-to-cluster-label assignments) for each tool, e.g. from Zenodo doi:10.5281/zenodo.4721496
- Peptide-spectrum match identifications from MassIVE reanalysis RMSV000000091.3 (PSMs filtered at 1% FDR) used as peptide labels
- Spectrum metadata (precursor charge) from the human draft proteome dataset (PRIDE PXD000561, 25 million spectra)

## Outputs

- Summary table of per-tool fraction of incorrectly clustered spectra (verified to be approximately 1%), e.g. cluster_comparison_table.csv
- Per-tool cluster size distribution tabulation (showing e.g. clusters of 1000-10000 spectra vs. maximum 100 spectra)
- Per-tool data-volume reduction (number of non-noise clusters divided by total number of spectra)
- Per-tool cluster completeness based on conditional entropy given peptide assignments

## How to apply

Retrieve deposited cluster assignments (e.g., from Zenodo doi:10.5281/zenodo.4721496) and peptide-spectrum match identifications from MassIVE reanalysis RMSV000000091.3 filtered at 1% FDR. For each tool select the configuration yielding approximately 1% incorrectly clustered spectra (falcon cosine distance threshold 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH cosine similarity 0.8, spectra-cluster threshold 0.99999). Normalize noise handling: retain falcon's '-1' noise label and relabel singleton clusters from the other tools as noise; restrict to spectra with precursor charges 2 and 3 and to clusters with at least two spectra. For each cluster determine the most frequent peptide label and mark member spectra with deviating labels as incorrectly clustered, ignoring unidentified spectra; compute the fraction over identified spectra in non-noise clusters and verify it is approximately 1%. Tabulate the per-tool cluster size distribution and derive data-volume reduction as non-noise clusters divided by total spectra; optionally compute cluster completeness as one minus the conditional entropy of the cluster distribution given peptide assignments, normalized by the maximum possible entropy reduction.

## Related tools

- **Python 3.8** (Used with pandas to load cluster assignments, compute incorrectly-clustered fractions, cluster size distributions, and completeness per tool) — https://github.com/bittremieux/falcon_notebooks
- **falcon** (Clustering tool being evaluated; its '-1' noise label is retained and its ~1% incorrect operating point is cosine distance threshold 0.05) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Comparison tool evaluated at p-value/clustering threshold -30.0; singletons relabeled as noise)
- **MS-Cluster** (Comparison tool evaluated at mixture probability 0.00001; singletons relabeled as noise)
- **msCRUSH** (Comparison tool evaluated at cosine similarity threshold 0.8; singletons relabeled as noise)
- **spectra-cluster** (Comparison tool evaluated at clustering threshold 0.99999; singletons relabeled as noise)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Each tool's computed fraction of incorrectly clustered spectra at the chosen configuration is approximately 1%
- Cluster size distribution matches the reported qualitative pattern: falcon, msCRUSH, and to a lesser extent MS-Cluster produce several larger clusters of 1000-10000 spectra, while MaRaCluster and spectra-cluster predominantly produce clusters of at most 100 spectra
- Noise handling is consistent: falcon's '-1' label retained and singleton clusters from MaRaCluster, MS-Cluster, msCRUSH, and spectra-cluster relabeled as noise before comparison
- Evaluation restricted to precursor charges 2 and 3 and to valid clusters with at least two spectra; unidentified spectra ignored when scoring correctness
- Data-volume reduction equals number of non-noise clusters divided by total number of spectra and falcon's value exceeds the comparison tools at the matched operating point

## Limitations

- The incorrectly-clustered metric depends on FDR-filtered PSM identifications (1% FDR, 10487235 spectrum identifications here); spectra without identifications are ignored
- The evaluation is indirect and does not measure performance on downstream tasks; cluster representative formation strategy (medoid vs consensus spectrum) effects were not evaluated
- Results are dataset- and threshold-specific: the ~1% operating points depend on the 25-million-spectrum human draft proteome dataset and each tool's parameterization
- Preprocessing choices (e.g. falcon's square root intensity scaling) influence the cosine distance threshold eps, so thresholds are not directly transferable across pipelines

## Evidence
