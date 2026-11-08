---
name: multiprocess-cpu-parallel-preprocessing
description: Use when large collections of raw MGF MS/MS spectra must be parsed and
  preprocessed (precursor-peak removal, peak filtering, L2 normalization) before clustering,
  and preprocessing time is a bottleneck.
when_to_use_negative:
- The input is not MGF peak data (e.g., already-preprocessed peak arrays or consensus
  spectra), so the parser and filtering rules would be redundant or inapplicable.
- The goal is the GPU-side clustering itself (HD encoding, Hamming distance computation,
  DBSCAN/hierarchical clustering) rather than CPU-side parsing and spectrum filtering.
- Only a small data set is available where parallel preprocessing offers no advantage;
  the speed benefit grows progressively with larger data sets.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Provides the multiprocessing pipeline, Cython-exposed C++ MGF parser, and
    Numba-parallelized preprocessor invoked by this skill
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Python
  role: Runtime for the multiprocessing and Numba JIT preprocessing kernels
- name: Cython
  role: Interface layer exposing the Boost Spirit X3 C++14 spectrum parser to Python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:13def3085294be54d4a4f42207fd4d0a4635548ba53c467d90ddf11e0ed29b93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - Cython
  - HyperSpec
  - Python
  - ThermoRawFileParser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multiprocess-cpu-parallel-preprocessing

## Summary

HyperSpec's parallel CPU preprocessing stage: MGF files are read and parsed concurrently across k CPU cores (default 6) using an optimized Boost Spirit X3 C++ parser exposed via Cython, then filtered/normalized by a Numba JIT-compiled preprocessor to attack the 20-90% preprocessing bottleneck in spectral clustering.

## When to use

Use when large collections of raw MGF MS/MS spectra must be parsed and preprocessed (precursor-peak removal, peak filtering, L2 normalization) before clustering, and preprocessing time is a bottleneck.

## When NOT to use

- The input is not MGF peak data (e.g., already-preprocessed peak arrays or consensus spectra), so the parser and filtering rules would be redundant or inapplicable.
- The goal is the GPU-side clustering itself (HD encoding, Hamming distance computation, DBSCAN/hierarchical clustering) rather than CPU-side parsing and spectrum filtering.
- Only a small data set is available where parallel preprocessing offers no advantage; the speed benefit grows progressively with larger data sets.

## Inputs

- Raw MGF peak files in an input directory (positional input_filepath)
- CLI preprocessing parameters: --cpu_core_preprocess (default 6), --remove_precursor_tol, --min_intensity, --min_peaks (5), --min_mz_range (250), --max_peaks_used (50), --scaling {off,root,log,rank}

## Outputs

- Intermediate files of preprocessed spectra: identifier, scan, precursor charge, precursor m/z, retention time, and filtered/normalized peak m/z and intensity arrays
- Preprocessed spectra ready to be sorted by ascending precursor m/z and handed to the HD encoding stage

## How to apply

Load the raw MGF files from the input directory and read each file in parallel, distributing parsing across k independent CPU cores with multiprocessing (the CLI exposes this as --cpu_core_preprocess, default 6). Invoke the low-level spectrum data parser, built with the Spirit X3 parser in Boost C++14 and compiled to Cython interfaces, to convert MGF data into numerical peak arrays; parsing is the bottleneck because peak information occupies over 95% of raw-file data volume. Feed the parsed arrays through the Numba JIT-compiled parallelized preprocessor, applying filters in order: remove precursor-related peaks and peaks with <1% intensity of the base peak (remove_precursor_tol, min_intensity); discard spectra with <5 valid peaks or <250 Da mass range (min_peaks=5, min_mz_range=250); retain at most the 50 highest-intensity peaks (max_peaks_used=50) and L2-normalize intensities to [0,1] per the scaling setting. Write the preprocessed spectra (identifier, scan, precursor charge, precursor m/z, retention time, filtered peak m/z and intensity arrays) to intermediate files so spectra can be sorted by ascending precursor m/z for bucket division and HD encoding.

## Related tools

- **HyperSpec** (Provides the multiprocessing pipeline, Cython-exposed C++ MGF parser, and Numba-parallelized preprocessor invoked by this skill) — https://github.com/wh-xu/Hyper-Spec
- **Python** (Runtime for the multiprocessing and Numba JIT preprocessing kernels)
- **Cython** (Interface layer exposing the Boost Spirit X3 C++14 spectrum parser to Python)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- No surviving spectrum has fewer than 5 valid peaks or a mass range below 250 Da between its minimum and maximum peaks.
- Every surviving spectrum retains at most 50 peaks (the highest-intensity ones, max_peaks_used=50).
- All retained peak intensities fall within [0,1] after L2 normalization.
- Precursor-related peaks and peaks below 1% of base peak intensity are absent from the preprocessed peak lists.
- Preprocessing wall time decreases as --cpu_core_preprocess is increased, and the speed advantage grows with larger data sets (Figure 8a behavior).

## Limitations

- The preprocessing speed advantage progressively grows only for larger data sets; gains may be minimal on small inputs.
- Default configuration assumes 6 CPU cores for preprocessing; throughput depends on available cores and storage speed (high-performance SSD recommended in the README).
- Parsing dominates the stage because peak information occupies over 95% of raw-file data volume, so I/O and parser throughput set the lower bound on runtime.

## Evidence

- [methods] multiprocessing across k CPU cores: "HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores"
- [methods] optimized parser and parallel preprocessor: "We implemented an optimized spectrum data parser and a parallelized preprocessor"
- [methods] peak information dominates raw-file volume: "since the peak information takes up over 95% of the data volume in raw files."
- [methods] first filtering rule: "First, peaks related to the precursor ion or with <1% intensity than the base peak intensity are removed."
- [methods] L2 normalization of intensities: "and the peak intensities are normalized to [0, 1] using their L2 norm."
- [readme] CLI knob for preprocessing parallelism: "--cpu_core_preprocess       The number of CPU cores used for preprocessing. (default: 6)"
- [abstract] preprocessing as the critical bottleneck: "HyperSpec includes optimized data preprocessing modules to reduce the spectrum preprocessing time, which is a critical bottleneck during spectral clustering"
