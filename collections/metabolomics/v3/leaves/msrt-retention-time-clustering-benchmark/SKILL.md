---
name: msrt-retention-time-clustering-benchmark
description: Use when clustering MS/MS metabolomics spectra and needing tool/preset
  comparison (purity vs completeness) without FDR-controlled identifications, on Orbitrap
  or qToF mzML/MGF data.
when_to_use_negative:
- Evaluating clustering on proteomics data where database-search (DB-Search) identifications
  with FDR control provide an established ground truth — the article validates MS-RT
  against DB-Search but DB-Search is the preferred method there
- You only have a single clustering run and no per-tool/per-parameter comparison is
  intended — the skill's N10/purity metrics only add value in comparative or completeness-sensitive
  evaluation
tools:
- name: MS-Cluster
  role: Clustering tool benchmarked; run via GNPS 2.0 Classical Networking Workflow,
    whose nf_output/clustering/clusterinfo.tsv already carries the MS-RT-required
    columns
- name: Falcon
  role: Clustering tool benchmarked; its raw output is reformatted by src/utility/summarize_results
    before MS-RT scoring
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool benchmarked; needs RT merging from PerScanSummarizer results
    via src/utility/maracluster_processing before MS-RT scoring
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: MS-RT
  role: The evaluation method itself, implemented as src/Clustering_benchmark_MS_RT.py
    scoring purity and N10 within a -tol 0.1 mass-to-RT window
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: msconvert
  role: ProteoWizard converter producing mzML inputs for clustering
- name: ThermoRawFileParser
  role: Cross-platform Thermo RAW to mzML/MGF converter (-f=1 mzML, -f=0 MGF)
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:b00f94163b5c105b475b25702451300b57fa9d238d7588324dc2c1c8f2feae5f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Falcon
  - MS-Cluster
  - MS-RT
  - MaRaCluster
  - ProteoWizard
  - Python
  - ThermoRawFileParser
  - msconvert
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# msrt-retention-time-clustering-benchmark

## Summary

Applies the MS-RT method — evaluating MS/MS clustering quality using precursor mass-to-retention-time consistency (purity and N10) instead of database-search FDR — to benchmark clustering tools (Falcon, MaRaCluster, MS-Cluster) on metabolomics mzML/MGF datasets.

## When to use

Use when clustering MS/MS metabolomics spectra and needing tool/preset comparison (purity vs completeness) without FDR-controlled identifications, on Orbitrap or qToF mzML/MGF data.

## When NOT to use

- Evaluating clustering on proteomics data where database-search (DB-Search) identifications with FDR control provide an established ground truth — the article validates MS-RT against DB-Search but DB-Search is the preferred method there
- You only have a single clustering run and no per-tool/per-parameter comparison is intended — the skill's N10/purity metrics only add value in comparative or completeness-sensitive evaluation

## Inputs

- Vendor RAW metabolomics files (e.g. Thermo .raw) from ProteomeXchange datasets such as MSV000093033 (Orbitrap algae) and MSV000081981 (qToF human stool)
- Converted mzML or MGF peak files
- Per-tool clustering outputs (Falcon falcon.csv, MaRaCluster clustering output, GNPS mscluster clusterinfo.tsv)
- Per-scan retention-time table from GNPS 2.0 PerScanSummarizer (nf_output/results.csv) for MaRaCluster merging
- Total MS/MS spectrum count for the dataset (e.g. -t 109333)

## Outputs

- Standardized per-cluster table with filename, scan, mass (precursor_mz), rt_time, and cluster columns for each tool
- Per-tool MS-RT purity and N10 scores (e.g. msrt_benchmark_results.csv) at multiple dataset sizes/parameters
- Benchmark plots of N10 and purity vs dataset size (e.g. figure8_msrt_benchmark.png)

## How to apply

Convert vendor RAW files to mzML/MGF with msconvert (ProteoWizard) or ThermoRawFileParser (-f=1 for mzML, -f=0 for MGF). Cluster spectra with each tool: Falcon (github.com/bittremieux/falcon), MaRaCluster (github.com/statisticalbiotechnology/maracluster), and MS-Cluster via the GNPS 2.0 Classical Networking Workflow (retrieve nf_output/clustering/clusterinfo.tsv with columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx). Post-process each output into the common schema (filename, scan, mass, rt_time, cluster): summarize Falcon results with src/utility/summarize_results and merge MaRaCluster clusters with per-scan retention times from the GNPS 2.0 PerScanSummarizer workflow (nf_output/results.csv) using src/utility/maracluster_processing. Then run src/Clustering_benchmark_MS_RT.py per tool with -c pointing to that tool's cluster file, -t set to the dataset's total MS/MS count (e.g. 109333 for the reference dataset), and -tol 0.1 as the mass-to-retention-time window tolerance; subsample datasets across increasing sizes to plot N10 and purity versus dataset size.

## Related tools

- **MS-Cluster** (Clustering tool benchmarked; run via GNPS 2.0 Classical Networking Workflow, whose nf_output/clustering/clusterinfo.tsv already carries the MS-RT-required columns)
- **Falcon** (Clustering tool benchmarked; its raw output is reformatted by src/utility/summarize_results before MS-RT scoring) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool benchmarked; needs RT merging from PerScanSummarizer results via src/utility/maracluster_processing before MS-RT scoring) — https://github.com/statisticalbiotechnology/maracluster
- **MS-RT** (The evaluation method itself, implemented as src/Clustering_benchmark_MS_RT.py scoring purity and N10 within a -tol 0.1 mass-to-RT window) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **msconvert** (ProteoWizard converter producing mzML inputs for clustering)
- **ThermoRawFileParser** (Cross-platform Thermo RAW to mzML/MGF converter (-f=1 mzML, -f=0 MGF)) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c data/mscluster/mscluster_cluster_info.tsv -t 109333 -tol 0.1
```

## Evaluation signals

- The MS-RT benchmarking script completes for every tool/dataset combination without schema errors
- Result tables carry exactly the required columns: filename, scan, mass (precursor_mz), rt_time, cluster (or #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx for GNPS output)
- On the Orbitrap dataset MSV000093033, MaRaCluster shows highest purity (~0.99) with lowest N10, while Falcon attains highest N10 (300) with purity ~0.93, matching Figures 6–7 orderings
- On the Corteva actinobacteria dataset, Falcon reaches N10 ≈ 1800 with purity ~0.98 and MS-Cluster N10 1100–1200 with purity 0.96–0.98, matching Figure 8
- Clustered-fraction expectations hold on qToF: Falcon clusters 70–80% of spectra while MaRaCluster clusters under 5–18%

## Limitations

- Metabolomics lacks FDR-controlled identifications, so MS-RT substitutes a retention-time proxy for database-search ground truth and was validated against DB-Search only on proteomics datasets (PXD021518, PXD02047)
- MS-RT scores depend on clustering tool parameters — Falcon needed N10 settings of 3000–5000 on the qToF dataset vs 300 on Orbitrap — so parameter grids must be rerun per dataset/instrument
- N10 and purity trade off: high purity presets (MaRaCluster up to purity 1 on qToF) can cluster under 5–18% of spectra, so a single metric is misleading without completeness

## Evidence
