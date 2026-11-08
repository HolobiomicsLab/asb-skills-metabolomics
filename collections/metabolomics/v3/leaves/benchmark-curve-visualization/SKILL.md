---
name: benchmark-curve-visualization
description: Use when MS-RT purity/N10 values have been collected across increasing
  subsampled dataset sizes for multiple MS/MS clustering tools (Falcon, MaRaCluster,
  MS-Cluster) and a size-versus-performance comparison figure is needed.
when_to_use_negative:
- Evaluating clustering against database-search identifications (DB-Search/FDR-controlled
  peptide IDs) rather than MS-RT metrics — use the clustering_eval_compare_DB pipeline
  instead
- Only a single dataset size / single run is available, so a size-versus-performance
  curve cannot be constructed
- Inputs are raw vendor files not yet converted to mzML/MGF and not yet clustered
  — conversion and clustering must precede metric aggregation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Clustering_benchmark_MS_RT.py
  role: Computes the N10 and purity values per clustering tool and dataset size that
    are aggregated and plotted as the benchmark curves
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: MS-RT
  role: Mass-to-retention-time evaluation method whose purity and N10 metrics form
    the y-axes of the benchmark curves
  repo: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: Falcon
  role: Clustering tool whose results (summarized via src/utility/summarize_results)
    supply the top-performing curve (N10 ~1800, purity ~0.98)
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool whose RT-merged results (via src/utility/maracluster_processing)
    supply the poorest N10/purity curve
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: MS-Cluster
  role: msCluster results from the GNPS 2.0 Classical Networking Workflow (nf_output/clustering/clusterinfo.tsv)
    supply the intermediate N10 1100–1200 curve
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:44545a15244e8cfbef6c70462d027b45da0499bf6ad03d6ce76b979a593724c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
  tools:
  - Clustering_benchmark_MS_RT.py
  - Falcon
  - MS-Cluster
  - MS-RT
  - MaRaCluster
  - ProteoWizard
  - Python
  - msconvert
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# benchmark-curve-visualization

## Summary

Aggregates purity and N10 (completeness) metrics from MS-RT clustering-benchmark runs and plots them versus subsampled dataset size, reproducing the paper's Figure 8-style tool-comparison curves (e.g. Falcon N10 ~1800/purity ~0.98 vs MS-Cluster N10 1100–1200 vs MaRaCluster).

## When to use

Use when MS-RT purity/N10 values have been collected across increasing subsampled dataset sizes for multiple MS/MS clustering tools (Falcon, MaRaCluster, MS-Cluster) and a size-versus-performance comparison figure is needed.

## When NOT to use

- Evaluating clustering against database-search identifications (DB-Search/FDR-controlled peptide IDs) rather than MS-RT metrics — use the clustering_eval_compare_DB pipeline instead
- Only a single dataset size / single run is available, so a size-versus-performance curve cannot be constructed
- Inputs are raw vendor files not yet converted to mzML/MGF and not yet clustered — conversion and clustering must precede metric aggregation

## Inputs

- Per-run N10 and purity metrics from src/Clustering_benchmark_MS_RT.py across increasing subsampled dataset sizes
- Normalized cluster-info tables (filename, scan, mass, rt_time, cluster) for Falcon, MaRaCluster, and msCluster/GNPS 2.0 clusterinfo.tsv
- Subsampled mzML (or MGF) dataset files from the Corteva actinobacteria 570-sample metabolomics study

## Outputs

- msrt_benchmark_results.csv aggregating N10 and purity per tool per dataset size
- figure8_msrt_benchmark.png plotting N10 and purity versus dataset size, one curve per clustering tool

## How to apply

Run src/Clustering_benchmark_MS_RT.py for each tool on its normalized cluster-info file (columns filename, scan, mass/precursor_mz, rt_time, cluster), repeating the run across increasing subsampled dataset sizes with -tol 0.1 defining the mass-to-retention-time window and -t giving the number of MS/MS spectra. Collect the N10 and purity values emitted per run, aggregate all runs into msrt_benchmark_results.csv, then plot both metrics versus dataset size with one curve per tool, saving the figure as figure8_msrt_benchmark.png. Rationale: N10 captures clustering completeness (number of clusters covering ≥10 spectra) while purity captures accuracy, so plotting both against dataset size reveals which tool scales best — the paper's Figure 8 shows Falcon dominating (N10 ~1800, purity ~0.98), MS-Cluster intermediate (N10 1100–1200, purity 0.96–0.98), and MaRaCluster with the poorest trade-off. Supplementary SI Figures 9–13 show the same curve format extended to sampling-rate sweeps and instrument types (qToF MSV000081981, Orbitrap MSV000093033).

## Related tools

- **Clustering_benchmark_MS_RT.py** (Computes the N10 and purity values per clustering tool and dataset size that are aggregated and plotted as the benchmark curves) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **MS-RT** (Mass-to-retention-time evaluation method whose purity and N10 metrics form the y-axes of the benchmark curves) — https://github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **Falcon** (Clustering tool whose results (summarized via src/utility/summarize_results) supply the top-performing curve (N10 ~1800, purity ~0.98)) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool whose RT-merged results (via src/utility/maracluster_processing) supply the poorest N10/purity curve) — https://github.com/statisticalbiotechnology/maracluster
- **MS-Cluster** (msCluster results from the GNPS 2.0 Classical Networking Workflow (nf_output/clustering/clusterinfo.tsv) supply the intermediate N10 1100–1200 curve)

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c cluster_info.tsv -t 1000000 -methods falcon -tol 0.1
```

## Evaluation signals

- Falcon curve reaches N10 of about 1800 with purity ~0.98 across parameter settings, matching the reported Figure 8 trend
- MS-Cluster curve reaches N10 of 1100–1200 with purity 0.96–0.98
- MaRaCluster curve shows the poorest N10/purity trade-off relative to Falcon and MS-Cluster
- Every point on a tool's curve derives from a run whose cluster-info file has exactly the columns filename, scan, mass, rt_time, cluster
- msrt_benchmark_results.csv contains paired N10 and purity values for every subsampled dataset size and every tool plotted

## Limitations

- MS-RT is an indirect proxy: it relies on the assumption that identical molecules share similar precursor mass and retention time, and metabolomics lacks FDR-controlled identifications for external ground truth
- Curves depend on the chosen -tol mass-to-retention-time window tolerance (0.1 in the reported workflow); different tolerances shift absolute N10 and purity values
- Figure trends are dataset- and instrument-dependent — sampling-rate sweeps differ for qToF (MSV000081981) versus Orbitrap (MSV000093033) data

## Evidence

- [intro] Objective: use MS-RT to evaluate clustering tools across data sizes: "we use MS-RT to evaluate the performance of several MS/MS clustering tools in metabolomics and highlight the differences in the clustering performance of MS/MS datasets across different data"
- [supplementary] Supplementary figure showing N10 and purity benchmark curves: "SI Figure 11: Benchmarking results (N10 and purity) for different sampling rates of the MSV000081981 dataset (qToF)"
- [readme] Benchmark repository provides the tooling behind the curves: "Including tools to benchmark clusterng tools for metabolomics datasets"
