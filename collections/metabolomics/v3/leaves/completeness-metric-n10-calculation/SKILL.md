---
name: completeness-metric-n10-calculation
description: Use when you have post-processed MS/MS clustering output (filename, scan,
  precursor mass, retention time, cluster index) and need a database-search-free completeness
  metric for benchmarking tools like Falcon, MaRaCluster, or msCluster.
when_to_use_negative:
- The clustering results have not yet been post-processed into the standardized cluster
  info format with a cluster index column — run the summarization/post-processing
  step first.
- The goal is to assess clustering purity (within-cluster spectral similarity) rather
  than completeness; N10 measures only completeness.
- No reliable total spectrum count (-t) can be supplied, since N10 is defined relative
  to 10% of the total dataset size.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Python 3.8
  role: Runtime for the benchmarking script src/Clustering_benchmark_MS_RT.py that
    computes N10 from cluster info files
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: Falcon
  role: MS/MS clustering tool whose output is post-processed and fed to the N10 benchmarking
    script
  repo: https://github.com/bittremieux/falcon
- name: msconvert (ProteoWizard)
  role: Converts vendor RAW files (e.g. the 2,212 RAW files of PXD000561) to mzML
    before clustering
- name: MS-Cluster
  role: Clustering tool runnable via the GNPS 2.0 Classical Networking Workflow producing
    clusterinfo.tsv usable as input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:8d8d06e6b90f154e61bd0fcc5850c1cbdfbc91a5b5c6268f5ffbdbd5b768cf79
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Falcon
  - MS-Cluster
  - ProteoWizard
  - Python
  - Python 3.8
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# completeness-metric-n10-calculation

## Summary

Computes the N10 clustering completeness metric — the size of the smallest cluster in the smallest set of clusters whose combined member count reaches 10% of the total dataset size — from standardized MS/MS cluster info files, and validates it as a proxy for proteomics completeness (Pearson r = 0.984 on PXD000561, SI Figure 5).

## When to use

Use when you have post-processed MS/MS clustering output (filename, scan, precursor mass, retention time, cluster index) and need a database-search-free completeness metric for benchmarking tools like Falcon, MaRaCluster, or msCluster.

## When NOT to use

- The clustering results have not yet been post-processed into the standardized cluster info format with a cluster index column — run the summarization/post-processing step first.
- The goal is to assess clustering purity (within-cluster spectral similarity) rather than completeness; N10 measures only completeness.
- No reliable total spectrum count (-t) can be supplied, since N10 is defined relative to 10% of the total dataset size.

## Inputs

- Post-processed cluster info file (TSV/CSV) with columns filename, scan, precursor mass, retention time, cluster index
- Total spectrum count for the dataset (e.g. 24,870,542 for PXD000561)
- Previously published proteomics completeness values for correlation validation (optional)

## Outputs

- N10 value (size of the smallest cluster in the smallest set of clusters covering 10% of the total dataset size)
- Pearson correlation of N10 against published proteomics completeness results

## How to apply

Run a chosen MS/MS clustering tool (e.g. Falcon on mzML files) and post-process its output into the standardized cluster info format with columns filename, scan, precursor mass, retention time, and cluster index. Load that file with the provided benchmarking script src/Clustering_benchmark_MS_RT.py, supplying the clustering file with -c, the total spectrum count with -t (for PXD000561 this is 24,870,542 across 2,212 RAW files), and the clustering method with -methods. The script computes N10: sort clusters and take the smallest set of clusters whose combined member count reaches 10% of the total dataset size; N10 is the size of the smallest cluster in that set. Lower N10 indicates fewer spectra fall into poorly covered clusters, i.e. better completeness. Validate correctness by computing the Pearson correlation of N10 against the published database-search-based proteomics completeness values, which should reproduce 0.984 (SI Figure 5).

## Related tools

- **Python 3.8** (Runtime for the benchmarking script src/Clustering_benchmark_MS_RT.py that computes N10 from cluster info files) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **Falcon** (MS/MS clustering tool whose output is post-processed and fed to the N10 benchmarking script) — https://github.com/bittremieux/falcon
- **msconvert (ProteoWizard)** (Converts vendor RAW files (e.g. the 2,212 RAW files of PXD000561) to mzML before clustering)
- **MS-Cluster** (Clustering tool runnable via the GNPS 2.0 Classical Networking Workflow producing clusterinfo.tsv usable as input)

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c data/mscluster/mscluster_cluster_info.tsv -t 24870542 -methods mscluster
```

## Evaluation signals

- Pearson correlation between computed N10 and published proteomics completeness results matches 0.984 on PXD000561 (SI Figure 5)
- N10 equals the size of the smallest cluster in the smallest set of clusters whose combined member count reaches 10% of the supplied total (-t)
- The total spectrum count supplied with -t matches the dataset (24,870,542 for PXD000561's 2,212 RAW files)
- Input cluster info file carries the required columns: filename, scan, precursor mass, retention time, cluster index

## Limitations

- N10's validation against database-search-based completeness was performed on proteomics datasets (e.g. PXD000561); metabolomics lacks FDR-controlled identifications for equivalent ground-truth evaluation.
- The metric requires an externally supplied total spectrum count, so errors in that count propagate directly into the 10% threshold.
- N10 captures completeness only; a separate purity evaluation is needed for a full picture of clustering performance.

## Evidence

- [supplementary] N10 validated against published proteomics completeness: [evidence span withheld — anchor status: not_found]
- [methods] Benchmarking script invocation and parameters: [evidence span withheld — anchor status: not_found]
- [methods] N10 definition: [evidence span withheld — anchor status: not_found]
