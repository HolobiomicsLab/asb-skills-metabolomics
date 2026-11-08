---
name: db-search-fdr-cluster-validation
description: Use when MS/MS spectra have been clustered (e.g. with MS-Cluster, MaRaCluster,
  or Falcon) and you need an independent, identification-based purity ground truth
  to validate a retention-time-based clustering metric such as MS-RT.
when_to_use_negative:
- No FDR-controlled database-search tool exists for the data domain (e.g. metabolomics),
  so the DB-Search reference purity cannot be computed — this is precisely the setting
  MS-RT was introduced to address
- The input spectra have not been clustered; this skill evaluates cluster purity,
  not raw spectral quality
- 'The dataset lacks retention time information (e.g. missing #RetTime or PerScanSummarizer
  output), since the comparison requires the MS-RT purity to be computable in parallel'
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-Cluster
  role: Produces the spectral clusters whose purity is validated by comparing MS-RT
    and DB-Search estimates
- name: MS-GF+
  role: Provides the FDR-controlled database-search identifications used as the reference
    ground-truth for cluster purity
- name: MS-RT
  role: Computes the retention-time-based per-cluster purity that is compared against
    the DB-Search purity
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: msconvert
  role: Converts vendor raw files to mzML/MGF before clustering and searching
  repo: http://proteowizard.sourceforge.net/download.html
- name: ThermoRawFileParser
  role: Cross-platform alternative for converting Thermo RAW files to mzML or MGF
  repo: https://github.com/compomics/ThermoRawFileParser
- name: Python
  role: Runs the comparison scripts (Clustering_benchmark_MS_RT.py, clustering_eval_compare_DB)
    in the Clustering_benchmarking virtual environment
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:55a09a3ff11140c728d0e963bd87ac3bf72df0fdd9f0dc0b42c8514955ac0a2d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
  tools:
  - DB-Search
  - MS-Cluster
  - MS-GF+
  - MS-RT
  - ProteoWizard
  - Python
  - ThermoRawFileParser
  - msconvert
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# db-search-fdr-cluster-validation

## Summary

Validates the purity of MS/MS spectral clusters by comparing the MS-RT estimate against a database-search (MS-GF+ with FDR control) reference, the well-established evaluation approach in proteomics where FDR-controlled identifications exist.

## When to use

Use when MS/MS spectra have been clustered (e.g. with MS-Cluster, MaRaCluster, or Falcon) and you need an independent, identification-based purity ground truth to validate a retention-time-based clustering metric such as MS-RT.

## When NOT to use

- No FDR-controlled database-search tool exists for the data domain (e.g. metabolomics), so the DB-Search reference purity cannot be computed — this is precisely the setting MS-RT was introduced to address
- The input spectra have not been clustered; this skill evaluates cluster purity, not raw spectral quality
- The dataset lacks retention time information (e.g. missing #RetTime or PerScanSummarizer output), since the comparison requires the MS-RT purity to be computable in parallel

## Inputs

- MS/MS spectral cluster assignments with per-cluster membership, e.g. nf_output/clustering/clusterinfo.tsv with columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx
- Raw mass spectrometry data files (e.g. Thermo .raw) from ProteomeXchange datasets such as PXD023047, converted to mzML or MGF format
- Retention time values per spectrum (e.g. from GNPS 2.0 PerScanSummarizer Workflow, nf_output/results.csv)

## Outputs

- Per-cluster purity values from MS-GF+ FDR-controlled database search
- Per-cluster purity values from the MS-RT method
- A per-cluster purity comparison table (MS-RT vs DB-Search) with agreement statistics such as the fraction of clusters within a 10% purity difference and the R2 at y=x

## How to apply

Cluster the converted mzML spectra (e.g. via the GNPS 2.0 Classical Networking Workflow producing nf_output/clustering/clusterinfo.tsv with columns '#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx'). Run MS-GF+ database search on the clustered spectra, controlling identifications at a defined false discovery rate so that each cluster's purity reflects the fraction of spectra sharing a confident peptide identification. Separately compute per-cluster purity with the MS-RT method (python src/Clustering_benchmark_MS_RT.py with -c <clusterinfo.tsv> -t <retention-time file> -tol 0.1). For every cluster, compare the MS-RT-derived purity to the DB-Search-derived purity: on the PXD023047 benchmark (11,152 MS-Cluster clusters), ~85.4% of clusters agree within a 10% purity difference with R2 = 0.783 along the y=x line (Figure 2.a), which defines the expected level of concordance. Use this agreement to decide whether the retention-time metric is a valid proxy in settings — such as metabolomics — where FDR-controlled database-search identifications are not practical.

## Related tools

- **MS-Cluster** (Produces the spectral clusters whose purity is validated by comparing MS-RT and DB-Search estimates)
- **MS-GF+** (Provides the FDR-controlled database-search identifications used as the reference ground-truth for cluster purity)
- **MS-RT** (Computes the retention-time-based per-cluster purity that is compared against the DB-Search purity) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **msconvert** (Converts vendor raw files to mzML/MGF before clustering and searching) — http://proteowizard.sourceforge.net/download.html
- **ThermoRawFileParser** (Cross-platform alternative for converting Thermo RAW files to mzML or MGF) — https://github.com/compomics/ThermoRawFileParser
- **Python** (Runs the comparison scripts (Clustering_benchmark_MS_RT.py, clustering_eval_compare_DB) in the Clustering_benchmarking virtual environment) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/mscluster/mscluster_cluster_info.tsv -t 109333 -methods mscluster -tol 0.1
```

## Evaluation signals

- On PXD023047 clustered with MS-Cluster, approximately 85.4% of the 11,152 clusters show MS-RT vs DB-Search purity agreement within a 10% difference (9,524 clusters)
- The per-cluster purity comparison yields an R2 of 0.783 along the y=x line, matching Figure 2.a of the paper
- Every cluster in the comparison table has both an MS-RT purity and a DB-Search purity value; clusters missing one estimate are excluded from the concordance calculation
- DB-Search purity is computed only from identifications surviving the chosen FDR threshold, so purity never exceeds the fraction of spectra with confident peptide IDs
- The comparison table reproduces the same cluster count as the clusterinfo.tsv input (11,152 clusters for PXD023047)

## Limitations

- The DB-Search reference approach relies on database-search tools with false discovery rate control, which exist for proteomics but are not currently practical in metabolomics — so this validation cannot be replicated directly in metabolomics data
- The reported concordance (85.4% within 10%, R2 = 0.783) was established on the PXD023047 proteomics dataset with MS-Cluster clustering; other clustering tools (MaRaCluster, Falcon) or datasets may show different agreement levels
- Agreement is assessed at the cluster level on purity; it does not validate MS-RT's sensitivity to retention-time outliers or clusters with sparse identifications

## Evidence

- [results] PXD023047 MS-Cluster concordance result: [evidence span withheld — anchor status: not_found]
