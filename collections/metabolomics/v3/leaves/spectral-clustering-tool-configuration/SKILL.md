---
name: spectral-clustering-tool-configuration
description: Use when profiling the runtime breakdown of CPU-only spectral clustering
  tools on large MS/MS data sets (e.g. PXD000561 MGF files) to quantify the preprocessing
  bottleneck before benchmarking a faster method such as HyperSpec.
when_to_use_negative:
- Input spectra are already preprocessed consensus spectra rather than raw MGF peak
  files — the bottleneck being profiled exists in raw-file preprocessing, not consensus
  handling.
- The goal is clustering quality comparison rather than stage-wise runtime profiling
  — this skill measures time distribution, not incorrect clustering ratio or completeness.
- The target tool is GPU-accelerated (e.g. HyperSpec) — its preprocessing modules
  are optimized to bypass the CPU preprocessing bottleneck being benchmarked here.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS-Cluster
  role: CPU-only spectral clustering tool run with default configuration to measure
    its preprocessing-vs-clustering runtime share
- name: spectra-cluster
  role: CPU-only spectral clustering tool run with default configuration to measure
    its preprocessing-vs-clustering runtime share
- name: MaRaCluster
  role: CPU-only spectral clustering tool run with default configuration to measure
    its preprocessing-vs-clustering runtime share
- name: ThermoRawFileParser
  role: Converts raw PRIDE files to MGF format, the required input format for the
    clustering tools
- name: HyperSpec
  role: Reference GPU-accelerated baseline whose optimized preprocessing modules reduce
    the preprocessing bottleneck quantified by this skill
  repo: https://github.com/wh-xu/Hyper-Spec
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:45bb1de29c02bc5acd078e2621c5e59146dd87050db96559e7a741a7b4327aed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - MS-Cluster
  - MaRaCluster
  - ThermoRawFileParser
  - spectra-cluster
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-clustering-tool-configuration

## Summary

Configure and run state-of-the-art spectral clustering tools (MS-Cluster, spectra-cluster, MaRaCluster) on MGF-formatted MS/MS data to measure stage-wise runtime profiles, with spectrum preprocessing benchmarked against its 20-90% share of overall runtime.

## When to use

Use when profiling the runtime breakdown of CPU-only spectral clustering tools on large MS/MS data sets (e.g. PXD000561 MGF files) to quantify the preprocessing bottleneck before benchmarking a faster method such as HyperSpec.

## When NOT to use

- Input spectra are already preprocessed consensus spectra rather than raw MGF peak files — the bottleneck being profiled exists in raw-file preprocessing, not consensus handling.
- The goal is clustering quality comparison rather than stage-wise runtime profiling — this skill measures time distribution, not incorrect clustering ratio or completeness.
- The target tool is GPU-accelerated (e.g. HyperSpec) — its preprocessing modules are optimized to bypass the CPU preprocessing bottleneck being benchmarked here.

## Inputs

- Raw MS/MS data files downloaded from PRIDE (e.g. PXD000561)
- MGF-formatted peak files (converted from raw via ThermoRawFileParser)

## Outputs

- Per-tool stage-wise timing measurements (preprocessing vs. full clustering wall-clock)
- Preprocessing runtime share per tool (fraction of overall runtime)
- runtime_profile.csv assembling per-tool timing results

## How to apply

Convert raw PRIDE files (e.g. PXD000561) to MGF with ThermoRawFileParser, then run MS-Cluster, spectra-cluster, and MaRaCluster with default configurations on the same MGF inputs. Time the preprocessing stage and the full clustering procedure separately using the Linux wall clock command. Compute preprocessing runtime share as the ratio of preprocessing time to total wall clock time for each CPU-only tool. The article reports this share falls in the 20-90% range for several state-of-the-art tools, which serves as the validation envelope. Only charge states 2 and 3 are clustered in HyperSpec's comparable setup, and CPU-only tools like MS-Cluster, spectra-cluster, and MaRaCluster 'run on CPU hardware only, which lacks massive parallelism', so expect hours-scale runtimes (8-30 h for 25 million spectra).

## Related tools

- **MS-Cluster** (CPU-only spectral clustering tool run with default configuration to measure its preprocessing-vs-clustering runtime share)
- **spectra-cluster** (CPU-only spectral clustering tool run with default configuration to measure its preprocessing-vs-clustering runtime share)
- **MaRaCluster** (CPU-only spectral clustering tool run with default configuration to measure its preprocessing-vs-clustering runtime share)
- **ThermoRawFileParser** (Converts raw PRIDE files to MGF format, the required input format for the clustering tools)
- **HyperSpec** (Reference GPU-accelerated baseline whose optimized preprocessing modules reduce the preprocessing bottleneck quantified by this skill) — https://github.com/wh-xu/Hyper-Spec

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- Measured preprocessing runtime share for each CPU-only tool (MS-Cluster, spectra-cluster, MaRaCluster) falls within the reported 20-90% range.
- Per-tool timings for preprocessing and full clustering are assembled into runtime_profile.csv with separate wall-clock measurements.
- The same MGF input files (converted from PXD000561 raw files) are used for all three tools so runtime shares are directly comparable.
- All three tools run with their default configurations, matching the paper's profiling protocol.
- Wall-clock timing is captured for both the preprocessing stage and the total run via the Linux wall clock command.

## Limitations

- CPU-only clustering of a 25-million-spectra draft human proteome data set (131 GB) can take 8-30 h, so profiling requires significant compute time on large data sets.
- Fast clustering tools (msCRUSH, falcon, ClusterSheep) exhibit reduced clustering quality compared to MaRaCluster and msCRUSH, so runtime profiles do not transfer across quality tiers.
- The 20-90% preprocessing share is reported only for 'several state-of-the-art spectral clustering tools'; new tools may fall outside this envelope.

## Evidence

- [methods] Spectrum preprocessing contributes 20-90% of overall runtime in state-of-the-art spectral clustering tools: "This is one of the bottlenecks during spectral clustering, contributing 20–90% of the overall runtime for several state-of-the-art spectral clustering tools"
- [intro] Existing clustering tools run on CPU only and are extremely slow on large data sets: "the clustering speed of MS-Cluster, spectra-cluster, and MaRaCluster can be extremely slow on large data sets as they run on CPU hardware only"
- [abstract] HyperSpec includes optimized preprocessing modules targeting this bottleneck: "HyperSpec includes optimized data preprocessing modules to reduce the spectrum preprocessing time, which is a critical bottleneck during spectral clustering"
- [readme] MGF is the required input format for HyperSpec clustering runs: "HyperSpec supports running using the command line and takes MGF peak files as input and exports the clustering result as a CSV file"
