---
name: gpu-accelerated-spectrum-preprocessing
description: Use when large MGF-format MS/MS data sets (tens of GB, millions of spectra)
  must be prepared for spectral clustering and raw preprocessing would otherwise dominate
  total runtime.
when_to_use_negative:
- Spectra are not in MGF format or have not been converted from raw vendor files (e.g.
  Thermo .raw) — HyperSpec takes MGF peak files as input only
- Only a small data set is being clustered where preprocessing is not the bottleneck
  and a CPU-only tool like MaRaCluster already finishes in acceptable time
- No CUDA-capable GPU is available — the HyperSpec pipeline requires Python 3.8+ with
  a properly installed GPU and CUDA environment
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: HyperSpec
  role: Provides the optimized spectrum data parser, parallelized preprocessor, multiprocessing
    orchestration, and GPU HD-encoding kernels that implement this preprocessing skill
  repo: https://github.com/wh-xu/Hyper-Spec
- name: ThermoRawFileParser
  role: Converts raw Thermo MS files to the MGF format that the preprocessing pipeline
    consumes, retaining charge 2 and charge 3 spectra
- name: Python
  role: Runtime for HyperSpec's fast, GPU-tailored kernels that vectorize the preprocessing
    computation
  repo: https://github.com/wh-xu/Hyper-Spec
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:ed66672c66500fb2d0fdf4bb0b687002702e2c6d57d5a23186b2697b5bf67607
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - HyperSpec
  - Python
  - ThermoRawFileParser
  - graphics processing units
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gpu-accelerated-spectrum-preprocessing

## Summary

An optimized, parallelized spectrum preprocessing pipeline (implemented in HyperSpec) that prepares MGF MS/MS spectra for spectral clustering by removing noise peaks, filtering low-quality spectra, and vectorizing computation across multiple CPU cores and GPU encoders, since preprocessing can consume 20–90% of clustering runtime.

## When to use

Use when large MGF-format MS/MS data sets (tens of GB, millions of spectra) must be prepared for spectral clustering and raw preprocessing would otherwise dominate total runtime.

## When NOT to use

- Spectra are not in MGF format or have not been converted from raw vendor files (e.g. Thermo .raw) — HyperSpec takes MGF peak files as input only
- Only a small data set is being clustered where preprocessing is not the bottleneck and a CPU-only tool like MaRaCluster already finishes in acceptable time
- No CUDA-capable GPU is available — the HyperSpec pipeline requires Python 3.8+ with a properly installed GPU and CUDA environment

## Inputs

- Folder of MGF peak files converted from raw MS data (e.g. via ThermoRawFileParser), retaining precursor charge 2 and charge 3 spectra
- CLI parameters: --cpu_core_preprocess (default 6), --batch_size (default 5000), --min_peaks (5), --min_mz_range (250 Da), --min_intensity (1% of base peak), --max_peaks_used (50), --remove_precursor_tol, --scaling {off,root,log,rank}, --hd_dim (2048), --hd_Q (16), --cluster_charges (2 3)
- Linux machine with NVIDIA GPU (GTX 1080Ti for small/medium sets; GTX 3090 required for the 21M-spectrum PXD000561 set), Python 3.8+ with CUDA, high-performance SSD recommended

## Outputs

- Preprocessed, filtered spectra (precursor/low-intensity peaks removed, ≥5 peaks, ≥250 Da range, ≤50 peaks, L2-normalized) sorted into buckets by ascending precursor m/z
- Binary hypervectors (bit-packed, 32 bits packed per 32-bit integer) encoded on GPU, optionally checkpointed via --checkpoint
- Clustering results CSV/parquet with bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, and is_representative columns

## How to apply

Run the HyperSpec pipeline (python src/main.py <mgf_folder> <output.csv>) which orchestrates five stages: spectrum preprocessing, bucket division by ascending precursor m/z, GPU hyperdimensional encoding, HD distance computation, and clustering; preprocessing is the first stage and the target of this skill. HyperSpec reads each MGF file in parallel, distributing the work to k independent CPU cores (--cpu_core_preprocess, default 6) using multiprocessing, an optimized spectrum data parser, and a parallelized preprocessor that vectorizes the computation. Within preprocessing, apply the article's filter chain: remove peaks related to the precursor ion or with <1% intensity relative to the base peak, drop spectra with <5 valid peaks or a <250 Da mass range, retain at most the 50 highest-intensity peaks, and L2-normalize intensities to [0, 1]. Tunable thresholds are exposed as CLI flags (--min_peaks, --min_mz_range, --min_intensity, --max_peaks_used, --remove_precursor_tol, plus --scaling {off,root,log,rank}); the defaults reproduce the paper's settings. Batch the encoded hypervectors on GPU (--batch_size, default 5000; hd_dim=2048, hd_Q=16) so encoding and downstream Hamming-distance computation also run on the GPU, reducing the bottleneck that CPU-only tools suffer. The rationale is that peak information occupies over 95% of raw-file data volume, so vectorized, multi-core preprocessing plus bit-packed storage (8-fold memory reduction) is what allows over 21 million spectra to be preprocessed and clustered in ~24 minutes instead of hours.

## Related tools

- **HyperSpec** (Provides the optimized spectrum data parser, parallelized preprocessor, multiprocessing orchestration, and GPU HD-encoding kernels that implement this preprocessing skill) — https://github.com/wh-xu/Hyper-Spec
- **ThermoRawFileParser** (Converts raw Thermo MS files to the MGF format that the preprocessing pipeline consumes, retaining charge 2 and charge 3 spectra)
- **Python** (Runtime for HyperSpec's fast, GPU-tailored kernels that vectorize the preprocessing computation) — https://github.com/wh-xu/Hyper-Spec

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- The clustering runtime for the draft human proteome data set (131 GB, >21 million spectra, PXD000561) should land on the order of ~24 min end-to-end (preprocessing through clustering), versus the >4 h reported for CPU-only MaRaCluster
- No spectrum in the output should violate the filter rules: every retained spectrum has ≥5 valid peaks, a ≥250 Da mass range between its minimum and maximum peaks, at most 50 peaks, and L2-normalized intensities in [0, 1]
- Precursor-ion-related peaks and peaks with <1% of the base-peak intensity must be absent from the preprocessed spectra
- Spectra should be sorted and grouped into buckets by ascending precursor m/z before encoding, and only charges 2 and 3 should be present when --cluster_charges 2 3 is used
- Output CSV/parquet must contain the columns bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative with one line per spectrum

## Limitations

- HyperSpec requires Python 3.8+ with CUDA and a properly installed GPU; clustering the PXD000561 data set specifically requires a GTX 3090 with larger memory, while GTX 1080Ti suffices only for smaller data sets
- Spectrum preprocessing contributes 20–90% of overall runtime for state-of-the-art clustering tools, so results are sensitive to I/O speed; high-performance SSD storage is recommended for best performance
- Only charges 2 and 3 are clustered by default; other charge states must be handled separately or the flags adjusted
- A multi-GPU mode is not yet implemented in HyperSpec and is proposed only as future work for near-linear speedup

## Evidence

- [methods] Preprocessing throughput on many cores: "HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores"
- [methods] Vectorized preprocessor rationale: [evidence span withheld — anchor status: not_found]
- [methods] Filter chain thresholds: "First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed."
- [methods] Retention and normalization rules: "Third, at most, 50 peaks with the highest intensities are retained"
- [methods] Peak data dominates file volume: "since the peak information takes up over 95% of the data volume in raw files."
- [abstract] End-to-end runtime claim: "HyperSpec produces results with comparable clustering quality as state-of-the-art spectral clustering tools while achieving speedups by orders of magnitude"
- [readme] GPU requirement from README: "_HyperSpec_ requires `Python 3.8+` with `CUDA` environment. A GPU should be installed properly."
- [readme] Concrete invocation from README: "python src/main.py ~/dataset/ ./output.csv  --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine"
