---
name: wall-clock-runtime-measurement
description: Use when benchmarking spectral clustering tools (HyperSpec, MS-Cluster,
  spectra-cluster, MaRaCluster) on MS/MS data such as PXD000561 and you need runtime
  shares per stage or end-to-end wall-clock times.
when_to_use_negative:
- The goal is clustering quality (incorrect clustering ratio, completeness, peptide
  identification from consensus spectra) rather than runtime — quality comparisons
  need different evaluation metrics
- The input is not a large-scale MS/MS clustering workload; profiling 20–90% preprocessing
  shares was established specifically for state-of-the-art spectral clustering tools
  on large datasets
- 'GPU memory is insufficient: the README notes that clustering the PXD000561 dataset
  requires a GTX 3090 with larger memory, so a GTX 1080Ti cannot reproduce the runtime
  numbers on that dataset'
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Tool under test whose end-to-end wall-clock runtime (preprocessing through
    clustering) is measured against CPU-only baselines
  repo: https://github.com/wh-xu/Hyper-Spec
- name: MS-Cluster
  role: CPU-only baseline clustering tool timed per stage to quantify the preprocessing
    runtime share
- name: spectra-cluster
  role: CPU-only baseline clustering tool timed per stage to quantify the preprocessing
    runtime share
- name: MaRaCluster
  role: CPU-only baseline clustering tool timed per stage; its >4 h runtime on PXD000561
    serves as the speedup reference point
- name: ThermoRawFileParser
  role: Converts PRIDE raw files to MGF format as the common input format for all
    timed tools
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:6c2f0f52fd568d4175dbc9fed4d84cc0a0403da468b285e4dc4f6e1d7ee0772e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - MS-Cluster
  - MaRaCluster
  - Python
  - ThermoRawFileParser
  - graphics processing units
  - spectra-cluster
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# wall-clock-runtime-measurement

## Summary

Stage-wise and end-to-end wall-clock profiling of MS/MS spectral clustering pipelines: measuring spectrum preprocessing time versus total clustering runtime with the Linux wall clock to locate bottlenecks and validate speedup claims.

## When to use

Use when benchmarking spectral clustering tools (HyperSpec, MS-Cluster, spectra-cluster, MaRaCluster) on MS/MS data such as PXD000561 and you need runtime shares per stage or end-to-end wall-clock times.

## When NOT to use

- The goal is clustering quality (incorrect clustering ratio, completeness, peptide identification from consensus spectra) rather than runtime — quality comparisons need different evaluation metrics
- The input is not a large-scale MS/MS clustering workload; profiling 20–90% preprocessing shares was established specifically for state-of-the-art spectral clustering tools on large datasets
- GPU memory is insufficient: the README notes that clustering the PXD000561 dataset requires a GTX 3090 with larger memory, so a GTX 1080Ti cannot reproduce the runtime numbers on that dataset

## Inputs

- Raw vendor MS/MS files for dataset PXD000561 downloaded from the PRIDE database (131 GB, >21 million spectra)
- MGF-format peak files converted from raw data via ThermoRawFileParser (charges 2 and 3 retained)
- Clustering tool executables: MS-Cluster, spectra-cluster, MaRaCluster (CPU-only baselines) and HyperSpec (GPU-accelerated)

## Outputs

- Per-tool preprocessing share of total runtime (ratio of preprocessing time to total wall-clock time)
- runtime_profile.csv with per-tool stage-wise timings for MS-Cluster, spectra-cluster, and MaRaCluster
- runtime_report.csv with HyperSpec end-to-end runtime in minutes

## How to apply

Download the PXD000561 raw files from PRIDE, convert them to MGF with ThermoRawFileParser (retaining precursor charge 2 and 3 spectra), then run each tool with default configurations while timing the preprocessing stage separately from the full clustering procedure using the Linux wall clock command. For HyperSpec, start timing at the beginning of parallel spectrum preprocessing and stop at the finish of clustering, excluding consensus spectrum generation. Compute the preprocessing runtime share as preprocessing time divided by total wall-clock time per tool, and write per-tool timings to a runtime_profile.csv (or runtime_report.csv for HyperSpec end-to-end). Validate by checking the measured preprocessing share falls within the reported 20–90% range and that HyperSpec's runtime is on the order of ~24 min for >21 million spectra, far below the >4 h reference for MaRaCluster.

## Related tools

- **HyperSpec** (Tool under test whose end-to-end wall-clock runtime (preprocessing through clustering) is measured against CPU-only baselines) — https://github.com/wh-xu/Hyper-Spec
- **MS-Cluster** (CPU-only baseline clustering tool timed per stage to quantify the preprocessing runtime share)
- **spectra-cluster** (CPU-only baseline clustering tool timed per stage to quantify the preprocessing runtime share)
- **MaRaCluster** (CPU-only baseline clustering tool timed per stage; its >4 h runtime on PXD000561 serves as the speedup reference point)
- **ThermoRawFileParser** (Converts PRIDE raw files to MGF format as the common input format for all timed tools)

## Examples

```
time python src/main.py ./PXD000561_mgf/ ./output.csv --use_gpu_cluster --cluster_charges 2 3 --eps 0.4 --cluster_alg hc_complete
```

## Evaluation signals

- Measured preprocessing share for MS-Cluster, spectra-cluster, and MaRaCluster falls within the reported 20–90% range of overall runtime
- HyperSpec end-to-end runtime is on the order of ~24 min for over 21 million spectra and far below MaRaCluster's >4 h reference runtime
- Timing window for HyperSpec starts at spectrum preprocessing and ends at clustering completion, excluding consensus spectrum generation
- Timing is measured consistently across tools using the Linux wall clock command with default tool configurations on the same MGF-converted PXD000561 data

## Limitations

- Preprocessing runtime share depends on the specific tools tested; the 20–90% figure is stated only for 'several state-of-the-art spectral clustering tools' and may not generalize to other pipelines
- Runtime results are hardware-dependent: HyperSpec requires a CUDA GPU and was tested on GTX 1080Ti and GTX 3090 on Linux; high-performance SSD storage is recommended for best performance
- Consensus spectrum generation is excluded from the measured HyperSpec runtime, so wall-clock numbers are not directly comparable to tools timed including post-processing

## Evidence

- [methods] Spectrum preprocessing contributes 20-90% of overall runtime in state-of-the-art spectral clustering tools: "This is one of the bottlenecks during spectral clustering, contributing 20–90% of the overall runtime for several state-of-the-art spectral clustering tools"
- [abstract] HyperSpec reduces clustering runtime of over 21 million spectra from 4 h to 24 min: [evidence span withheld — anchor status: not_found]
- [readme] README runtime claim on draft human proteome dataset: "HyperSpec shortens the runtime on the draft human proteome dataset with 25 million spectra from a few hours to <15 minutes."
- [intro] Large-scale draft human proteome dataset baseline runtime: "amounting to 25 million MS/MS spectra and 131 GB of MS data, requires several hours to days of processing time"
