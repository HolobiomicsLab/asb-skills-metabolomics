---
name: mgf-spectrum-parsing
description: Use when clustering large-scale MS/MS data where raw MGF files must be
  converted into numeric peak arrays and preprocessed, since peak information occupies
  over 95% of raw-file data volume and preprocessing is a critical bottleneck contributing
  20-90% of overall runtime.
when_to_use_negative:
- Input spectra are already preprocessed numeric peak arrays rather than raw MGF files
  - the parser targets MGF text formats
- Running on CPU-only environments without GPU-backed clustering downstream, since
  HyperSpec requires Python 3.8+ with CUDA and a properly installed NVIDIA GPU
- Storage is slow (e.g. non-SSD), since the README recommends high-performance SSD
  for best performance and preprocessing I/O dominates for large data sets
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: HyperSpec
  role: Provides the multiprocessing MGF reader, Spirit X3 C++ spectrum data parser
    exposed via Cython, and the Numba-JIT parallelized preprocessor implementing the
    ordered filtering rules
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Cython
  role: Compiles the Spirit X3 C++14 parser to Python interfaces so MGF text can be
    converted to numerical peak arrays
- name: Numba
  role: JIT-compiles the vectorized preprocessor kernels that apply peak/spectrum
    filters and L2 normalization
- name: ThermoRawFileParser
  role: Used to convert raw files to MGF upstream of parsing in the evaluation workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:9a121d2fc2f0cbe947039d2dd1d2eb6872cd456fb2faa9be7608a3c905bbc0bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - Cython
  - HyperSpec
  - Numba
  - Python
  - ThermoRawFileParser
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# mgf-spectrum-parsing

## Summary

Optimized parallel parsing of MGF peak files followed by vectorized spectrum preprocessing (precursor/low-intensity peak removal, min-peaks and m/z-range filtering, top-50 peak retention, L2 normalization) to prepare MS/MS spectra for hyperdimensional clustering.

## When to use

Use when clustering large-scale MS/MS data where raw MGF files must be converted into numeric peak arrays and preprocessed, since peak information occupies over 95% of raw-file data volume and preprocessing is a critical bottleneck contributing 20-90% of overall runtime.

## When NOT to use

- Input spectra are already preprocessed numeric peak arrays rather than raw MGF files - the parser targets MGF text formats
- Running on CPU-only environments without GPU-backed clustering downstream, since HyperSpec requires Python 3.8+ with CUDA and a properly installed NVIDIA GPU
- Storage is slow (e.g. non-SSD), since the README recommends high-performance SSD for best performance and preprocessing I/O dominates for large data sets

## Inputs

- Folder of MGF peak files containing raw MS/MS spectra (input_filepath positional argument)
- CLI configuration parameters: remove_precursor_tol, min_intensity, min_peaks, min_mz_range, max_peaks_used, scaling (off/root/log/rank)
- Number of CPU cores for preprocessing (cpu_core_preprocess, default 6)

## Outputs

- Intermediate preprocessed spectra files with per-spectrum identifier, scan, precursor charge, precursor m/z, retention time, and filtered/normalized peak m/z and intensity arrays
- Spectra eligible for sorting by ascending precursor m/z and bucket division before HD encoding

## How to apply

Distribute MGF file reading across k independent CPU cores via multiprocessing (default 6 cores via --cpu_core_preprocess), invoking the Spirit X3-based C++14 parser compiled to Cython to convert MGF peaks into numerical arrays. Apply the parallelized Numba-JIT preprocessor in three ordered filtering steps: first remove peaks related to the precursor ion or with <1% intensity relative to the base peak (--remove_precursor_tol, --min_intensity); second discard spectra with <5 valid peaks (--min_peaks) or a <250 Da mass range between minimum and maximum peaks (--min_mz_range); third retain at most the 50 highest-intensity peaks (--max_peaks_used) and normalize intensities to [0, 1] via L2 norm (--scaling off/root/log/rank). Write preprocessed spectra (identifier, scan, precursor charge, precursor m/z, retention time, and filtered m/z and intensity arrays) to intermediate files so they can be sorted by ascending precursor m/z for bucket division and handed to HD encoding. The rationale is vectorization: the parallelized preprocessor reduces preprocessing time by vectorizing computation, and preprocessing speed advantage grows for larger data sets.

## Related tools

- **HyperSpec** (Provides the multiprocessing MGF reader, Spirit X3 C++ spectrum data parser exposed via Cython, and the Numba-JIT parallelized preprocessor implementing the ordered filtering rules) — https://github.com/wh-xu/Hyper-Spec
- **Cython** (Compiles the Spirit X3 C++14 parser to Python interfaces so MGF text can be converted to numerical peak arrays)
- **Numba** (JIT-compiles the vectorized preprocessor kernels that apply peak/spectrum filters and L2 normalization)
- **ThermoRawFileParser** (Used to convert raw files to MGF upstream of parsing in the evaluation workflow)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- Every surviving spectrum has at least 5 valid peaks (min_peaks=5)
- Every surviving spectrum has a mass range between its minimum and maximum peaks of at least 250 Da (min_mz_range=250)
- At most 50 peaks are retained per spectrum (max_peaks_used=50)
- Peak intensities fall within [0, 1] after L2 normalization
- No peaks related to the precursor ion or below 1% of base peak intensity remain
- Preprocessed spectra include complete metadata (identifier, scan, precursor_charge, precursor_mz, retention_time) plus peak arrays, matching the schema later exported to the parquet clustering results

## Limitations

- Spectrum parsing is the bottleneck because peak information takes up over 95% of the data volume in raw files
- Preprocessing contributes 20-90% of overall runtime in state-of-the-art spectral clustering tools, so parallelization and vectorization are essential but the speedup requires multiprocessing with multiple CPU cores
- HyperSpec has only been tested on NVIDIA GPUs (GTX 1080Ti, GTX 3090) on Linux; other GPUs need further testing, and PXD000561 requires a GTX 3090 with larger memory
- The CLI flags min_peaks, min_mz_range, min_mz, max_mz, remove_precursor_tol, min_intensity, max_peaks_used, and scaling exist but the README does not document their exact defaults beyond what the paper states

## Evidence

- [readme] Multiprocessing MGF reading and distribution across CPU cores: "--cpu_core_preprocess       The number of CPU cores used for preprocessing. (default: 6)"
- [readme] MGF input format and CSV/parquet output: "_HyperSpec_ supports running using the command line and takes `MGF` peak files as input and exports the clustering result as a CSV file"
- [abstract] Bottleneck rationale for optimized preprocessing: "HyperSpec includes optimized data preprocessing modules to reduce the spectrum preprocessing time, which is a critical bottleneck during spectral clustering"
- [methods] Peak information dominates raw-file data volume: "since the peak information takes up over 95% of the data volume in raw files."
- [methods] Parallelized preprocessor rationale: [evidence span withheld — anchor status: not_found]
- [methods] Ordered filtering step 1: remove precursor and low-intensity peaks: "First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed."
- [methods] Ordered filtering step 2: min peaks and m/z range: "Second, spectra with <5 valid peaks or with a <250 Da mass range between their minimum and maximum peaks are removed."
- [methods] Ordered filtering step 3: top-50 retention and normalization: "Third, at most, 50 peaks with the highest intensities are retained"
