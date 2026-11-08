---
name: clustering-quality-metric-tabulation
description: Decision support for choosing a spectrum clustering tool when you have
  cluster assignments from multiple tools plus peptide-labeled PSMs (e.g., 1% FDR)
  and need to compare cluster size distributions, purity, completeness, and data-volume
  reduction at a comparable error rate.
when_to_use_negative:
- You lack peptide-labeled PSM identifications for the same spectra, so incorrectly
  clustered spectra cannot be defined and purity-based comparison is impossible.
- Tools are evaluated at arbitrary, non-matched error rates (e.g., default thresholds),
  making cluster size distributions and completeness incomparable across tools.
- The goal is computational runtime benchmarking rather than clustering-quality characterization;
  this skill tabulates quality metrics, not wall-clock performance.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: falcon
  role: Produces cosine-distance/DBSCAN clustering results whose cluster sizes, purity,
    and completeness are tabulated against the comparison tools
  repo: https://github.com/bittremieux/falcon
- name: MS-Cluster
  role: Comparison clustering tool whose results at mixture probability 0.00001 (~1%
    incorrectly clustered) are tabulated
- name: spectra-cluster
  role: Comparison clustering tool whose results at threshold 0.99999 (~1% incorrectly
    clustered) are tabulated
- name: MaRaCluster
  role: Comparison clustering tool whose results at threshold -30.0 (~1% incorrectly
    clustered) are tabulated
- name: msCRUSH
  role: Comparison clustering tool whose results at similarity 0.8 (~1% incorrectly
    clustered) are tabulated
- name: Python 3.8
  role: Implements the pandas-based metric computation (fraction clustered, incorrectly
    clustered, conditional-entropy completeness, data-volume reduction)
  repo: https://github.com/bittremieux/falcon_notebooks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:8f07d2165a8ed8c4254273bb2cf0e104ae52dbdbe10e9f4e6102df0ba252119b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - MS-Cluster
  - MaRaCluster
  - Python 3.8
  - falcon
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# clustering-quality-metric-tabulation

## Summary

Compares MS/MS spectrum clustering tools by tabulating cluster size distributions, fraction of incorrectly clustered spectra (via PSM peptide labels), cluster completeness (entropy-based), and data-volume reduction at a matched ~1% incorrectly-clustered rate, as done for falcon, MS-Cluster, msCRUSH, MaRaCluster, and spectra-cluster on the 25-million-spectrum human draft proteome dataset.

## When to use

Decision support for choosing a spectrum clustering tool when you have cluster assignments from multiple tools plus peptide-labeled PSMs (e.g., 1% FDR) and need to compare cluster size distributions, purity, completeness, and data-volume reduction at a comparable error rate.

## When NOT to use

- You lack peptide-labeled PSM identifications for the same spectra, so incorrectly clustered spectra cannot be defined and purity-based comparison is impossible.
- Tools are evaluated at arbitrary, non-matched error rates (e.g., default thresholds), making cluster size distributions and completeness incomparable across tools.
- The goal is computational runtime benchmarking rather than clustering-quality characterization; this skill tabulates quality metrics, not wall-clock performance.

## Inputs

- Clustering result files for each tool mapping MS/MS spectra to cluster labels (mzML/mzXML/MGF-derived spectra, e.g., falcon CSV cluster assignments from Zenodo doi:10.5281/zenodo.4721496)
- PSM identifications with peptide labels filtered at 1% FDR (e.g., MassIVE reanalysis RMSV000000091.3)
- Per-tool clustering configurations at matched error rates (falcon cosine distance 0.05; MaRaCluster -30.0; MS-Cluster mixture probability 0.00001; msCRUSH similarity 0.8; spectra-cluster threshold 0.99999)

## Outputs

- cluster_comparison_table.csv with per-tool metrics: fraction clustered, fraction incorrectly clustered, cluster completeness, cluster size distribution, data-volume reduction
- Per-tool cluster size distribution tables (small clusters ≤100 spectra vs. large clusters 1000–10000 spectra)

## How to apply

Download precomputed clustering results (e.g., from Zenodo doi:10.5281/zenodo.4721496) and PSM identifications (e.g., MassIVE reanalysis RMSV000000091.3, filtered at 1% FDR) to obtain peptide labels. Restrict spectra to precursor charges 2 and 3, treat clusters with fewer than two spectra as invalid, and relabel singletons as noise for consistent cross-tool comparison. For each tool select the configuration giving approximately 1% incorrectly clustered spectra (falcon cosine distance 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH similarity 0.8, spectra-cluster threshold 0.99999) and compute the fraction of clustered spectra and incorrectly clustered spectra per tool using pandas. Compute cluster completeness as one minus the conditional entropy of the cluster distribution given peptide assignments, normalized by the maximum entropy reduction the peptide assignments could provide, and derive data-volume reduction as non-noise clusters divided by total spectra. Tabulate per-tool cluster size distributions and assemble all metrics into a single table (cluster_comparison_table.csv) to expose trade-offs, e.g., whether a tool produces many small clusters (≤100 spectra) versus a few large clusters (1000–10000 spectra) at equal purity.

## Related tools

- **falcon** (Produces cosine-distance/DBSCAN clustering results whose cluster sizes, purity, and completeness are tabulated against the comparison tools) — https://github.com/bittremieux/falcon
- **MS-Cluster** (Comparison clustering tool whose results at mixture probability 0.00001 (~1% incorrectly clustered) are tabulated)
- **spectra-cluster** (Comparison clustering tool whose results at threshold 0.99999 (~1% incorrectly clustered) are tabulated)
- **MaRaCluster** (Comparison clustering tool whose results at threshold -30.0 (~1% incorrectly clustered) are tabulated)
- **msCRUSH** (Comparison clustering tool whose results at similarity 0.8 (~1% incorrectly clustered) are tabulated)
- **Python 3.8** (Implements the pandas-based metric computation (fraction clustered, incorrectly clustered, conditional-entropy completeness, data-volume reduction)) — https://github.com/bittremieux/falcon_notebooks

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Each tool's tabulated configuration yields approximately 1% incorrectly clustered spectra (e.g., falcon at cosine distance threshold 0.05).
- Valid clusters contain at least two spectra; singleton clusters are relabeled as noise consistently across all five tools.
- Cluster completeness equals one minus the conditional entropy of cluster distribution given peptide assignments, normalized by the maximum achievable entropy reduction.
- Data-volume reduction equals the number of non-noise clusters divided by the total number of spectra.
- The assembled table (cluster_comparison_table.csv) shows the expected pattern: MaRaCluster and spectra-cluster produce predominantly small clusters (≤100 spectra) while falcon and msCRUSH produce several large clusters (1000–10000 spectra) at the matched error rate.

## Limitations

- Comparison is sensitive to each tool's threshold choice; only configurations at a comparable (~1%) incorrectly-clustered rate are directly comparable.
- Spectrum preprocessing (min_peaks, min_mz_range, scaling) changes spectral characteristics and thus affects the cosine distance threshold eps and resulting purity; the effect of preprocessing on various similarity measures was not evaluated.
- Performance is evaluated on clustering properties, not on downstream identification tasks, and the effect of the cluster representative formation strategy (medoid vs. consensus) was not assessed.

## Evidence

- [results] At approximately 1% incorrectly clustered spectra, MaRaCluster and spectra-cluster predominantly produce small clusters: [evidence span withheld — anchor status: not_found]
