---
name: tandem-ms-spectra-clustering
description: Use when a metabolomics or proteomics data-dependent acquisition dataset
  (vendor raw or mzML/MGF) contains repeated MS/MS spectra of the same molecule and
  redundancy reduction via clustering into consensus clusters is needed.
when_to_use_negative:
- When the dataset is already a de-redundified feature table with no repeated MS/MS
  spectra left to cluster
- When molecule identifications with FDR control are available and external database-search
  evaluation, rather than clustering-based deduplication, is the goal
- When spectra come from a single acquisition with no redundancy, since clustering
  adds no value for de-duplication
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3520
tools:
- name: MS-Cluster
  role: Primary clustering tool executed via the GNPS 2.0 Classical Networking Workflow;
    outputs clusterinfo.tsv used for downstream benchmarking
- name: Falcon
  role: Spectrum clustering tool using feature hashing, nearest-neighbor search, and
    density-based clustering for millions of MS/MS spectra
  repo: https://github.com/bittremieux/falcon
- name: MaRaCluster
  role: Clustering tool run via its batch and consensus sub-commands on a flat file
    listing mz2 spectrum files
  repo: https://github.com/statisticalbiotechnology/maracluster
- name: msconvert
  role: ProteoWizard tool used to convert vendor raw data to mzML before clustering
- name: ThermoRawFileParser
  role: Cross-platform converter of Thermo RAW files to mzML or MGF formats
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:c07d8c866389a57716ed0333112e0ab6e6ca53a23a2223d43f1cdf6d09a4b806
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
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# tandem-ms-spectra-clustering

## Summary

Deduplicate repeated tandem mass spectra from metabolomics acquisitions by grouping MS/MS spectra into clusters using tools such as MS-Cluster, Falcon, and MaRaCluster, enabling redundancy reduction and downstream molecular networking on large datasets.

## When to use

Use when a metabolomics or proteomics data-dependent acquisition dataset (vendor raw or mzML/MGF) contains repeated MS/MS spectra of the same molecule and redundancy reduction via clustering into consensus clusters is needed.

## When NOT to use

- When the dataset is already a de-redundified feature table with no repeated MS/MS spectra left to cluster
- When molecule identifications with FDR control are available and external database-search evaluation, rather than clustering-based deduplication, is the goal
- When spectra come from a single acquisition with no redundancy, since clustering adds no value for de-duplication

## Inputs

- Vendor raw mass spectrometry files (e.g. Thermo .raw) from data-dependent acquisition
- Converted mzML or MGF peak-list files (one file per line for MaRaCluster batch)
- Publicly available datasets from ProteomeXchange (e.g. MSV000081981, MSV000093033, PXD023047, PXD021518)

## Outputs

- Cluster assignments mapping each MS/MS scan to a cluster identifier (e.g. clusterinfo.tsv with columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx)
- Formatted clustering results ready for MS-RT benchmarking or DB-Search comparison
- Purity and completeness (N10) metrics per clustering tool and sampling rate

## How to apply

First convert vendor raw files (e.g. Thermo .raw) to open formats with msconvert or ThermoRawFileParser, exporting mzML (or MGF for tools requiring peak lists). Then run one of the clustering tools: MS-Cluster via the GNPS 2.0 Classical Networking Workflow (downloading nf_output/clustering/clusterinfo.tsv), Falcon via its pip-installed command-line interface which bins high-resolution spectra into low-dimensional vectors using feature hashing, builds nearest-neighbor indexes to compute a sparse pairwise distance matrix, and performs density-based clustering, or MaRaCluster via its batch sub-command. For MS-Cluster the required output columns are #Filename, #Scan, #ParentMass, #RetTime, and #ClusterIdx. Finally, benchmark cluster quality using the MS-RT method (src/Clustering_benchmark_MS_RT.py with a 0.1 tolerance) or, in proteomics, against database-search identifications with FDR control.

## Related tools

- **MS-Cluster** (Primary clustering tool executed via the GNPS 2.0 Classical Networking Workflow; outputs clusterinfo.tsv used for downstream benchmarking)
- **Falcon** (Spectrum clustering tool using feature hashing, nearest-neighbor search, and density-based clustering for millions of MS/MS spectra) — https://github.com/bittremieux/falcon
- **MaRaCluster** (Clustering tool run via its batch and consensus sub-commands on a flat file listing mz2 spectrum files) — https://github.com/statisticalbiotechnology/maracluster
- **msconvert** (ProteoWizard tool used to convert vendor raw data to mzML before clustering)
- **ThermoRawFileParser** (Cross-platform converter of Thermo RAW files to mzML or MGF formats) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
maracluster batch --input mzml_files.txt --output clusters.tsv; falcon-ms --config falcon.ini; ThermoRawFileParser.exe -i your_file.raw -o output_folder -f=1
```

## Evaluation signals

- Output file contains the required columns '#Filename', '#Scan', '#ParentMass', '#RetTime', '#ClusterIdx' (for MS-Cluster results)
- MS-RT benchmark script (src/Clustering_benchmark_MS_RT.py -c <cluster_info_file> -t <number_of_msms> -methods mscluster -tol 0.1) completes without error and produces purity and N10 values
- Purity values should approach 1.0 (e.g. ~0.998 at 70% sampling on MSV000081981) while N10 reflects the number of large clusters
- At equal sampling fractions, Falcon's N10 (~2500) should exceed MS-Cluster's (~1000) on qToF data while purity values converge to nearly the same value
- Clustering results merge consistently with retention-time information extracted via the GNPS 2.0 PerScanSummarizer Workflow for downstream analysis

## Limitations

- In metabolomics, FDR-controlled identifications are not currently practical, so external ground-truth evaluation of clustering quality (as done in proteomics via database search) is not available; the MS-RT method is used instead
- Clustering performance varies with dataset size: at small sampling rates (10%) purity is near 0.992-0.998 with low N10, and only at larger sampling fractions do differences between tools (e.g. Falcon's larger N10) become apparent
- Different instrument platforms (qToF vs Orbitrap) show different benchmarking behavior (SI Figures 11 and 13), so conclusions from one platform may not transfer

## Evidence

- [readme] Falcon pipeline: hashing, nearest-neighbor, density clustering: [evidence span withheld — anchor status: external_doc]
