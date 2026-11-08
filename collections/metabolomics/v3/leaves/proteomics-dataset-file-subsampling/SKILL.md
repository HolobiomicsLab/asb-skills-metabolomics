---
name: proteomics-dataset-file-subsampling
description: 'Use when benchmarking MS/MS clustering tools (MS-Cluster, Falcon) across
  dataset sizes: create random file-level subsamples at fixed fractions from a large
  qToF dataset such as MSV000081981.'
when_to_use_negative:
- When the question concerns clustering quality at a single fixed dataset size rather
  than scaling behaviour across sizes
- When the dataset has too few files for a meaningful fraction series, making each
  subsample nearly the full dataset
- When evaluating tools on identification quality rather than clustering purity/completeness
  scaling
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS-Cluster
  role: Clustering tool run on each file-level subsample to measure how purity and
    N10 change with sampling rate
- name: Falcon
  role: Clustering tool benchmarked on the same subsamples; reaches ~2500 N10 at 70%
    sampling vs ~1000 for MS-Cluster
  repo: https://github.com/bittremieux/falcon
- name: MS-RT
  role: Benchmarking script applied to each subsample's clustering output to derive
    purity and N10
- name: msconvert
  role: Converts the downloaded dataset's raw files to mzML before subsampling
- name: ThermoRawFileParser
  role: Alternative cross-platform converter of Thermo RAW files to mzML/MGF prior
    to subsampling
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:5a38932a08987bca7e74ce2b9a8f341d4fbf921e473f25a871b3362ba528b27a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Falcon
  - MS-Cluster
  - MS-RT
  - ProteoWizard
  - Python
  - ThermoRawFileParser
  - msconvert
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# proteomics-dataset-file-subsampling

## Summary

Generates file-level random subsamples of a mass-spectrometry dataset (e.g. 10–100% of the 2,307 mzML files in MSV000081981) so clustering tools can be benchmarked as a function of dataset size on purity and N10 completeness.

## When to use

Use when benchmarking MS/MS clustering tools (MS-Cluster, Falcon) across dataset sizes: create random file-level subsamples at fixed fractions from a large qToF dataset such as MSV000081981.

## When NOT to use

- When the question concerns clustering quality at a single fixed dataset size rather than scaling behaviour across sizes
- When the dataset has too few files for a meaningful fraction series, making each subsample nearly the full dataset
- When evaluating tools on identification quality rather than clustering purity/completeness scaling

## Inputs

- Pool of mzML files converted from a ProteomeXchange dataset (e.g. MSV000081981, 2,307 qToF files)
- Subsampling fraction or list of fractions (10%, 20%, …, 100%)

## Outputs

- One mzML file-set per sampling fraction
- Per-fraction cluster_info files from MS-Cluster/Falcon runs
- Sampling-rate benchmarking table of purity and N10 vs fraction (cf. SI Figure 11)

## How to apply

Download the full dataset from ProteomeXchange/MassIVE and convert all vendor files to mzML with msconvert or ThermoRawFileParser. From the pool of mzML files (2,307 files for MSV000081981), draw random file-level subsamples at fixed fractions — 10%, 20%, …, 100% — keeping each fraction a distinct benchmarking point. Run the clustering tool being evaluated (e.g. MS-Cluster via the GNPS 2.0 Classical Networking Workflow, or Falcon) identically on every subsample so only dataset size varies. After benchmarking each subsample's cluster_info file with MS-RT, compile purity and N10 against sampling fraction (as in SI Figure 11 for MSV000081981) to reveal how each tool's completeness scales with data volume. The rationale is that clustering behaviour changes with data size, so per-tool comparisons must be repeated across fractions rather than at a single full-dataset run.

## Related tools

- **MS-Cluster** (Clustering tool run on each file-level subsample to measure how purity and N10 change with sampling rate)
- **Falcon** (Clustering tool benchmarked on the same subsamples; reaches ~2500 N10 at 70% sampling vs ~1000 for MS-Cluster) — https://github.com/bittremieux/falcon
- **MS-RT** (Benchmarking script applied to each subsample's clustering output to derive purity and N10)
- **msconvert** (Converts the downloaded dataset's raw files to mzML before subsampling)
- **ThermoRawFileParser** (Alternative cross-platform converter of Thermo RAW files to mzML/MGF prior to subsampling) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/mscluster/mscluster_cluster_info.tsv -t <number_of_msms> -methods mscluster -tol 0.1
```

## Evaluation signals

- At 10% sampling both MS-Cluster and Falcon show purity near 0.992–0.998 with N10 in the low hundreds
- At 70% sampling of MSV000081981 both tools converge to average purity ~0.9986 while Falcon's N10 (~2500) exceeds MS-Cluster's (~1000)
- Each fraction's cluster_info.tsv contains the required columns (#Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx)
- The sampling curve (purity and N10 vs fraction) reproduces the trends of SI Figure 11 for MSV000081981
- Subsample counts equal the intended fraction of the 2,307-file pool for each benchmarking point

## Limitations

- Sampling rate affects purity and N10 differently: equal purity can coincide with very different N10, so both metrics must be reported together
- Requires the full dataset to be downloaded and converted first; conclusions are dataset- and instrument-specific (qToF MSV000081981, Orbitrap MSV000093033)
- File-level subsampling is random, so repeated draws may shift purity/N10 slightly

## Evidence
