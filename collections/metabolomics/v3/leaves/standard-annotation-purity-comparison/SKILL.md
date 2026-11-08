---
name: standard-annotation-purity-comparison
description: Use when validating a metabolomics MS/MS clustering purity metric (e.g.,
  MS-RT) against ground-truth manual annotations of chemical standards, where a standard-library
  dataset with known compound identities is available and FDR-controlled database
  search identifications are not.
when_to_use_negative:
- Working on proteomics datasets where FDR-controlled database search identifications
  exist — use the DB-Search evaluation path instead of standard-annotation comparison
- The dataset contains no manually annotated chemical standards, so no ground truth
  is available for purity comparison
- 'Clustering output lacks the required columns (#Filename, #Scan, #ParentMass, #RetTime,
  #ClusterIdx) or retention time information, making MS-RT purity computation impossible'
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-Cluster
  role: Cluster MS/MS spectra via the GNPS 2.0 Classical Networking Workflow to produce
    the clusterinfo.tsv evaluated for purity
- name: Clustering_benchmark_MS_RT.py
  role: Benchmarking script that computes MS-RT cluster purity and compares it against
    manual chemical standard annotations at tolerance -tol 0.1
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- name: msconvert
  role: ProteoWizard tool to convert vendor raw files to mzML before clustering
- name: ThermoRawFileParser
  role: Alternative cross-platform converter of Thermo RAW files to mzML or MGF format
  repo: https://github.com/compomics/ThermoRawFileParser
- name: Python 3.8
  role: Runtime environment (virtualenv 'Clustering_benchmarking') for the benchmarking
    and post-processing scripts
  repo: github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00881
    title: 'MS-RT: A Method for Evaluating MS/MS Clustering Performance for Metabolomics
      Data'
  source_content_hash: sha256:7014277768bfceecbe53dc89e17b8c913f25cd9ec22346f41179b9f348138529
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Clustering_benchmark_MS_RT.py
  - MS-Cluster
  - MS-RT
  - ProteoWizard
  - Python
  - Python 3.8
  - ThermoRawFileParser
  - msconvert
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# standard-annotation-purity-comparison

## Summary

Validates the MS-RT purity method for metabolomics MS/MS clustering by benchmarking cluster purity against manually annotated chemical standard identifications (2,179 standards, MSV000080554), demonstrating >95% consistency between MS-RT and manual m/z-matched annotations.

## When to use

Use when validating a metabolomics MS/MS clustering purity metric (e.g., MS-RT) against ground-truth manual annotations of chemical standards, where a standard-library dataset with known compound identities is available and FDR-controlled database search identifications are not.

## When NOT to use

- Working on proteomics datasets where FDR-controlled database search identifications exist — use the DB-Search evaluation path instead of standard-annotation comparison
- The dataset contains no manually annotated chemical standards, so no ground truth is available for purity comparison
- Clustering output lacks the required columns (#Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx) or retention time information, making MS-RT purity computation impossible

## Inputs

- MSV000080554 standard-library raw mass spectrometry data from ProteomeXchange
- Vendor raw files converted to mzML format (msconvert or ThermoRawFileParser)
- MS-Cluster clustering results: nf_output/clustering/clusterinfo.tsv with columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx
- Manual m/z-matched annotations of 2,179 chemical standards
- Total MS/MS spectra count for normalization (390,746)

## Outputs

- Cluster consistency fraction (MS-RT vs manual annotations within Y = X ± 0.1; expected 2,309/2,414, >95%)
- Average purity for MS-RT (expected 0.919) and manual annotation (expected 0.928), reproducing SI Figure 14
- Results table of benchmark metrics

## How to apply

Download the MSV000080554 standard-library dataset from ProteomeXchange and convert vendor raw files to mzML using msconvert (ProteoWizard) or ThermoRawFileParser. Cluster the spectra with MS-Cluster via the GNPS 2.0 Classical Networking Workflow and retrieve nf_output/clustering/clusterinfo.tsv, which carries the required columns #Filename, #Scan, #ParentMass, #RetTime, and #ClusterIdx. Run the benchmark script src/Clustering_benchmark_MS_RT.py with -c pointing at the clusterinfo TSV, -t set to the total number of MS/MS spectra (390,746) for normalization, and -tol 0.1 setting the mass-to-retention-time (MS-RT) window tolerance. The script computes MS-RT cluster purity and compares it to manual m/z-matched annotations of the 2,179 chemical standards, counting clusters whose spectra fall within the Y = X ± 0.1 consistency window. Record the cluster consistency fraction (expected >95%, 2,309/2,414 clusters) and average purity (expected 0.919 MS-RT vs 0.928 manual, reproducing SI Figure 14) in a results table.

## Related tools

- **MS-Cluster** (Cluster MS/MS spectra via the GNPS 2.0 Classical Networking Workflow to produce the clusterinfo.tsv evaluated for purity)
- **Clustering_benchmark_MS_RT.py** (Benchmarking script that computes MS-RT cluster purity and compares it against manual chemical standard annotations at tolerance -tol 0.1) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark
- **msconvert** (ProteoWizard tool to convert vendor raw files to mzML before clustering)
- **ThermoRawFileParser** (Alternative cross-platform converter of Thermo RAW files to mzML or MGF format) — https://github.com/compomics/ThermoRawFileParser
- **Python 3.8** (Runtime environment (virtualenv 'Clustering_benchmarking') for the benchmarking and post-processing scripts) — github.com/XianghuWang-287/Metabolomics_Clustering_Benchmark

## Examples

```
python3 src/Clustering_benchmark_MS_RT.py -c ./data/mscluster/mscluster_cluster_info.tsv -t 390746 -methods mscluster -tol 0.1
```

## Evaluation signals

- Cluster consistency fraction of MS-RT vs manual annotations exceeds 95% (2,309 out of 2,414 clusters within the Y = X ± 0.1 window)
- Average purity of MS-RT reaches 0.919 vs 0.928 for manual annotation, reproducing SI Figure 14
- Input clusterinfo.tsv contains the exact columns #Filename, #Scan, #ParentMass, #RetTime, #ClusterIdx
- Normalization uses the correct total MS/MS spectra count (390,746) passed via -t
- The MS-RT tolerance window is set to 0.1 via -tol, matching the Y = X ± 0.1 consistency criterion

## Limitations

- Requires a manually annotated chemical standard library dataset (2,179 standards) as ground truth, which is not generally available for complex metabolomics samples
- Metabolomics lacks FDR-controlled identifications, so external ground-truth evaluation of clustering is otherwise impractical — this skill only applies where standard annotations exist
- The Y = X ± 0.1 consistency window and purity thresholds are specific to this benchmark setup and may need adjustment for other instruments (e.g., qToF vs Orbitrap datasets like MSV000081981 and MSV000093033)

## Evidence

- [results] MS-RT purity vs manual standards: [evidence span withheld — anchor status: not_found]
