---
name: cross-dataset-clustering-ordering-comparison
description: Use when you have clustered spectra from ≥2 MS/MS clustering tools on
  ≥2 metabolomics datasets and need MS-RT purity/N10 scores to test whether tool rankings
  (e.g. Falcon vs MaRaCluster) transfer across instruments.
when_to_use_negative:
- Evaluation is on proteomics data where FDR-controlled database-search identifications
  exist — use the DB-Search comparison (src/clustering_eval_compare_DB) instead of
  MS-RT, which is a surrogate designed for metabolomics where 'FDR controlled identifications
  are not currently practical'
- Only one clustering tool or one dataset is available — a cross-tool, cross-dataset
  ordering comparison needs at least two tools and datasets to be meaningful
- Vendor raw files have not yet been converted — run msconvert/ThermoRawFileParser
  conversion and clustering first; this skill is the downstream evaluation step
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: MS-RT
  role: Retention-time-based benchmark metric used to score clustering purity and
    completeness (N10) per tool-dataset pair via Clustering_benchmark_MS_RT.py
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: MS-Cluster
  role: Clustering tool evaluated via GNPS 2.0 Classical Networking workflow; its
    clusterinfo.tsv already carries the required columns
- name: Falcon
  role: Clustering tool evaluated across its parameter grid; results reformatted to
    the benchmark schema with summarize_results
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool evaluated across its parameter grid; requires GNPS PerScanSummarizer
    RT retrieval merged via maracluster_processing before benchmarking
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: msconvert
  role: ProteoWizard tool converting vendor raw files to mzML/MGF before clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:c8ddc926bcd8ef45c0c005029bc0da4738551f52c4d47bd07a4786e4248e3548
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-dataset-clustering-ordering-comparison

## Summary

Benchmark MS/MS clustering tools (MS-Cluster, Falcon, MaRaCluster) across metabolomics datasets by running the MS-RT purity/completeness (N10) evaluation per tool-dataset pair and comparing whether the tools' relative orderings hold across instruments (Orbitrap vs qToF).

## When to use

Use when you have clustered spectra from ≥2 MS/MS clustering tools on ≥2 metabolomics datasets and need MS-RT purity/N10 scores to test whether tool rankings (e.g. Falcon vs MaRaCluster) transfer across instruments.

## When NOT to use

- Evaluation is on proteomics data where FDR-controlled database-search identifications exist — use the DB-Search comparison (src/clustering_eval_compare_DB) instead of MS-RT, which is a surrogate designed for metabolomics where 'FDR controlled identifications are not currently practical'
- Only one clustering tool or one dataset is available — a cross-tool, cross-dataset ordering comparison needs at least two tools and datasets to be meaningful
- Vendor raw files have not yet been converted — run msconvert/ThermoRawFileParser conversion and clustering first; this skill is the downstream evaluation step

## Inputs

- Per-tool cluster assignments as TSVs with columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx (GNPS 2.0 clusterinfo.tsv; Falcon summarized output; MaRaCluster RT-merged output)
- Converted mzML or MGF spectra files from ProteomeXchange datasets MSV000093033 (Orbitrap) and MSV000081981 (qToF)
- Total MS/MS spectrum count per dataset (e.g. 109333) for the -t benchmark parameter

## Outputs

- Per-tool-dataset MS-RT purity and N10 completeness scores
- Cross-dataset relative tool orderings reproducing the article's Figures 6-7 patterns

## How to apply

Run each clustering tool on each converted dataset (mzML/MGF from msconvert or ThermoRawFileParser), then normalize all outputs to the common schema (#Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx) — GNPS 2.0 Classical Networking's clusterinfo.tsv already conforms, Falcon results are reformatted via src/utility/summarize_results, and MaRaCluster results need RT from the GNPS 2.0 PerScanSummarizer workflow merged in via src/utility/maracluster_processing. Execute src/Clustering_benchmark_MS_RT.py once per tool-dataset combination with -c pointing to that tool's post-processed cluster_info file, -t set to the dataset's total MS/MS count (e.g. 109333 for the reference dataset), and -tol 0.1 for the MS-RT window. Collect the per-tool purity and N10 scores and compare orderings: on the Orbitrap algae dataset MSV000093033 MaRaCluster reaches purity 0.99 but the lowest N10, while Falcon attains the highest N10 (300) with purity 0.93; on the qTof human stool dataset MSV000081981 Falcon clusters 70–80% of spectra at moderate purity while MaRaCluster reaches purity up to 1 but clusters under 5–18%. The key decision point is instrument-dependent trade-off framing: completeness-oriented (Falcon) vs purity-oriented (MaRaCluster) rankings can flip priority depending on the dataset, so report orderings per dataset rather than as a single winner.

## Related tools

- **MS-RT** (Retention-time-based benchmark metric used to score clustering purity and completeness (N10) per tool-dataset pair via Clustering_benchmark_MS_RT.py) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **MS-Cluster** (Clustering tool evaluated via GNPS 2.0 Classical Networking workflow; its clusterinfo.tsv already carries the required columns)
- **Falcon** (Clustering tool evaluated across its parameter grid; results reformatted to the benchmark schema with summarize_results) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool evaluated across its parameter grid; requires GNPS PerScanSummarizer RT retrieval merged via maracluster_processing before benchmarking) — https://github.com/statisticalbiotechnology/maracluster
- **msconvert** (ProteoWizard tool converting vendor raw files to mzML/MGF before clustering)

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c data/falcon/falcon_cluster_info.tsv -t 109333 -tol 0.1
```

## Evaluation signals

- Clustering_benchmark_MS_RT.py completes without error for every tool-dataset combination (each of MS-Cluster, Falcon, MaRaCluster × MSV000093033, MSV000081981)
- Per-tool scores reproduce the Figures 6-7 relative orderings: on MSV000093033 MaRaCluster purity ≈0.99 with lowest N10 and Falcon highest N10 = 300 at purity ≈0.93
- On MSV000081981 (qTof), Falcon clusters ~70-80% of spectra at moderate purity while MaRaCluster reaches purity up to 1 but clusters <5-18% of spectra
- Every input cluster file conforms to the five required columns (#Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx) before benchmarking
- The same -t total MS/MS count and -tol 0.1 window are applied consistently across tools within each dataset so scores are comparable

## Limitations

- Metabolomics lacks FDR-controlled molecule identifications, so MS-RT ordering fidelity rests on MS1 retention-time consistency rather than identification ground truth
- Rankings are instrument- and dataset-dependent: qTof data show a completeness-vs-purity trade-off (Falcon vs MaRaCluster) that differs from the Orbitrap dataset, and supplementary figures show results vary with sampling rates of MSV000081981 and MSV000093033
- MaRaCluster output lacks retention times natively; RT must be fetched via the GNPS 2.0 PerScanSummarizer workflow and merged, adding an extra failure point before benchmarking

## Evidence

- [abstract] This MS-RT method and MS/MS clustering tool benchmarking will provide valuable real world practical recommendations: [evidence span withheld — anchor status: not_found]
