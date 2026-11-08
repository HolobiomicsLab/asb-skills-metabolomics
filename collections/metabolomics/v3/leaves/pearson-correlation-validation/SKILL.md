---
name: pearson-correlation-validation
description: Use when a newly proposed clustering evaluation metric must be validated
  against an established ground-truth evaluation on a dataset where reference results
  exist, via Pearson correlation.
when_to_use_negative:
- Input is a metabolomics dataset with no published database-search-based completeness
  reference, since FDR-controlled identifications are not practical in metabolomics
  and no ground-truth correlation target exists
- The clustering results have not been post-processed into the standardized format
  (filename, scan, mass, rt_time, cluster) expected by Clustering_benchmark_MS_RT.py
- The goal is purity assessment rather than completeness validation of a clustering
  metric
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Python 3.8
  role: Runtime for the benchmarking script Clustering_benchmark_MS_RT.py that computes
    N10 and the Pearson correlation
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: Falcon
  role: MS/MS clustering tool whose output is post-processed and benchmarked to produce
    N10 values for correlation against published completeness
  repo: https://github.com/bittremieux/falcon
- name: msconvert
  role: ProteoWizard tool used to convert vendor RAW files (e.g. the 2,212 PXD000561
    RAW files) to mzML before clustering
- name: ThermoRawFileParser
  role: Alternative cross-platform converter from Thermo RAW to mzML/MGF prior to
    clustering
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:17b704525785af1efa74aa3e0d8b8513fc3b1576832c7d9642e8ccdacfcec6a6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Falcon
  - ProteoWizard
  - Python
  - Python 3.8
  - ThermoRawFileParser
  - msconvert
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pearson-correlation-validation

## Summary

Validates a new clustering completeness metric (N10) by computing the Pearson correlation between N10 values and previously published database-search-based proteomics completeness results, checking against the reported benchmark of 0.984 on PXD000561.

## When to use

Use when a newly proposed clustering evaluation metric must be validated against an established ground-truth evaluation on a dataset where reference results exist, via Pearson correlation.

## When NOT to use

- Input is a metabolomics dataset with no published database-search-based completeness reference, since FDR-controlled identifications are not practical in metabolomics and no ground-truth correlation target exists
- The clustering results have not been post-processed into the standardized format (filename, scan, mass, rt_time, cluster) expected by Clustering_benchmark_MS_RT.py
- The goal is purity assessment rather than completeness validation of a clustering metric

## Inputs

- Post-processed MS/MS cluster info file (columns: filename, scan, precursor mass, retention time, cluster index)
- Total spectrum count for the dataset (e.g. 24,870,542 for PXD000561)
- Previously published proteomics completeness results from database-search-based evaluation
- Clustering method identifier (e.g. Falcon, MaRaCluster, msCluster)

## Outputs

- N10 completeness metric values per clustering method
- Pearson correlation coefficient between N10 and published proteomics completeness (expected 0.984 on PXD000561)
- Comparison plot analogous to SI Figure 5

## How to apply

Run the selected MS/MS clustering tool (e.g. Falcon) on vendor RAW files converted to mzML with msconvert or ThermoRawFileParser, then post-process the output into the standardized cluster info format with columns filename, scan, precursor mass, retention time, and cluster index. Feed this file to the benchmarking script src/Clustering_benchmark_MS_RT.py with -c for the clustering file, -t for the total spectrum count (24,870,542 for PXD000561), and -methods for the clustering method. From the clustering results compute N10, defined as the size of the smallest cluster in the smallest set of clusters whose combined member count reaches 10% of the total dataset size. Compute the Pearson correlation between the N10 values and the previously published proteomics completeness results obtained from database-search-based evaluation, and verify the coefficient against the reported value of 0.984 (SI Figure 5). The rationale is that MS-RT and N10 need external validation on proteomics datasets where FDR-controlled database search identifications provide a well-established evaluation ground truth.

## Related tools

- **Python 3.8** (Runtime for the benchmarking script Clustering_benchmark_MS_RT.py that computes N10 and the Pearson correlation) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **Falcon** (MS/MS clustering tool whose output is post-processed and benchmarked to produce N10 values for correlation against published completeness) — https://github.com/bittremieux/falcon
- **msconvert** (ProteoWizard tool used to convert vendor RAW files (e.g. the 2,212 PXD000561 RAW files) to mzML before clustering)
- **ThermoRawFileParser** (Alternative cross-platform converter from Thermo RAW to mzML/MGF prior to clustering) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
python src/Clustering_benchmark_MS_RT.py -c data/falcon/falcon_cluster_info.tsv -t 24870542 -methods falcon
```

## Evaluation signals

- Computed Pearson correlation between N10 and published proteomics completeness matches the reported 0.984 on PXD000561 (SI Figure 5)
- N10 is computed correctly: the smallest cluster within the smallest set of clusters whose combined member count reaches 10% of the total spectrum count supplied via -t
- Cluster info file contains all required columns (filename, scan, precursor mass, retention time, cluster index) before benchmarking
- Total spectrum count passed to -t matches the dataset's actual MS/MS spectrum count (24,870,542 for PXD000561)
- Correlation is high (near 0.984), confirming N10 is a valid proxy for clustering completeness

## Limitations

- Validation requires proteomics datasets with established database-search-based evaluation; metabolomics lacks FDR-controlled identifications, so no external ground truth exists there
- The result depends on the clustering tool and its parameter settings; completeness varies across tools (MS-Cluster, MaRaCluster, Falcon) and parameters
- The 0.984 value is specific to PXD000561 with a total of 24,870,542 spectra; other datasets or sampling rates will yield different correlations

## Evidence

- [results] N10 Pearson correlation of 0.984 with proteomics completeness: [evidence span withheld — anchor status: not_found]
- [methods] N10 metric definition: [evidence span withheld — anchor status: not_found]
- [methods] Benchmarking script invocation: [evidence span withheld — anchor status: not_found]
- [methods] Dataset scale: [evidence span withheld — anchor status: not_found]
