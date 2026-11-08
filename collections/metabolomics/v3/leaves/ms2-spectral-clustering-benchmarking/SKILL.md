---
name: ms2-spectral-clustering-benchmarking
description: Use when repeated MS/MS acquisitions in DDA metabolomics data must be
  de-duplicated by clustering, and the clustering quality (purity vs completeness
  N10) of Falcon, MaRaCluster, or MS-Cluster must be compared without metabolite ground
  truth.
when_to_use_negative:
- The dataset is proteomics with FDR-controlled database search identifications available
  — use the DB-Search purity reference approach directly instead of MS-RT
- Input is already a validated/annotated set of chemical standards whose purity can
  be assessed by manual m/z-matched annotation without benchmarking clustering tools
- Only a single clustering tool is needed for production clustering (no tool comparison),
  and clustering quality need not be measured
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MS-Cluster (msCluster)
  role: Clustering tool benchmarked; run via GNPS 2.0 Classical Networking Workflow
    and evaluated with MS-RT
- name: Falcon
  role: Clustering tool benchmarked; results summarized into the common schema with
    src/utility/summarize_results
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool benchmarked; output merged with retention times via src/utility/maracluster_processing
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: msconvert (ProteoWizard)
  role: Converts vendor RAW files to mzML before clustering
- name: ThermoRawFileParser
  role: Cross-platform alternative for converting Thermo RAW files to mzML (-f=1)
    before clustering
  repo: https://github.com/compomics/ThermoRawFileParser
- name: Clustering_benchmark_MS_RT.py
  role: Python script from the benchmarking repo that computes purity and N10 with
    the MS-RT method at a given -tol window
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:e4ea1f16475cad1b2092d1f22a56d0293591eb0a684fe25c7094e47ef8f45f88
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Clustering_benchmark_MS_RT.py
  - Falcon
  - MS-Cluster (msCluster)
  - MS-RT
  - MaRaCluster
  - ProteoWizard
  - Python
  - ThermoRawFileParser
  - msconvert
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# ms2-spectral-clustering-benchmarking

## Summary

Benchmark MS/MS spectrum clustering tools (MS-Cluster, MaRaCluster, Falcon) for metabolomics data using the MS-RT (MS1 retention time) method, scoring purity and completeness (N10) without needing FDR-controlled identifications.

## When to use

Use when repeated MS/MS acquisitions in DDA metabolomics data must be de-duplicated by clustering, and the clustering quality (purity vs completeness N10) of Falcon, MaRaCluster, or MS-Cluster must be compared without metabolite ground truth.

## When NOT to use

- The dataset is proteomics with FDR-controlled database search identifications available — use the DB-Search purity reference approach directly instead of MS-RT
- Input is already a validated/annotated set of chemical standards whose purity can be assessed by manual m/z-matched annotation without benchmarking clustering tools
- Only a single clustering tool is needed for production clustering (no tool comparison), and clustering quality need not be measured

## Inputs

- Vendor RAW mass spectrometry files (e.g. Thermo .raw) downloaded from ProteomeXchange
- mzML (or MGF) files converted from RAW via msconvert or ThermoRawFileParser
- GNPS 2.0 Classical Networking Workflow output nf_output/clustering/clusterinfo.tsv for MS-Cluster
- Falcon clustering output (falcon.csv)
- MaRaCluster clustering output plus retention times from GNPS 2.0 PerScanSummarizer (nf_output/results.csv)
- Total MS/MS spectrum count (-t) for purity normalization

## Outputs

- Unified cluster info tables with columns filename, scan, mass (precursor_mz), rt_time, cluster
- Per-cluster purity and N10 values from src/Clustering_benchmark_MS_RT.py
- Aggregated metrics table msrt_benchmark_results.csv (purity and N10 versus dataset size)
- Benchmark figure (e.g. figure8_msrt_benchmark.png) comparing clustering tools
- Per-cluster MS-RT vs DB-Search purity comparison table for proteomics validation

## How to apply

Convert vendor RAW files to mzML with msconvert (ProteoWizard) or ThermoRawFileParser (-f=1), then cluster the spectra with each tool: MS-Cluster via the GNPS 2.0 Classical Networking Workflow (retrieve nf_output/clustering/clusterinfo.tsv), Falcon per its GitHub instructions, and MaRaCluster per its GitHub instructions. Post-process all outputs into a common schema with columns filename (#Filename), scan (#Scan), mass (#ParentMass), rt_time (#RetTime), cluster (#ClusterIdx); summarize Falcon with src/utility/summarize_results and merge MaRaCluster output with retention times from the GNPS 2.0 PerScanSummarizer workflow (nf_output/results.csv) via src/utility/maracluster_processing. Run src/Clustering_benchmark_MS_RT.py with -tol 0.1 (the mass-to-retention-time window tolerance) across increasing subsampled dataset sizes, then aggregate purity and N10 values versus dataset size. The rationale: metabolomics lacks FDR-controlled identifications for external ground truth, so MS-RT exploits the fact that spectra of the same molecule share precursor mass and retention time, and the method is validated against DB-Search purity in proteomics (R2 = 0.783, y = x).

## Related tools

- **MS-Cluster (msCluster)** (Clustering tool benchmarked; run via GNPS 2.0 Classical Networking Workflow and evaluated with MS-RT)
- **Falcon** (Clustering tool benchmarked; results summarized into the common schema with src/utility/summarize_results) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool benchmarked; output merged with retention times via src/utility/maracluster_processing) — https://github.com/statisticalbiotechnology/maracluster
- **msconvert (ProteoWizard)** (Converts vendor RAW files to mzML before clustering)
- **ThermoRawFileParser** (Cross-platform alternative for converting Thermo RAW files to mzML (-f=1) before clustering) — https://github.com/compomics/ThermoRawFileParser
- **Clustering_benchmark_MS_RT.py** (Python script from the benchmarking repo that computes purity and N10 with the MS-RT method at a given -tol window) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/mscluster/mscluster_cluster_info.tsv -t 390746 -methods mscluster -tol 0.1
```

## Evaluation signals

- On the Corteva actinobacteria dataset, Falcon reaches N10 of about 1800 with purity ~0.98, MS-Cluster N10 1100–1200 with purity 0.96–0.98, and MaRaCluster shows the poorest N10/purity trade-off (Figure 8 trends)
- MS-RT vs DB-Search purity on PXD023047 clustered with MS-Cluster agrees for 85.4% of the 11,152 clusters (9,524) with R2 = 0.783 along y = x (Figure 2.a)
- On the MSV000080554 standard-library dataset, MS-RT consistency with manual chemical standard annotations exceeds 95% (2,309 of 2,414 clusters within Y = X ± 0.1); average purity 0.919 MS-RT vs 0.928 manual (SI Figure 14)
- All post-processed cluster files contain the required columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx before benchmarking
- The -tol parameter (e.g. 0.1) sets the mass-to-retention-time window used by MS-RT and must be consistent across all compared tools

## Limitations

- Metabolomics lacks FDR-controlled identifications needed for external ground-truth evaluation of MS/MS clustering, so MS-RT is a proxy rather than a direct identification-based metric
- MaRaCluster clustering output does not carry retention times natively and must be merged with RTs from the GNPS 2.0 PerScanSummarizer workflow
- Falcon requires Python 3.8+ and runs on Linux and OSX only; ThermoRawFileParser versions before 1.5.0 require Mono on Linux/Mac
- Benchmark conclusions depend on subsampled dataset size; N10 and purity vary with sampling rate (e.g. SI Figures 11 and 13 for qToF and Orbitrap datasets)

## Evidence
