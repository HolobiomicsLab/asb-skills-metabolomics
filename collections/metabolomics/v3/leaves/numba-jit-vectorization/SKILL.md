---
name: numba-jit-vectorization
description: Use when spectrum preprocessing of large MGF-backed MS/MS datasets is
  CPU-bound in Python loops and peak-filtering rules (precursor removal, min-intensity
  cutoffs, top-N peak retention, L2 normalization) can be rewritten as vectorized
  Numba JIT kernels.
when_to_use_negative:
- The workload is not numeric peak-array filtering/normalization (e.g., vendor raw
  files still needing conversion, or the GPU-side HD encoding and clustering stages).
- The dataset is tiny or the kernel has irregular branching that cannot be vectorized,
  so JIT compilation overhead and vectorization gains do not pay off.
- No Python/NumPy environment is available and the code cannot be rewritten against
  a JIT-vectorized array stack.
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Reference implementation whose parallelized preprocessor vectorizes spectrum
    filtering rules across CPU cores
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Numba
  role: JIT-compiles the vectorized preprocessing kernels applied to parsed numerical
    peak arrays
- name: Python
  role: Host language for the JIT-vectorized preprocessing pipeline and multiprocessing
    dispatch
- name: Cython
  role: Interfaces to the C++ Spirit X3 parser that supplies the numerical arrays
    consumed by the preprocessor
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:30fcfb95463161db2d1f49c808c65143c1a2e728947729e8a8280e389d95e39e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# numba-jit-vectorization

## Summary

Vectorize spectrum preprocessing (peak filtering, intensity normalization) as Numba JIT-compiled parallel kernels instead of Python-level loops, following HyperSpec's approach to attacking the 20–90% preprocessing bottleneck in spectral clustering of large-scale MS/MS data.

## When to use

Use when spectrum preprocessing of large MGF-backed MS/MS datasets is CPU-bound in Python loops and peak-filtering rules (precursor removal, min-intensity cutoffs, top-N peak retention, L2 normalization) can be rewritten as vectorized Numba JIT kernels.

## When NOT to use

- The workload is not numeric peak-array filtering/normalization (e.g., vendor raw files still needing conversion, or the GPU-side HD encoding and clustering stages).
- The dataset is tiny or the kernel has irregular branching that cannot be vectorized, so JIT compilation overhead and vectorization gains do not pay off.
- No Python/NumPy environment is available and the code cannot be rewritten against a JIT-vectorized array stack.

## Inputs

- Raw MGF peak files from an input dataset directory
- Parsed numerical peak m/z and intensity arrays (output of the Spirit X3/Cython MGF parser)
- CLI preprocessing parameters: min_intensity, remove_precursor_tol, min_peaks=5, min_mz_range=250, max_peaks_used=50, scaling {off,root,log,rank}
- Number of CPU cores for parallel preprocessing (--cpu_core_preprocess, default 6)

## Outputs

- Preprocessed spectra as filtered peak m/z and intensity arrays with intensities L2-normalized to [0, 1]
- Intermediate files carrying identifier, scan, precursor_charge, precursor_mz, retention_time plus filtered peak arrays, ready for m/z-sorted bucket division and HD encoding
- Preprocessing runtime measurements showing progressive speedup with dataset size

## How to apply

Convert each filtering rule (removal of precursor-ion-related peaks and peaks with <1% intensity relative to the base peak, dropping spectra with <5 valid peaks or <250 Da m/z range, retaining at most the 50 highest-intensity peaks, L2-normalizing intensities to [0, 1]) into array/vectorized operations rather than per-spectrum Python loops, compiling them with Numba JIT. Distribute parsing and preprocessing of the input MGF files across k independent CPU cores with multiprocessing (HyperSpec defaults to 6 cores via the CLI), since parsing is itself the bottleneck — peak information occupies over 95% of raw-file data volume. Feed the JIT preprocessor numerical peak arrays produced by the low-level parser, and write out the preprocessed spectra (identifier, scan, precursor charge, precursor m/z, retention time, filtered/normalized peak arrays) as intermediate files. The rationale is that HyperSpec's parallelized preprocessor reduces preprocessing time by vectorizing the computation, and its speed advantage progressively grows for larger datasets.

## Related tools

- **HyperSpec** (Reference implementation whose parallelized preprocessor vectorizes spectrum filtering rules across CPU cores) — https://github.com/wh-xu/Hyper-Spec
- **Numba** (JIT-compiles the vectorized preprocessing kernels applied to parsed numerical peak arrays)
- **Python** (Host language for the JIT-vectorized preprocessing pipeline and multiprocessing dispatch)
- **Cython** (Interfaces to the C++ Spirit X3 parser that supplies the numerical arrays consumed by the preprocessor)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- No spectrum with fewer than 5 valid peaks or a <250 Da mass range between minimum and maximum peaks survives preprocessing.
- At most 50 peaks with the highest intensities are retained per spectrum, and peak intensities fall within [0, 1] after L2 normalization.
- Measured preprocessing time decreases versus the naive per-spectrum Python implementation, and the speedup grows progressively for larger datasets (cf. Figure 8a).
- Preprocessed metadata records complete bucket, precursor_charge, precursor_mz, identifier, scan, retention_time fields per spectrum.
- Speedup is reproducible when changing the number of CPU cores (--cpu_core_preprocess), confirming parallel distribution across k independent cores.

## Limitations

- The preprocessing speed advantage of the vectorized kernels progressively grows only for larger datasets; small datasets may show modest gains.
- Parallelism is bounded by the k independent CPU cores allotted to preprocessing (default 6 in the HyperSpec CLI).
- Requires a Python 3.8+ environment; Numba JIT compilation adds first-run overhead and kernels must be re-authored whenever filtering rules or scaling options (off/root/log/rank) change.

## Evidence

- [methods] HyperSpec's parallelized preprocessor reduces the preprocessing time by vectorizing the computation.: [evidence span withheld — anchor status: not_found]
- [methods] Multiprocessing across CPU cores: "HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores"
- [abstract] Preprocessing as clustering bottleneck: "HyperSpec includes optimized data preprocessing modules to reduce the spectrum preprocessing time, which is a critical bottleneck during spectral clustering"
- [readme] Default core count in CLI: "The number of CPU cores used for preprocessing. (default: 6)"
