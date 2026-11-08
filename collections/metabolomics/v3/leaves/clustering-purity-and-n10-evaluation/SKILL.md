---
name: clustering-purity-and-n10-evaluation
description: Use when you have run one or more MS/MS clustering tools (MS-Cluster,
  Falcon, MaRaCluster) on a metabolomics or proteomics dataset and need to quantify
  clustering quality in terms of purity and N10 completeness without relying on FDR-controlled
  identifications.
when_to_use_negative:
- You have no retention-time information for the spectra (e.g. clustering outputs
  lacking rt_time column) and no proteomics identifications for DB-Search cross-validation
  — purity and N10 cannot be computed.
- You want an absolute ground-truth cluster assignment rather than a relative quality
  score; MS-RT purity is a retention-time consistency proxy, not a verified molecular
  identity.
- You are clustering spectra from a single acquisition with essentially no redundant
  repeated acquisitions of the same molecule, where deduplication metrics have little
  meaning.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-RT
  role: Benchmark script (src/Clustering_benchmark_MS_RT.py) that computes purity
    and N10 from a tool's cluster_info file using retention-time consistency at tolerance
    0.1.
- name: MS-Cluster
  role: Clustering tool evaluated; output retrieved as nf_output/clustering/clusterinfo.tsv
    from the GNPS 2.0 Classical Networking Workflow.
- name: Falcon
  role: Clustering tool evaluated; results summarized with src/utility/summarize_results
    before benchmarking.
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool evaluated; results merged with retention time via src/utility/maracluster_processing
    before benchmarking.
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: DB-Search
  role: 'Proteomics cross-validation route: compare MS-RT purity/N10 against database-search
    identifications on datasets such as PXD021518.'
- name: Metabolomics_Clustering_Benchmark
  role: Repository hosting the MS-RT benchmark scripts, per-tool post-processing utilities,
    and dataset download/conversion instructions.
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:57e86879e18012651070a2dd9f557eeaf2f5fce75a3217eff7983c77dc167e32
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - DB-Search
  - Falcon
  - MS-Cluster
  - MS-RT
  - MaRaCluster
  - Metabolomics_Clustering_Benchmark
  - ProteoWizard
  - Python
  - msconvert
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 24
---

# clustering-purity-and-n10-evaluation

## Summary

Benchmarks MS/MS clustering tools (MS-Cluster, Falcon, MaRaCluster) using the MS-RT method, scoring cluster purity and N10 completeness against the validated ground truth of retention time consistency and database-search identifications.

## When to use

Use when you have run one or more MS/MS clustering tools (MS-Cluster, Falcon, MaRaCluster) on a metabolomics or proteomics dataset and need to quantify clustering quality in terms of purity and N10 completeness without relying on FDR-controlled identifications.

## When NOT to use

- You have no retention-time information for the spectra (e.g. clustering outputs lacking rt_time column) and no proteomics identifications for DB-Search cross-validation — purity and N10 cannot be computed.
- You want an absolute ground-truth cluster assignment rather than a relative quality score; MS-RT purity is a retention-time consistency proxy, not a verified molecular identity.
- You are clustering spectra from a single acquisition with essentially no redundant repeated acquisitions of the same molecule, where deduplication metrics have little meaning.

## Inputs

- cluster_info file from MS-Cluster (GNPS 2.0 Classical Networking Workflow nf_output/clustering/clusterinfo.tsv)
- formatted Falcon output (post src/utility/summarize_results, e.g. falcon.csv summary)
- formatted MaRaCluster output (post src/utility/maracluster_processing with merged retention time)
- per-scan retention time file from GNPS 2.0 PerScanSummarizer Workflow (nf_output/results.csv)
- mzML or MGF spectra files converted from vendor raw data
- total MS/MS spectrum count for the dataset

## Outputs

- per-tool purity and N10 benchmark results from Clustering_benchmark_MS_RT.py
- sampling-rate benchmark table (e.g. sampling_benchmark_msv000081981.csv) of purity and N10 vs fraction
- purity and N10 curves for comparison across clustering tools and dataset sizes

## How to apply

Prepare each tool's clustering output in a unified format with columns 'filename' (#Filename), 'scan' (#Scan), 'mass' (#ParentMass), 'rt_time' (#RetTime), and 'cluster' (#ClusterIdx), using the benchmark repo's per-tool post-processing scripts (e.g. src/utility/summarize_results for Falcon, src/utility/maracluster_processing for MaRaCluster, and GNPS 2.0 Classical Networking Workflow nf_output/clustering/clusterinfo.tsv for MS-Cluster). Run the benchmark script src/Clustering_benchmark_MS_RT.py with a cluster_info file, the total number of MS/MS spectra (-t), a tool method flag (-methods mscluster|falcon|maracluster), and a retention-time tolerance of 0.1, which evaluates whether spectra grouped into the same cluster have MS1 retention times consistent within that tolerance (purity) and how the top clusters cover identifications (N10 completeness). For proteomics datasets where database-search identifications exist, cross-validate MS-RT purity and N10 against DB-Search-based evaluation using src/clustering_eval_compare_DB, as the paper validates MS-RT against established DB-search FDR-based metrics on datasets such as PXD021518. Repeat the benchmark across sampling fractions (10%-100% of the file set, e.g. 2,307 mzML files in MSV000081981) to build purity and N10 curves that reveal completeness differences at equal purity, such as Falcon reaching N10 ~2500 vs MS-Cluster ~1000 at 70% sampling while both sit near purity 0.9986.

## Related tools

- **MS-RT** (Benchmark script (src/Clustering_benchmark_MS_RT.py) that computes purity and N10 from a tool's cluster_info file using retention-time consistency at tolerance 0.1.)
- **MS-Cluster** (Clustering tool evaluated; output retrieved as nf_output/clustering/clusterinfo.tsv from the GNPS 2.0 Classical Networking Workflow.)
- **Falcon** (Clustering tool evaluated; results summarized with src/utility/summarize_results before benchmarking.) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool evaluated; results merged with retention time via src/utility/maracluster_processing before benchmarking.) — https://github.com/statisticalbiotechnology/maracluster
- **DB-Search** (Proteomics cross-validation route: compare MS-RT purity/N10 against database-search identifications on datasets such as PXD021518.)
- **Metabolomics_Clustering_Benchmark** (Repository hosting the MS-RT benchmark scripts, per-tool post-processing utilities, and dataset download/conversion instructions.) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/mscluster/mscluster_cluster_info.tsv -t 2307 -methods mscluster -tol 0.1
```

## Evaluation signals

- Benchmark script src/Clustering_benchmark_MS_RT.py runs without errors and accepts the tool's method flag (-methods mscluster|falcon|maracluster) and tolerance 0.1.
- Cluster_info input contains the expected columns 'filename' (#Filename), 'scan' (#Scan), 'mass' (#ParentMass), 'rt_time' (#RetTime), 'cluster' (#ClusterIdx).
- At 70% sampling of MSV000081981 both MS-Cluster and Falcon show average purity near 0.9986, with Falcon's N10 (~2500) exceeding MS-Cluster's (~1000).
- At 10% sampling both tools yield purity around 0.992-0.998 with N10 in the low hundreds, consistent with SI Figure 11 trends.
- For proteomics datasets, MS-RT purity and N10 agree with DB-Search-based evaluation on datasets such as PXD021518, as shown in the SI Figure 4 consistency heatmap.

## Limitations

- MS-RT purity relies on retention-time consistency as a proxy for correct clustering; it does not verify chemical identity and cannot substitute for database-search FDR-controlled evaluation in metabolomics.
- The method is validated against DB-Search evaluation only on proteomics datasets (e.g. PXD021518); its behavior on purely metabolomics datasets is inferred rather than externally verified.
- Results depend on the chosen retention-time tolerance (0.1 in the paper's benchmark); different tolerances will change purity values.
- Sampling-rate curves require subsampling the full file set (e.g. 10%-100% of 2,307 mzML files in MSV000081981), so large datasets need substantial compute for each fraction.

## Evidence

- [methods] MS-RT benchmark invocation: [evidence span withheld — anchor status: not_found]
- [abstract] MS-RT method rationale: "this study introduces the MS1-retention time (MS-RT) method to assess MS/MS clustering performance in metabolomics data sets"
- [methods] Retention time extraction workflow: [evidence span withheld — anchor status: not_found]
