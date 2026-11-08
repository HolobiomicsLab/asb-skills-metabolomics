---
name: cluster-completeness-entropy-computation
description: Use when comparing spectrum clustering tools (falcon, MS-Cluster, msCRUSH,
  MaRaCluster, spectra-cluster) and you need a normalized, information-theoretic purity
  metric per tool at a matched fraction of incorrectly clustered spectra (~1%).
when_to_use_negative:
- No peptide-level ground truth is available — conditional entropy of the cluster
  distribution given peptide assignments requires PSM identifications to label clusters;
  use it only when peptide labels exist.
- The goal is computational runtime benchmarking rather than clustering-quality assessment
  — the article separates quality metrics (completeness, incorrectly-clustered fraction)
  from performance comparison.
- Analyzing metabolomics or top-down data with falcon defaults — preprocessing settings
  (min_peaks=5, min_mz_range=250, min_mz=101, max_mz=500) are intended for bottom-up
  proteomics and must be adjusted, invalidating the ~1% operating-point thresholds.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: falcon
  role: Spectrum clustering tool being evaluated; its cosine distance 0.05 result
    provides the ~1% incorrectly-clustered operating point and its completeness is
    compared against alternatives
  repo: https://github.com/bittremieux/falcon
- name: MS-Cluster
  role: Comparison clustering tool; its mixture probability 0.00001 result is evaluated
    with the completeness metric
- name: msCRUSH
  role: Comparison clustering tool; its similarity 0.8 result is evaluated with the
    completeness metric
- name: MaRaCluster
  role: Comparison clustering tool; its threshold -30.0 result is evaluated with the
    completeness metric
- name: spectra-cluster
  role: Comparison clustering tool; its threshold 0.99999 result is evaluated with
    the completeness metric
- name: pandas
  role: Python library used to compute clustered/incorrectly-clustered fractions and
    assemble per-tool metrics into cluster_comparison_table.csv
- name: Python 3.8
  role: Runtime for the evaluation workflow; falcon itself was implemented in Python
    3.8 and requires 3.8+
  repo: https://github.com/bittremieux/falcon
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:af334d646e9cf8e61432114a86bfdfe19b29a0987cd151707b4ee6a5a0e177c6
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
  - pandas
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# cluster-completeness-entropy-computation

## Summary

An information-theoretic evaluation of MS/MS spectrum clustering quality: cluster completeness is computed as one minus the conditional entropy of the cluster distribution given peptide assignments, normalized by the maximum entropy reduction those peptide assignments could provide. It enables fair cross-tool comparison of clustering purity at a matched (~1%) incorrectly-clustered-spectra operating point.

## When to use

Use when comparing spectrum clustering tools (falcon, MS-Cluster, msCRUSH, MaRaCluster, spectra-cluster) and you need a normalized, information-theoretic purity metric per tool at a matched fraction of incorrectly clustered spectra (~1%).

## When NOT to use

- No peptide-level ground truth is available — conditional entropy of the cluster distribution given peptide assignments requires PSM identifications to label clusters; use it only when peptide labels exist.
- The goal is computational runtime benchmarking rather than clustering-quality assessment — the article separates quality metrics (completeness, incorrectly-clustered fraction) from performance comparison.
- Analyzing metabolomics or top-down data with falcon defaults — preprocessing settings (min_peaks=5, min_mz_range=250, min_mz=101, max_mz=500) are intended for bottom-up proteomics and must be adjusted, invalidating the ~1% operating-point thresholds.

## Inputs

- Clustering results for all five tools from Zenodo doi:10.5281/zenodo.4721496 (per-spectrum cluster labels)
- PSM identifications with peptide labels from MassIVE reanalysis RMSV000000091.3, filtered at 1% FDR (10,487,235 spectrum identifications)
- Human draft proteome dataset by Kim et al. (PRIDE PXD000561, 25 million spectra) as the underlying spectra

## Outputs

- Per-tool cluster completeness values (normalized 1 - conditional entropy given peptide assignments)
- Per-tool fraction of clustered spectra and fraction of incorrectly clustered spectra
- Per-tool cluster size distribution and data-volume reduction (non-noise clusters / total spectra)
- Combined metrics table saved as cluster_comparison_table.csv

## How to apply

Download deposited clustering results for all five tools (Zenodo doi:10.5281/zenodo.4721496) and peptide labels from PSM identifications (MassIVE reanalysis RMSV000000091.3, filtered at 1% FDR, 10,487,235 spectrum identifications). Restrict to precursor charges 2 and 3, require valid clusters to contain at least two spectra, and relabel singleton clusters as noise so all tools are compared consistently. Select each tool's ~1% incorrectly-clustered configuration (falcon cosine distance 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH similarity 0.8, spectra-cluster threshold 0.99999) and use pandas to compute the fraction of clustered and incorrectly clustered spectra per tool. Compute completeness per tool as one minus the conditional entropy of the cluster distribution given the peptide assignments, normalized by the maximum entropy reduction the peptide assignments could provide, and tabulate alongside cluster size distributions and data-volume reduction (non-noise clusters divided by total spectra) into cluster_comparison_table.csv.

## Related tools

- **falcon** (Spectrum clustering tool being evaluated; its cosine distance 0.05 result provides the ~1% incorrectly-clustered operating point and its completeness is compared against alternatives) — https://github.com/bittremieux/falcon
- **MS-Cluster** (Comparison clustering tool; its mixture probability 0.00001 result is evaluated with the completeness metric)
- **msCRUSH** (Comparison clustering tool; its similarity 0.8 result is evaluated with the completeness metric)
- **MaRaCluster** (Comparison clustering tool; its threshold -30.0 result is evaluated with the completeness metric)
- **spectra-cluster** (Comparison clustering tool; its threshold 0.99999 result is evaluated with the completeness metric)
- **pandas** (Python library used to compute clustered/incorrectly-clustered fractions and assemble per-tool metrics into cluster_comparison_table.csv)
- **Python 3.8** (Runtime for the evaluation workflow; falcon itself was implemented in Python 3.8 and requires 3.8+) — https://github.com/bittremieux/falcon

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Each tool's selected configuration yields approximately 1% incorrectly clustered spectra (e.g., falcon at cosine distance threshold 0.05).
- Singleton clusters are relabeled as noise and only clusters with at least two spectra are counted, so valid clusters per tool are consistently defined.
- Falcon and msCRUSH achieve the highest completeness at a low number of incorrectly clustered spectra, while MaRaCluster outperforms the other tools in completeness at slightly higher incorrectly-clustered rates.
- Cluster size distributions confirm the pattern: MaRaCluster and spectra-cluster predominantly produce small clusters of at most 100 spectra, whereas falcon, msCRUSH, and to a lesser extent MS-Cluster produce several larger clusters of 1000 to 10000 spectra.
- The combined metrics table cluster_comparison_table.csv is produced with completeness, fraction clustered, incorrectly-clustered fraction, and data-volume reduction per tool.

## Limitations

- The metric is sensitive to the quality of peptide labels: PSMs were filtered at 1% FDR, so mislabeled spectra propagate into the conditional entropy estimate.
- Performance was evaluated on a single human draft proteome dataset (Kim et al., 25 million spectra, precursor charges 2 and 3); transferability to other data types is not established.
- The effect of cluster representative formation strategy (e.g., medoid selection vs. consensus spectrum) on downstream applications was not evaluated, so completeness does not predict downstream identification gains.
- Preprocessing (peak removal, square-root intensity scaling) influences the cosine threshold eps and therefore the ~1% operating point; settings must be matched per tool.

## Evidence

- [other] Compute cluster completeness per tool as one minus the conditional entropy of the cluster distribution given the peptide assignments, normalized by the maximum entropy reduction the peptide assignments could provide.: [evidence span withheld — anchor status: not_found]
- [methods] Peptide-spectrum matches (PSMs) were filtered at 1% false discovery rate, resulting in 10487235 spectrum identifications.: "Peptide-spectrum matches (PSMs) were filtered at 1% false discovery rate, resulting in 10487235 spectrum identifications."
- [methods] The clustering result with approximately 1% incorrectly clustered spectra was obtained with a cosine distance threshold of 0.05.: "The clustering result with approximately 1% incorrectly clustered spectra was obtained with a cosine distance threshold of 0.05."
- [results] At a low number of incorrectly clustered spectra, falcon and msCRUSH achieve the highest completeness, while MaRaCluster outperforms the other clustering tools in terms of completeness at slightly: "At a low number of incorrectly clustered spectra, falcon and msCRUSH achieve the highest completeness, while MaRaCluster outperforms the other clustering tools in terms of completeness at slightly"
- [results] MaRaCluster and spectra-cluster predominantly produce small clusters of at most 100 spectra, whereas falcon, msCRUSH, and to a lesser extent MS-Cluster produce several larger clusters of 1000 to 10000 spectra.: [evidence span withheld — anchor status: not_found]
