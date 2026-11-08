---
name: ms-spectra-clustering-runtime-profiling
description: Decision support for profiling spectrum preprocessing as a runtime bottleneck
  before choosing or optimizing an MS/MS spectral clustering tool; triggered when
  MGF-converted PRIDE data are available and per-stage timings are needed.
when_to_use_negative:
- Input data are already preprocessed or stored in a format other than raw PRIDE files
  (e.g., vendor MGF ready for clustering), so no raw-to-MGF conversion or preprocessing
  share measurement is applicable.
- The goal is to profile a GPU-accelerated tool such as HyperSpec rather than the
  CPU-only reference tools; HyperSpec's optimized preprocessing module makes it a
  different runtime profile.
- Only a single clustering tool is of interest and cross-tool runtime comparison is
  not needed, so the three-tool profiling workflow is unnecessary.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: ThermoRawFileParser
  role: Converts PRIDE raw files to MGF format prior to running the clustering tools
    for timing.
- name: MS-Cluster
  role: CPU spectral clustering tool timed to measure the preprocessing share of overall
    runtime.
- name: spectra-cluster
  role: CPU spectral clustering tool timed to measure the preprocessing share of overall
    runtime.
- name: MaRaCluster
  role: CPU spectral clustering tool timed to measure the preprocessing share of overall
    runtime.
- name: HyperSpec
  role: Reference tool whose optimized preprocessing modules motivated the profiling;
    provided as contrast but not part of the CPU-only timing set.
  repo: https://github.com/wh-xu/Hyper-Spec
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:5ed499fe1b1bf09ee43763d81289ecfde8bc3a4c65c16552718ad22a723e858f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ms-spectra-clustering-runtime-profiling

## Summary

Profiles the stage-wise runtime of state-of-the-art CPU spectral clustering tools (MS-Cluster, spectra-cluster, MaRaCluster) by converting PRIDE raw files to MGF with ThermoRawFileParser, timing preprocessing versus total wall clock, and reporting the preprocessing share as a fraction of overall runtime (reported range: 20–90%).

## When to use

Decision support for profiling spectrum preprocessing as a runtime bottleneck before choosing or optimizing an MS/MS spectral clustering tool; triggered when MGF-converted PRIDE data are available and per-stage timings are needed.

## When NOT to use

- Input data are already preprocessed or stored in a format other than raw PRIDE files (e.g., vendor MGF ready for clustering), so no raw-to-MGF conversion or preprocessing share measurement is applicable.
- The goal is to profile a GPU-accelerated tool such as HyperSpec rather than the CPU-only reference tools; HyperSpec's optimized preprocessing module makes it a different runtime profile.
- Only a single clustering tool is of interest and cross-tool runtime comparison is not needed, so the three-tool profiling workflow is unnecessary.

## Inputs

- PRIDE raw MS/MS files (e.g., PXD000561), converted to MGF
- ThermoRawFileParser for raw-to-MGF conversion
- MS-Cluster, spectra-cluster, and MaRaCluster binaries with default configurations
- Linux wall clock command for per-stage timing

## Outputs

- runtime_profile.csv with per-tool preprocessing time, total wall clock time, and preprocessing share
- Measured preprocessing share per tool (validated against the 20–90% reported range)

## How to apply

Download raw files for the target data set (e.g., PXD000561) from PRIDE and convert them to MGF format using ThermoRawFileParser. Run MS-Cluster, spectra-cluster, and MaRaCluster with their default configurations on the MGF data set, using the Linux wall clock command to time the preprocessing stage and the full clustering procedure separately. Compute the preprocessing runtime share as the ratio of preprocessing time to total wall clock time for each CPU-only tool. Validate that the measured share falls within the reported 20–90% range, then assemble the per-tool timing results into a runtime_profile.csv. The rationale is that spectrum preprocessing is a critical bottleneck: HyperSpec developed optimized preprocessing modules specifically to reduce this stage's time, and peak information occupies over 95% of raw-file data volume, so the preprocessing share determines whether acceleration efforts should target preprocessing or clustering.

## Related tools

- **ThermoRawFileParser** (Converts PRIDE raw files to MGF format prior to running the clustering tools for timing.)
- **MS-Cluster** (CPU spectral clustering tool timed to measure the preprocessing share of overall runtime.)
- **spectra-cluster** (CPU spectral clustering tool timed to measure the preprocessing share of overall runtime.)
- **MaRaCluster** (CPU spectral clustering tool timed to measure the preprocessing share of overall runtime.)
- **HyperSpec** (Reference tool whose optimized preprocessing modules motivated the profiling; provided as contrast but not part of the CPU-only timing set.) — https://github.com/wh-xu/Hyper-Spec

## Examples

```
ThermoRawFileParser --input PXD000561.raw --output PXD000561.mgf; time ms-cluster PXD000561.mgf; time spectra-cluster PXD000561.mgf; time MaRaCluster PXD000561.mgf
```

## Evaluation signals

- Preprocessing time and total wall clock time are recorded separately for each tool (MS-Cluster, spectra-cluster, MaRaCluster).
- Preprocessing share, computed as preprocessing time divided by total wall clock time, falls within the reported 20–90% range for each CPU-only tool.
- runtime_profile.csv contains one row per tool with the measured preprocessing share.
- Raw files are confirmed converted to MGF by ThermoRawFileParser before any clustering run is timed.

## Limitations

- Profiling is limited to CPU-only clustering tools; HyperSpec's GPU-accelerated pipeline is not directly comparable, since its optimized preprocessing module already targets this bottleneck.
- Timing relies on the Linux wall clock command, so runs are sensitive to system load and storage device speed (the HyperSpec README recommends high-performance SSD for the best performance).
- The reported 20–90% range is derived from default configurations of the three reference tools; different parameter choices may shift the preprocessing share.
- Runtime conclusions apply to the draft human proteome scale (≈25 million spectra, 131 GB); smaller data sets may not exhibit the same bottleneck.

## Evidence

- [methods] Preprocessing contributes 20–90% of overall runtime in state-of-the-art spectral clustering tools: "This is one of the bottlenecks during spectral clustering, contributing 20–90% of the overall runtime for several state-of-the-art spectral clustering tools"
- [abstract] HyperSpec includes optimized data preprocessing modules to reduce the spectrum preprocessing time, which is a critical bottleneck during spectral clustering: "HyperSpec includes optimized data preprocessing modules to reduce the spectrum preprocessing time, which is a critical bottleneck during spectral clustering"
- [methods] Peak information occupies over 95% of raw-file data volume, motivating preprocessing optimization: "since the peak information takes up over 95% of the data volume in raw files"
- [intro] Draft human proteome data set: 25 million MS/MS spectra, 131 GB, requiring hours to days of processing time: "a large-scale draft human proteome data set,1 amounting to 25 million MS/MS spectra and 131 GB of MS data, requires
several hours to days of processing time."
- [readme] HyperSpec README recommends SSD storage and documents GPU memory constraints for large datasets: "We recommend using high-performance SSD as the storage device for the best performance."
