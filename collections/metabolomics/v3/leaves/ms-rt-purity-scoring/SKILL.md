---
name: ms-rt-purity-scoring
description: Use when clustering tandem mass spectra (e.g. from PXD023047 with MS-Cluster,
  Falcon, or MaRaCluster) and you need a per-cluster purity score that does not depend
  on FDR-controlled database-search identifications, or when validating an MS-RT purity
  score against a DB-Search reference.
when_to_use_negative:
- When FDR-controlled database-search identifications are already available for the
  dataset (e.g. proteomics PXD021518) and the DB-Search purity method is directly
  applicable, since DB-Search is the established reference approach.
- When the clustering output lacks the required clusterinfo.tsv columns ('#Filename',
  '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx') or the underlying retention time
  data, as the MS-RT scoring cannot be computed.
- When you need molecule-level structural identifications rather than a clustering-quality
  metric; MS-RT evaluates cluster purity, not compound annotation.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS-Cluster
  role: Produces the clustered spectra (clusterinfo.tsv) that MS-RT purity is computed
    on
  repo: https://gnps2.org/workflowinput?workflowname=classical_networking_workflow
- name: MS-RT
  role: The purity-scoring method itself, implemented in src/Clustering_benchmark_MS_RT.py
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: DB-Search
  role: Reference purity method (MS-GF+/FDR) used to validate MS-RT on proteomics
    datasets
- name: msconvert
  role: Converts vendor RAW files (e.g. PXD023047) to mzML for clustering and MS-RT
    scoring
  repo: http://proteowizard.sourceforge.net/download.html
- name: ThermoRawFileParser
  role: Alternative cross-platform converter of Thermo RAW to mzML/MGF
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:fcb271528a6f46d5f6b460353608de419375cdf4c34d2c1cf669df7e7abe8f83
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DB-Search
  - MS-Cluster
  - MS-RT
  - ProteoWizard
  - Python
  - ThermoRawFileParser
  - msconvert
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# ms-rt-purity-scoring

## Summary

Evaluate per-cluster purity of MS/MS spectral clusters using the MS-RT method, which leverages MS1 precursor mass and retention time consistency within clusters instead of database-search identifications, and validate it against DB-Search-based purity on proteomics datasets such as PXD023047.

## When to use

Use when clustering tandem mass spectra (e.g. from PXD023047 with MS-Cluster, Falcon, or MaRaCluster) and you need a per-cluster purity score that does not depend on FDR-controlled database-search identifications, or when validating an MS-RT purity score against a DB-Search reference.

## When NOT to use

- When FDR-controlled database-search identifications are already available for the dataset (e.g. proteomics PXD021518) and the DB-Search purity method is directly applicable, since DB-Search is the established reference approach.
- When the clustering output lacks the required clusterinfo.tsv columns ('#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx') or the underlying retention time data, as the MS-RT scoring cannot be computed.
- When you need molecule-level structural identifications rather than a clustering-quality metric; MS-RT evaluates cluster purity, not compound annotation.

## Inputs

- MS-Cluster/Falcon/MaRaCluster clustering output (e.g. nf_output/clustering/clusterinfo.tsv from GNPS 2.0 Classical Networking Workflow) with columns '#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx'
- Proteomics dataset raw files (e.g. PXD023047 from ProteomeXchange) converted to mzML via msconvert or ThermoRawFileParser
- Optional: DB-Search (MS-GF+/FDR) identification-based purity table for the same clusters, used as a validation reference

## Outputs

- Per-cluster MS-RT purity score for every cluster in the dataset
- Per-cluster DB-Search purity score (when validating against proteomics reference)
- Agreement statistics comparing MS-RT vs DB-Search purity: percentage of clusters within 10% purity difference and R2 at y=x (e.g. 85.4% agreement and R2 = 0.783 for PXD023047, Figure 2.a)

## How to apply

For each spectral cluster produced by MS-Cluster, Falcon, or MaRaCluster, compute purity as the proportion of cluster members whose MS1 mass and retention time are consistent, using the script src/Clustering_benchmark_MS_RT.py with the clustered spectra's clusterinfo.tsv (columns '#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx') as input. The MS-RT method replaces the database-search-based purity used in proteomics, which is impractical for metabolomics because FDR-controlled identifications are unavailable; it instead scores clusters on retention-time and mass agreement. To validate, compare MS-RT purity against DB-Search purity (MS-GF+/FDR-controlled identifications) on a proteomics dataset such as PXD023047, where roughly 85.4% of the 11,152 MS-Cluster clusters agree within 10% purity difference and an R2 of 0.783 is expected along the y=x line (Figure 2.a). Run the benchmark with parameters like -c <mscluster_cluster_info.tsv> -t 109333 -methods mscluster -tol 0.1, and inspect the resulting per-cluster purity comparison table. Outputs should include a per-cluster MS-RT purity value, an optional DB-Search purity value for validation, and agreement statistics (percent within 10% difference, R2).

## Related tools

- **MS-Cluster** (Produces the clustered spectra (clusterinfo.tsv) that MS-RT purity is computed on) — https://gnps2.org/workflowinput?workflowname=classical_networking_workflow
- **MS-RT** (The purity-scoring method itself, implemented in src/Clustering_benchmark_MS_RT.py) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **DB-Search** (Reference purity method (MS-GF+/FDR) used to validate MS-RT on proteomics datasets)
- **msconvert** (Converts vendor RAW files (e.g. PXD023047) to mzML for clustering and MS-RT scoring) — http://proteowizard.sourceforge.net/download.html
- **ThermoRawFileParser** (Alternative cross-platform converter of Thermo RAW to mzML/MGF) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/mscluster/mscluster_cluster_info.tsv -t 109333 -methods mscluster -tol 0.1
```

## Evaluation signals

- On PXD023047 clustered with MS-Cluster, the fraction of clusters whose MS-RT and DB-Search purity differ by ≤10% should be approximately 85.4% (9,524 of 11,152).
- The R2 of the per-cluster MS-RT vs DB-Search purity comparison at y=x should be approximately 0.783 (Figure 2.a).
- The input clusterinfo.tsv must contain the columns '#Filename', '#Scan', '#ParentMass', '#RetTime', and '#ClusterIdx' before running the scoring script.
- The number of clusters analyzed should match the clustering tool's reported cluster count (e.g. 11,152 MS-Cluster clusters for PXD023047).
- Python environment must be >= 3.8 (e.g. the 'Clustering_benchmarking' virtualenv with requirement.txt installed).

## Limitations

- MS-RT is a proxy metric: it assumes spectra of the same molecule share consistent MS1 mass and retention time, which may break down for co-eluting isomers or retention-time drift between runs.
- Validation values (85.4% agreement, R2 = 0.783) are dataset-specific to PXD023047 clustered with MS-Cluster and may vary with other datasets or clustering tools.
- The method requires retention time and parent mass to be extracted and joined correctly (e.g. via GNPS 2.0 PerScanSummarizer Workflow, nf_output/results.csv) before scoring.
- In metabolomics, FDR-controlled identifications are not currently practical, so MS-RT cannot be cross-validated against a true identification-based ground truth there.

## Evidence
