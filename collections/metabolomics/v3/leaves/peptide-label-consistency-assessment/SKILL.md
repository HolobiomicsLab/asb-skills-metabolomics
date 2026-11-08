---
name: peptide-label-consistency-assessment
description: Use when spectrum clustering results (cluster assignments per MS/MS spectrum)
  must be validated against peptide-spectrum match labels to measure cluster purity,
  e.g. tuning thresholds to ~1% incorrectly clustered spectra.
when_to_use_negative:
- Clustering metabolomics or top-down data with falcon default preprocessing — README
  states defaults are intended for bottom-up proteomics and settings (min_peaks, min_mz_range)
  likely need adjustment
- The spectra lack peptide identifications (PSM labels), so incorrectly-clustered
  fractions and completeness cannot be computed
- The goal is computational runtime benchmarking rather than clustering-quality assessment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Spectrum clustering tool whose results at cosine distance threshold 0.05 yield
    ~1% incorrectly clustered spectra for label-consistency evaluation
  repo: https://github.com/bittremieux/falcon
- name: MS-Cluster
  role: Comparison clustering tool evaluated at mixture probability 0.00001
- name: MaRaCluster
  role: Comparison clustering tool evaluated at threshold -30.0
- name: msCRUSH
  role: Comparison clustering tool evaluated at similarity 0.8
- name: spectra-cluster
  role: Comparison clustering tool evaluated at threshold 0.99999
- name: Python 3.8
  role: Implementation language; pandas used to compute metrics and assemble cluster_comparison_table.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:0469ff931140c01ba168409142f1b42a29655630c1a26f548c76971d8b32a44b
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Peptide-label consistency assessment

## Summary

Quantify how internally consistent spectrum clusters are with peptide identifications by computing the fraction of incorrectly clustered spectra and cluster completeness (normalized conditional entropy of cluster assignments given peptide labels) for clustering results such as falcon, MS-Cluster, MaRaCluster, msCRUSH, and spectra-cluster.

## When to use

Use when spectrum clustering results (cluster assignments per MS/MS spectrum) must be validated against peptide-spectrum match labels to measure cluster purity, e.g. tuning thresholds to ~1% incorrectly clustered spectra.

## When NOT to use

- Clustering metabolomics or top-down data with falcon default preprocessing — README states defaults are intended for bottom-up proteomics and settings (min_peaks, min_mz_range) likely need adjustment
- The spectra lack peptide identifications (PSM labels), so incorrectly-clustered fractions and completeness cannot be computed
- The goal is computational runtime benchmarking rather than clustering-quality assessment

## Inputs

- Cluster assignment files (CSV: spectrum id + cluster label per tool) from falcon, MS-Cluster, MaRaCluster, msCRUSH, spectra-cluster (Zenodo doi:10.5281/zenodo.4721496)
- PSM identifications at 1% FDR from MassIVE reanalysis RMSV000000091.3 mapping spectra to peptide labels
- Human draft proteome MS/MS dataset (Kim et al., PRIDE PXD000561, ~25 million spectra)

## Outputs

- Per-tool table of fraction clustered, fraction incorrectly clustered, cluster completeness, cluster size distribution, and data-volume reduction
- cluster_comparison_table.csv

## How to apply

Download deposited clustering results (e.g. from Zenodo doi:10.5281/zenodo.4721496) and PSM identifications filtered at 1% FDR from MassIVE reanalysis RMSV000000091.3 to obtain peptide labels. Restrict the comparison to MS/MS spectra with precursor charges 2 and 3, require valid clusters to contain at least two spectra, and relabel singleton clusters as noise so all tools are compared consistently. Select each tool's clustering at its ~1% incorrectly-clustered configuration (falcon cosine distance 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH similarity 0.8, spectra-cluster threshold 0.99999), then compute the fraction of clustered spectra, fraction of incorrectly clustered spectra, and cluster completeness as one minus the conditional entropy of the cluster distribution given peptide assignments, normalized by the maximum entropy reduction the peptide assignments could provide. Tabulate per-tool metrics including cluster size distributions and data-volume reduction (non-noise clusters / total spectra) with pandas and save as cluster_comparison_table.csv.

## Related tools

- **falcon** (Spectrum clustering tool whose results at cosine distance threshold 0.05 yield ~1% incorrectly clustered spectra for label-consistency evaluation) — https://github.com/bittremieux/falcon
- **MS-Cluster** (Comparison clustering tool evaluated at mixture probability 0.00001)
- **MaRaCluster** (Comparison clustering tool evaluated at threshold -30.0)
- **msCRUSH** (Comparison clustering tool evaluated at similarity 0.8)
- **spectra-cluster** (Comparison clustering tool evaluated at threshold 0.99999)
- **Python 3.8** (Implementation language; pandas used to compute metrics and assemble cluster_comparison_table.csv)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.05
```

## Evaluation signals

- falcon clustering at cosine distance threshold 0.05 yields approximately 1% incorrectly clustered spectra
- PSM identifications filtered at 1% FDR should total 10,487,235 spectrum identifications for label assignment
- Only precursor charges 2 and 3 are retained; singleton clusters are relabeled as noise across all tools
- Cluster completeness computed as 1 minus conditional entropy of clusters given peptide labels, normalized by maximum achievable entropy reduction
- Expected qualitative result: MaRaCluster and spectra-cluster produce small clusters (≤100 spectra) while falcon, msCRUSH, and MS-Cluster produce clusters of 1000–10000 spectra at comparable ~1% incorrect rate

## Limitations

- Performance is evaluated indirectly via label consistency rather than directly on downstream identification tasks
- The effect of chimeric spectra on clustering and identification remains an open research question
- The full effect of MS/MS spectrum preprocessing on various similarity measures was not evaluated
- The strategy used to form cluster representatives (medoid vs consensus spectrum) may affect downstream application performance and was not evaluated

## Evidence
