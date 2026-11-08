---
name: deposited-omics-result-reanalysis
description: Use when a task requires comparing clustering-quality characteristics
  of publicly deposited spectrum clustering results against peptide labels, rather
  than re-running the clustering tools on raw mzML/MGF files.
when_to_use_negative:
- The clustering has not been performed yet and must be run from raw peak files (mzML,
  mzXML, or MGF) with falcon's feature hashing, nearest neighbor indexing, and DBSCAN
  pipeline — this skill only reanalyzes deposited clustering outputs
- The question is about clustering computational runtime or memory performance on
  raw spectra rather than clustering-quality characteristics of finished clustering
  results
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Deposited clustering result whose cluster size distribution and ~1% incorrect
    configuration (cosine distance 0.05) is reanalyzed and benchmarked against the
    comparison tools
  repo: https://github.com/bittremieux/falcon
- name: MS-Cluster
  role: Comparison tool whose deposited clustering output at mixture probability 0.00001
    is reanalyzed for cluster size distribution and purity
- name: msCRUSH
  role: Comparison tool whose deposited clustering output at similarity threshold
    0.8 is reanalyzed
- name: MaRaCluster
  role: Comparison tool whose deposited clustering output at threshold -30.0 is reanalyzed;
    known to produce small clusters of at most 100 spectra at ~1% incorrect rate
- name: spectra-cluster
  role: Comparison tool whose deposited clustering output at threshold 0.99999 is
    reanalyzed
- name: falcon_notebooks
  role: Companion repository containing the code used to compare the clustering tools
    and generate the comparison figures
  repo: https://github.com/bittremieux/falcon_notebooks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:a3ed7ffbb01bbaec344a1196b3ec81956b4a046c51ad773d23a1ec01fe4bd223
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - MS-Cluster
  - MaRaCluster
  - Python 3.8
  - falcon
  - falcon_notebooks
  - msCRUSH
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# deposited-omics-result-reanalysis

## Summary

Reanalysis skill that retrieves pre-computed MS/MS spectrum clustering results deposited on Zenodo (doi:10.5281/zenodo.4721496) and PSM identifications from MassIVE reanalysis RMSV000000091.3, then recomputes cluster size distributions, clustering purity, completeness, and data-volume reduction across five clustering tools (falcon, MS-Cluster, msCRUSH, MaRaCluster, spectra-cluster) on the 25-million-spectrum human draft proteome dataset without re-running the clustering itself.

## When to use

Use when a task requires comparing clustering-quality characteristics of publicly deposited spectrum clustering results against peptide labels, rather than re-running the clustering tools on raw mzML/MGF files.

## When NOT to use

- The clustering has not been performed yet and must be run from raw peak files (mzML, mzXML, or MGF) with falcon's feature hashing, nearest neighbor indexing, and DBSCAN pipeline — this skill only reanalyzes deposited clustering outputs
- The question is about clustering computational runtime or memory performance on raw spectra rather than clustering-quality characteristics of finished clustering results

## Inputs

- Deposited clustering result files for falcon, MS-Cluster, msCRUSH, MaRaCluster, and spectra-cluster from Zenodo doi:10.5281/zenodo.4721496
- PSM spectrum identifications with peptide labels from MassIVE reanalysis RMSV000000091.3 (1% FDR-filtered)
- Human draft proteome dataset by Kim et al. (PRIDE PXD000561) metadata for spectrum provenance

## Outputs

- Per-tool cluster size distribution table
- Per-tool fraction of clustered spectra and fraction of incorrectly clustered spectra
- Per-tool cluster completeness (1 - conditional entropy of clusters given peptide assignments)
- Per-tool data-volume reduction (non-noise clusters / total spectra)
- cluster_comparison_table.csv combining all per-tool metrics

## How to apply

Download the deposited clustering results for all five tools from Zenodo (doi:10.5281/zenodo.4721496) and the peptide labels from MassIVE reanalysis RMSV000000091.3, which contains PSMs filtered at 1% FDR (10,487,235 spectrum identifications) searched against the UniProtKB/Swiss-Prot human reference proteome. Restrict the comparison to MS/MS spectra with precursor charges 2 and 3, require valid clusters to contain at least two spectra, and relabel singleton clusters as noise for consistent cross-tool comparison. Select each tool's clustering result at its approximately 1% incorrectly-clustered configuration: falcon at cosine distance threshold 0.05, MaRaCluster threshold -30.0, MS-Cluster mixture probability 0.00001, msCRUSH similarity 0.8, and spectra-cluster threshold 0.99999. Compute, per tool, the fraction of clustered spectra, the fraction of incorrectly clustered spectra, the cluster size distribution, completeness as one minus the conditional entropy of the cluster distribution given peptide assignments (normalized by maximum possible entropy reduction), and data-volume reduction as the number of non-noise clusters divided by the total number of spectra. Assemble the per-tool metrics into a single table (cluster_comparison_table.csv) using Python 3.8 with pandas, then validate that falcon reaches the ~1% incorrect rate with clusters of 1000–10000 spectra and greater data-volume reduction than MaRaCluster and spectra-cluster, which predominantly produce clusters of at most 100 spectra.

## Related tools

- **falcon** (Deposited clustering result whose cluster size distribution and ~1% incorrect configuration (cosine distance 0.05) is reanalyzed and benchmarked against the comparison tools) — https://github.com/bittremieux/falcon
- **MS-Cluster** (Comparison tool whose deposited clustering output at mixture probability 0.00001 is reanalyzed for cluster size distribution and purity)
- **msCRUSH** (Comparison tool whose deposited clustering output at similarity threshold 0.8 is reanalyzed)
- **MaRaCluster** (Comparison tool whose deposited clustering output at threshold -30.0 is reanalyzed; known to produce small clusters of at most 100 spectra at ~1% incorrect rate)
- **spectra-cluster** (Comparison tool whose deposited clustering output at threshold 0.99999 is reanalyzed)
- **falcon_notebooks** (Companion repository containing the code used to compare the clustering tools and generate the comparison figures) — https://github.com/bittremieux/falcon_notebooks

## Examples

```
import pandas as pd; clusters = pd.read_csv('falcon.csv', names=['spectrum', 'cluster']); sizes = clusters.groupby('cluster').size(); sizes[sizes >= 2].to_csv('cluster_size_distribution.csv')
```

## Evaluation signals

- Each tool's selected configuration yields approximately 1% incorrectly clustered spectra (verified against PSM peptide labels)
- At the ~1% incorrect rate, falcon and msCRUSH produce several large clusters of 1000–10000 spectra, whereas MaRaCluster and spectra-cluster predominantly produce small clusters of at most 100 spectra
- Valid clusters are required to contain at least two spectra; singleton clusters are relabeled as noise before computing fractions
- Data-volume reduction equals the number of non-noise clusters divided by the total number of spectra and is greatest for falcon among the compared tools
- Output table cluster_comparison_table.csv contains one row per tool with clustered fraction, incorrect fraction, completeness, size distribution, and data-volume reduction columns

## Limitations

- Performance of the clustering tools on downstream tasks (e.g., peptide/protein identification from consensus spectra) is not directly evaluated; the comparison is indirect
- The strategy used to form cluster representatives (e.g., medoid selection vs. consensus spectrum compilation) affects downstream utility but was not evaluated in this reanalysis
- PSM identifications cover only 10,487,235 of the ~25 million spectra (1% FDR), so incorrect-clustering fractions are estimated against a partial labeling; analysis is restricted to precursor charges 2 and 3 of the human draft proteome dataset

## Evidence

- [results] Tool-dependent cluster size distributions: [evidence span withheld — anchor status: not_found]
- [discussion] Downstream performance not directly evaluated: [evidence span withheld — anchor status: not_found]
