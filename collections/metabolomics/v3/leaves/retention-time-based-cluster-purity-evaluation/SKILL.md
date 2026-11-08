---
name: retention-time-based-cluster-purity-evaluation
description: Use when you need to assess how well a tool clusters MS/MS spectra from
  metabolomics datasets (qToF or Orbitrap, e.g. MSV000081981 or MSV000093033) without
  ground-truth identifications, or when benchmarking clustering purity against a database-search-based
  evaluation.
when_to_use_negative:
- When proteomics database search identifications with FDR control are already available
  and DB-Search-based evaluation can be run directly (e.g. PXD021518 proteomics data),
  DB-Search is the established evaluation method
- When the input is not MS/MS spectra (e.g. already a feature table or consensus spectra
  with no per-scan retention time), MS-RT purity scoring is not applicable
- 'When the clustering tool output cannot be mapped to per-scan precursor m/z and
  retention time (missing #ParentMass or #RetTime columns), the method cannot be computed'
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-Cluster
  role: MS/MS clustering tool whose output (clusterinfo.tsv from GNPS 2.0 Classical
    Networking Workflow) is scored for purity by MS-RT
- name: Falcon
  role: MS/MS clustering tool whose results are summarized with src/utility/summarize_results
    and then evaluated with MS-RT
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: MS/MS clustering tool whose results are merged with retention time and then
    evaluated with MS-RT
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: DB-Search
  role: Established proteomics evaluation approach used to validate MS-RT
- name: msconvert
  role: ProteoWizard tool to convert vendor raw files to mzML or MGF before clustering
  repo: http://proteowizard.sourceforge.net/download.html
- name: ThermoRawFileParser
  role: Cross-platform converter of Thermo RAW files to mzML or MGF prior to clustering
  repo: https://github.com/compomics/ThermoRawFileParser
- name: Python 3.8
  role: Runtime environment for the benchmarking and utility scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:d2f73e861e74b4746d94a3dae5c048108504ca1405f0886153a360a9e1f31ee1
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
  - ProteoWizard
  - Python
  - Python 3.8
  - ThermoRawFileParser
  - msconvert
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# retention-time-based-cluster-purity-evaluation

## Summary

MS-RT evaluates MS/MS clustering purity in metabolomics by exploiting the observation that MS/MS spectra of the same molecule share MS1 precursor m/z and retention time, avoiding the need for FDR-controlled identifications. It is validated against database-search-based evaluation on proteomics data and applied to benchmark MS-Cluster, Falcon, and MaRaCluster on metabolomics datasets.

## When to use

Use when you need to assess how well a tool clusters MS/MS spectra from metabolomics datasets (qToF or Orbitrap, e.g. MSV000081981 or MSV000093033) without ground-truth identifications, or when benchmarking clustering purity against a database-search-based evaluation.

## When NOT to use

- When proteomics database search identifications with FDR control are already available and DB-Search-based evaluation can be run directly (e.g. PXD021518 proteomics data), DB-Search is the established evaluation method
- When the input is not MS/MS spectra (e.g. already a feature table or consensus spectra with no per-scan retention time), MS-RT purity scoring is not applicable
- When the clustering tool output cannot be mapped to per-scan precursor m/z and retention time (missing #ParentMass or #RetTime columns), the method cannot be computed

## Inputs

- Vendor raw mass spectrometry data (e.g. Thermo .raw) from ProteomeXchange datasets such as PXD021518, PXD023047, PXD02047, MSV000081981, or MSV000093033
- Converted mzML or MGF spectrum files
- Clustering output from MS-Cluster (msCluster), Falcon, or MaRaCluster
- Retention time information (e.g. from GNPS 2.0 PerScanSummarizer Workflow results.csv) when not already present in the clustering output
- Total number of MS/MS spectra for normalization (e.g. 390746)

## Outputs

- MS-RT cluster purity estimate per clustering tool and parameter setting
- Consistency heatmap of MS-RT vs. DB-Search (e.g. SI Figure 4 for PXD021518)
- Benchmarking results (purity and N10) across sampling rates and instrument types, e.g. SI Figures 11 (qToF MSV000081981) and 13 (Orbitrap MSV000093033)
- Formatted clusterinfo table with columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx

## How to apply

Download the dataset from ProteomeXchange and convert vendor raw files to mzML or MGF with msconvert (ProteoWizard) or ThermoRawFileParser. Cluster the spectra with MS-Cluster (easiest via the GNPS 2.0 Classical Networking Workflow, retrieving nf_output/clustering/clusterinfo.tsv with columns '#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx'), Falcon, or MaRaCluster (merging retention time via the GNPS 2.0 PerScanSummarizer Workflow results file nf_output/results.csv). Run the benchmarking script src/Clustering_benchmark_MS_RT.py with the total number of MS/MS spectra for normalization and a mass-to-retention-time tolerance (e.g. -tol 0.1 in the consistency window Y = X ± 0.1). The method computes cluster purity by scoring how many clusters contain spectra consistent in m/z and retention time; validation against DB-Search on proteomics data (PXD021518, and jointly PXD02047/PXD021518) establishes that the purity estimate tracks the established approach, and SI figures report results for different sampling rates on qToF (MSV000081981) and Orbitrap (MSV000093033) instruments.

## Related tools

- **MS-Cluster** (MS/MS clustering tool whose output (clusterinfo.tsv from GNPS 2.0 Classical Networking Workflow) is scored for purity by MS-RT)
- **Falcon** (MS/MS clustering tool whose results are summarized with src/utility/summarize_results and then evaluated with MS-RT) — https://github.com/bittremieux/falcon
- **MaRaCluster** (MS/MS clustering tool whose results are merged with retention time and then evaluated with MS-RT) — https://github.com/statisticalbiotechnology/maracluster
- **DB-Search** (Established proteomics evaluation approach used to validate MS-RT)
- **msconvert** (ProteoWizard tool to convert vendor raw files to mzML or MGF before clustering) — http://proteowizard.sourceforge.net/download.html
- **ThermoRawFileParser** (Cross-platform converter of Thermo RAW files to mzML or MGF prior to clustering) — https://github.com/compomics/ThermoRawFileParser
- **Python 3.8** (Runtime environment for the benchmarking and utility scripts)

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/mscluster/mscluster_cluster_info.tsv -t 390746 -methods mscluster -tol 0.1
```

## Evaluation signals

- Clustering output must expose the required columns ('filename' → #Filename, 'scan' → #Scan, 'mass' → #ParentMass, 'rt_time' → #RetTime, 'cluster' → #ClusterIdx) before benchmarking
- MS-RT purity should track DB-Search purity when both are applied to the same proteomics dataset (e.g. SI Figure 4 consistency heatmap for PXD021518, and joint PXD02047/PXD021518 SI Figure 9)
- Run the benchmark script with the correct total number of MS/MS spectra (-t) so purity is properly normalized (e.g. 390746 in the example)
- Compare purity and N10 across sampling-rate sweeps (SI Figures 11 and 13 for MSV000081981 and MSV000093033) to confirm stable ranking of tools across sampling rates
- Verify that XIC spectra used for spot checks (e.g. EK_Q_07.mzML at MS-Cluster 1293 of PXD023047) show consistent m/z and retention time within the same cluster

## Limitations

- MS-RT is a proxy evaluation: it relies on the assumption that repeated acquisitions of the same molecule have consistent precursor m/z and retention time, and it is validated against, not a substitute for, FDR-controlled identifications
- Mass spectrometry-based metabolomics lacks FDR-controlled identifications, so MS-RT cannot be cross-checked against a ground truth there; the validation is done on proteomics data instead
- The benchmark depends on vendor raw files being converted to mzML/MGF and on clustering tools' outputs being post-processed into a common column format; inconsistent formats break the pipeline

## Evidence

- [intro] Clustering tool set: [evidence span withheld — anchor status: not_found]
