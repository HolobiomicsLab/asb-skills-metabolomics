---
name: msms-clustering-parameter-sweep-evaluation
description: Use when benchmarking tandem-MS clustering tools on untargeted metabolomics
  spectra (Orbitrap or qTof mzML/MGF) and you need purity/N10 trade-off curves across
  tool parameter settings rather than a single clustering run.
when_to_use_negative:
- Evaluating clustering on proteomics data where database-search-based FDR-controlled
  evaluation (DB-Search) is available instead of the MS-RT proxy
- You need consensus spectra or downstream molecular networking output, not purity/N10
  performance scoring of clustering configurations
- 'The dataset has no usable MS1 retention-time information, since MS-RT scoring depends
  on #RetTime consistency within the -tol 0.1 window'
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Falcon
  role: Density-based MS/MS spectrum clustering tool swept across its parameter grid;
    outputs formatted with summarize_results before MS-RT scoring
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool whose batch results are merged with GNPS PerScanSummarizer
    retention times via maracluster_processing before MS-RT scoring
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: MS-Cluster
  role: Clustering tool run through the GNPS 2.0 Classical Networking workflow; nf_output/clustering/clusterinfo.tsv
    already carries the required benchmark columns
- name: msconvert
  role: ProteoWizard converter producing mzML/MGF inputs for the clustering tools
- name: ThermoRawFileParser
  role: Cross-platform alternative converter of Thermo RAW files to mzML (-f=1) or
    MGF (-f=0)
  repo: https://github.com/compomics/ThermoRawFileParser
- name: MS-RT
  role: Retention-time-based benchmark (Clustering_benchmark_MS_RT.py) scoring purity
    and N10 for each clustering configuration
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:1b9d802397ba60f01414efa4ba162104c314c43be167b7eab0769b6d356f8419
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 22
---

# msms-clustering-parameter-sweep-evaluation

## Summary

Run MS/MS clustering tools (Falcon, MaRaCluster, MS-Cluster) across their parameter grids on metabolomics mzML/MGF datasets and score each configuration's purity and completeness (N10) with the MS-RT benchmark, since metabolomics lacks FDR-controlled identifications for external evaluation.

## When to use

Use when benchmarking tandem-MS clustering tools on untargeted metabolomics spectra (Orbitrap or qTof mzML/MGF) and you need purity/N10 trade-off curves across tool parameter settings rather than a single clustering run.

## When NOT to use

- Evaluating clustering on proteomics data where database-search-based FDR-controlled evaluation (DB-Search) is available instead of the MS-RT proxy
- You need consensus spectra or downstream molecular networking output, not purity/N10 performance scoring of clustering configurations
- The dataset has no usable MS1 retention-time information, since MS-RT scoring depends on #RetTime consistency within the -tol 0.1 window

## Inputs

- Vendor raw mass spectrometry files or mzML/MGF conversions of ProteomeXchange datasets (e.g. MSV000093033 Orbitrap algae, MSV000081981 qTof human stool)
- Falcon clustering output (falcon.csv)
- MaRaCluster clustering output plus GNPS 2.0 PerScanSummarizer results.csv with per-scan retention times
- GNPS 2.0 Classical Networking clusterinfo.tsv (#Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx)
- Total MS/MS spectrum count per dataset (e.g. 109333)

## Outputs

- Post-processed per-tool cluster tables with filename, scan, mass, rt_time, and cluster columns
- Per-tool MS-RT benchmark scores (purity and N10) for each parameter-grid configuration
- Relative orderings of clustering tools reproducible as Figures 6-7 purity/completeness comparisons

## How to apply

Convert vendor raw files to mzML/MGF via msconvert (ProteoWizard) or ThermoRawFileParser, then run each clustering tool across its parameter grid: Falcon with published settings (e.g. N10 300, purity 0.93 on Orbitrap MSV000093033; N10 3000-5000 on qTof MSV000081981) formatted via src/utility/summarize_results into filename/scan/mass/rt_time/cluster columns; MaRaCluster with per-scan retention times fetched through the GNPS 2.0 PerScanSummarizer workflow and merged via maracluster_processing; MS-Cluster via GNPS 2.0 Classical Networking, taking nf_output/clustering/clusterinfo.tsv directly. Score each tool/dataset configuration by running src/Clustering_benchmark_MS_RT.py separately per tool with -c pointing to its post-processed cluster_info file, -t set to the dataset's total MS/MS count (e.g. 109333), and -tol 0.1 for the MS-RT window. Compare purity-N10 trade-offs, expecting orderings such as MaRaCluster's highest purity (0.99) but lowest N10 versus Falcon's highest N10 (300) with purity 0.93 on the Orbitrap dataset, and the qTof pattern where Falcon clusters 70-80% of spectra at moderate purity while MaRaCluster reaches purity up to 1 but clusters under 5-18% of spectra.

## Related tools

- **Falcon** (Density-based MS/MS spectrum clustering tool swept across its parameter grid; outputs formatted with summarize_results before MS-RT scoring) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool whose batch results are merged with GNPS PerScanSummarizer retention times via maracluster_processing before MS-RT scoring) — https://github.com/statisticalbiotechnology/maracluster
- **MS-Cluster** (Clustering tool run through the GNPS 2.0 Classical Networking workflow; nf_output/clustering/clusterinfo.tsv already carries the required benchmark columns)
- **msconvert** (ProteoWizard converter producing mzML/MGF inputs for the clustering tools)
- **ThermoRawFileParser** (Cross-platform alternative converter of Thermo RAW files to mzML (-f=1) or MGF (-f=0)) — https://github.com/compomics/ThermoRawFileParser
- **MS-RT** (Retention-time-based benchmark (Clustering_benchmark_MS_RT.py) scoring purity and N10 for each clustering configuration) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark

## Examples

```
python3 src/utility/summarize_results ./falcon.csv output_summary; python src/Clustering_benchmark_MS_RT.py -c output_summary -t 109333 -tol 0.1
```

## Evaluation signals

- Clustering_benchmark_MS_RT.py completes for every tool/dataset combination without missing-column errors
- Post-processed cluster tables expose exactly the required columns: filename/#Filename, scan/#Scan, mass/#ParentMass, rt_time/#RetTime, cluster/#ClusterIdx
- Per-tool MS-RT scores reproduce the Figures 6-7 relative orderings reported in the article under MS-RT evaluation
- Orbitrap reference configuration lands near MaRaCluster purity 0.99 with lowest N10, and Falcon purity 0.93 with N10 300
- qTof results show Falcon clustering 70-80% of spectra at moderate purity while MaRaCluster clusters under 5-18% of spectra even at purity up to 1

## Limitations

- MS-RT is a proxy metric: metabolomics lacks FDR-controlled identifications, so correctness is inferred from MS1 retention-time consistency rather than validated molecule IDs
- MaRaCluster's high purity comes at the cost of completeness (poor N10 on Orbitrap data; clusters under 5-18% of qTof spectra), so a single metric is misleading
- The MS-RT benchmark requires the dataset's total MS/MS count passed via -t and a fixed MS-RT window of -tol 0.1; mis-set values invalidate cross-tool comparisons

## Evidence

- [results] On the Orbitrap metabolomics dataset (MSV000093033), MaRaCluster achieves the highest purity (0.99) but the lowest N10 (poor completeness), while Falcon attains the highest N10 (300) with relatively high purity (0.93).: [evidence span withheld — anchor status: not_found]
- [other] Run src/Clustering_benchmark_MS_RT.py separately for each tool with -c pointing to that tool's post-processed cluster_info file, -t set to the dataset's total MS/MS count (e.g. 109333 for the reference dataset), and -tol 0.1: [evidence span withheld — anchor status: not_found]
