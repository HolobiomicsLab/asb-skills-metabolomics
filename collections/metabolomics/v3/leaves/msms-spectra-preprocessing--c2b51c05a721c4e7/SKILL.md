---
name: msms-spectra-preprocessing
description: Use when preparing raw MGF spectra from PRIDE-style datasets (e.g., PXD000561
  charge 2/3 MS/MS) for spectral clustering and you need reproducible peak filtering,
  intensity normalization, and low-quality spectrum removal.
when_to_use_negative:
- Input is already preprocessed consensus spectra or a spectral library — re-filtering
  would strip valid low-intensity peaks and distort normalized intensities.
- You need exhaustive fragment-peak information for de novo sequencing; the <1% base-peak
  intensity cutoff and 50-peak cap discard most peaks by design for clustering, not
  identification.
- Spectra are charge states other than 2/3 or below the 5-peak/250 Da quality gates
  — they are removed rather than clustered.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: HyperSpec
  role: Provides the optimized Spectrum Data Parser and Parallelized Preprocessor
    (hybrid C++-Python, Numba-JIT/CUDA vectorized) that execute all filtering and
    normalization steps
  repo: https://github.com/wh-xu/Hyper-Spec
- name: ThermoRawFileParser
  role: Converts Thermo raw files downloaded from PRIDE into the MGF format HyperSpec
    consumes
- name: Python
  role: Runtime environment (3.8+ with CUDA) in which the preprocessing kernels and
    Numba-JIT parallelized preprocessor execute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:6679158c89c8b4dd564666f0d2525f8133552a00f2f1e59fa1a75fc874cdcc29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - CuPy
  - HyperSpec
  - Python
  - ThermoRawFileParser
  - cuML
  - fastcluster
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8695652173913043
n_claims: 23
---

# msms-spectra-preprocessing

## Summary

Preprocess MGF-format MS/MS spectra into clean, L2-normalized, ≤50-peak inputs before hypervector encoding and clustering, using HyperSpec's hybrid C++-Python parser and Numba/CUDA-parallelized preprocessor to cut the preprocessing bottleneck that consumes 20-90% of runtime in state-of-the-art clustering tools.

## When to use

Use when preparing raw MGF spectra from PRIDE-style datasets (e.g., PXD000561 charge 2/3 MS/MS) for spectral clustering and you need reproducible peak filtering, intensity normalization, and low-quality spectrum removal.

## When NOT to use

- Input is already preprocessed consensus spectra or a spectral library — re-filtering would strip valid low-intensity peaks and distort normalized intensities.
- You need exhaustive fragment-peak information for de novo sequencing; the <1% base-peak intensity cutoff and 50-peak cap discard most peaks by design for clustering, not identification.
- Spectra are charge states other than 2/3 or below the 5-peak/250 Da quality gates — they are removed rather than clustered.

## Inputs

- Raw MS/MS files downloaded from PRIDE converted to MGF format (e.g., PXD000561)
- MGF peak files folder passed as input_filepath to HyperSpec (python src/main.py)

## Outputs

- Preprocessed, filtered spectra with ≤50 peaks, intensities normalized to [0,1], sorted by ascending precursor m/z
- Compressed parquet metadata recording bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, and is_representative

## How to apply

Convert raw files from PRIDE to MGF (e.g., with ThermoRawFileParser), then run HyperSpec's pipeline: a Spectrum Data Parser reads each file in parallel across k CPU cores, and a Parallelized Preprocessor applies fixed thresholds — remove precursor-related peaks and peaks with <1% of base-peak intensity, drop spectra with <5 valid peaks or a <250 Da mass range, retain at most the 50 most intense peaks, and L2-normalize intensities to [0, 1]. Next, spectra are sorted by ascending precursor m/z so nearby precursor masses land in shared buckets (controlled via --mz_interval and --precursor_tol, default clustering charges 2 and 3). The rationale is that peak information dominates file volume (>95%), so vectorized, Numba-JIT/CUDA preprocessing of bit-packed binary hypervectors (8x memory saving) is where the biggest runtime win lives. Judgment: the preprocessed set should retain only spectra passing the peak-count/mass-range gates, with intensities in [0,1] summing to a unit L2 norm, ready for HD encoding with --hd_dim 2048.

## Related tools

- **HyperSpec** (Provides the optimized Spectrum Data Parser and Parallelized Preprocessor (hybrid C++-Python, Numba-JIT/CUDA vectorized) that execute all filtering and normalization steps) — https://github.com/wh-xu/Hyper-Spec
- **ThermoRawFileParser** (Converts Thermo raw files downloaded from PRIDE into the MGF format HyperSpec consumes)
- **Python** (Runtime environment (3.8+ with CUDA) in which the preprocessing kernels and Numba-JIT parallelized preprocessor execute)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_charges 2 3 --hd_dim 2048 --hd_Q 16 --min_peaks 5 --min_mz_range 250 --min_intensity 0.01 --max_peaks_used 50
```

## Evaluation signals

- Each retained spectrum has at least 5 valid peaks and a ≥250 Da mass range between minimum and maximum peaks.
- All retained spectra have at most 50 peaks, ranked by intensity, with intensities L2-normalized into [0,1].
- Precursor-related peaks and any peaks <1% of base-peak intensity are absent from preprocessed output.
- Output metadata parquet parses cleanly with required columns (bucket, precursor_charge, precursor_mz, identifier, scan, retention_time, cluster, is_representative) and only charges 2/3 present when --cluster_charges 2 3 is set.
- Preprocessing completes without dropping valid charge-2/3 spectra relative to MGF input counts, consistent with 'preprocessing contributes 20-90% of runtime' being reduced via vectorization.

## Limitations

- Preprocessing thresholds are fixed for clustering quality, not peptide identification: removing peaks <1% base-peak intensity and capping at 50 peaks discards information relevant to downstream search engines.
- Spectra with <5 valid peaks or <250 Da mass range are permanently dropped, biasing against sparse low-quality acquisitions.
- The parser uses multiprocessing across k CPU cores, so throughput depends on available cores; GPU clustering of PXD000561-scale data requires a GTX 3090-class GPU with larger memory.

## Evidence

- [methods] First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed.: "First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed."
- [methods] Second, spectra with <5 valid peaks or with a <250 Da mass range between their minimum and maximum peaks are removed.: "Second, spectra with <5 valid peaks or with a <250 Da mass range between their minimum and maximum peaks are removed."
- [methods] Third, at most, 50 peaks with the highest intensities are retained: "Third, at most, 50 peaks with the highest intensities are retained"
- [methods] HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores: "HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores"
- [readme] _HyperSpec_ supports running using the command line and takes `MGF` peak files as input and exports the clustering result as a CSV file: "_HyperSpec_ supports running using the command line and takes `MGF` peak files as input"
