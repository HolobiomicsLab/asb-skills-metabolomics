---
name: cpp-cython-interface-wrapping
description: Use when a Python analysis pipeline needs to call compiled C++ code —
  e.g. an optimized spectrum/MGF parser for MS data — and raw-file parsing dominates
  runtime (peak information takes up over 95% of the data volume in raw files).
when_to_use_negative:
- The bottleneck is not native parsing — e.g. when spectral clustering or distance
  computation dominates and GPU kernels (HyperSpec's HD encoding / Hamming distance)
  are the right optimization target instead.
- The data are already numerical peak arrays or preprocessed spectra; re-wrapping
  a parser would add overhead with no benefit.
- The target environment cannot compile native extensions (no C++14 toolchain, no
  CUDA/NVIDIA GPU setup for the surrounding HyperSpec pipeline, or no Cython available),
  in which case a pure-Python or prebuilt-binary route is safer.
edam_operation: http://edamontology.org/operation_3797
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3314
tools:
- name: HyperSpec
  role: Python library whose optimized spectrum data parser is implemented in C++
    (Boost Spirit X3) and exposed via Cython interfaces, wrapping native parsing speed
    for the Python clustering pipeline
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Python
  role: High-level language the Cython-wrapped parser is called from; Python kernels
    also exploit GPU resources for downstream clustering
- name: Cython
  role: Interface layer that compiles the C++ parser into Python-importable bindings
- name: Numba
  role: JIT-compiles the parallelized preprocessor that consumes the Cython-exposed
    parser's output and vectorizes peak filtering
- name: ThermoRawFileParser
  role: Upstream converter producing MGF inputs for the wrapped parser in the HyperSpec
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:42176b2d63b7664687669018a3a48b74e3b9f47465e348373f29bd1b09552f29
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
  edam_operation: http://edamontology.org/operation_3797
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cpp-cython-interface-wrapping

## Summary

Wrap a high-performance C++ parser (built with the Spirit X3 parser in Boost C++14) into Cython interfaces callable from Python, so a fast MGF spectrum parser can be exposed to a Python pipeline without giving up native parsing speed on large MS files.

## When to use

Use when a Python analysis pipeline needs to call compiled C++ code — e.g. an optimized spectrum/MGF parser for MS data — and raw-file parsing dominates runtime (peak information takes up over 95% of the data volume in raw files).

## When NOT to use

- The bottleneck is not native parsing — e.g. when spectral clustering or distance computation dominates and GPU kernels (HyperSpec's HD encoding / Hamming distance) are the right optimization target instead.
- The data are already numerical peak arrays or preprocessed spectra; re-wrapping a parser would add overhead with no benefit.
- The target environment cannot compile native extensions (no C++14 toolchain, no CUDA/NVIDIA GPU setup for the surrounding HyperSpec pipeline, or no Cython available), in which case a pure-Python or prebuilt-binary route is safer.

## Inputs

- Raw MGF peak files (MGF format MS/MS spectra from the input dataset directory)
- C++14 source implementing the spectrum parser with Boost Spirit X3
- Cython .pyx/.pxd interface definitions bridging C++ types to Python

## Outputs

- Python-callable module exposing the C++ parser (numerical peak arrays per spectrum: peak m/z and intensities)
- Parsed spectrum data handed to the downstream preprocessor (identifier, scan, precursor charge, precursor m/z, retention time, and filtered peak arrays)

## How to apply

Implement the performance-critical component (for HyperSpec, the MGF spectrum data parser that converts raw peak text into numerical peak arrays) in C++ using the Spirit X3 parser from Boost, compiled under C++14. Expose the C++ implementation to Python by writing Cython interface wrappers, so the Python orchestrator can invoke the native parser transparently as part of the parallelized preprocessing pipeline. Complement the Cython-exposed C++ parser with a Numba JIT-compiled parallelized preprocessor that vectorizes peak filtering and normalization (remove precursor-ion peaks and peaks <1% of the base peak, drop spectra with <5 valid peaks or <250 Da m/z range, retain the 50 most intense peaks, L2-normalize intensities to [0,1]). Because parsing is the bottleneck, this C++/Cython split lets HyperSpec distribute file reading across k independent CPU cores (default 6) while keeping per-file parse cost near native speed; README usage shows this exposed parser driven via `python src/main.py` with `--cpu_core_preprocess` set to the desired core count.

## Related tools

- **HyperSpec** (Python library whose optimized spectrum data parser is implemented in C++ (Boost Spirit X3) and exposed via Cython interfaces, wrapping native parsing speed for the Python clustering pipeline) — https://github.com/wh-xu/Hyper-Spec
- **Python** (High-level language the Cython-wrapped parser is called from; Python kernels also exploit GPU resources for downstream clustering)
- **Cython** (Interface layer that compiles the C++ parser into Python-importable bindings)
- **Numba** (JIT-compiles the parallelized preprocessor that consumes the Cython-exposed parser's output and vectorizes peak filtering)
- **ThermoRawFileParser** (Upstream converter producing MGF inputs for the wrapped parser in the HyperSpec workflow)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine
```

## Evaluation signals

- Parsed output yields numerical peak m/z and intensity arrays per MGF spectrum that satisfy the downstream preprocessor's contract (no spectrum with <5 valid peaks or <250 Da m/z range survives; at most 50 peaks retained per spectrum; intensities within [0,1] after L2 normalization).
- Preprocessing time scales with the number of CPU cores set via --cpu_core_preprocess (default 6), and the speed advantage grows for larger data sets (HyperSpec Figure 8a), indicating the native parser — not Python overhead — dominates.
- The parser handles the full input volume: since peak information takes up over 95% of the data volume in raw files, a successful wrap should keep parse time far below what the same logic in pure Python would require, consistent with preprocessing contributing 20–90% of runtime in state-of-the-art tools being reduced.
- Downstream clustering results (e.g. output.csv / parquet with bucket, precursor_mz, identifier, cluster, is_representative) remain consistent before and after the wrap — the interface change must not alter parsed peak content.

## Limitations

- Requires a compiled build path: HyperSpec needs Python 3.8+ with CUDA and an installed NVIDIA GPU (tested on GTX 1080Ti and RTX 3090 on Linux); clustering PXD000561 requires a GTX 3090 with larger memory.
- Cython/C++ compilation adds build complexity — the README recommends Docker (`docker build --no-cache -f ./docker/Dockerfile -t hyper_spec .`) or `sh install.sh` rather than ad-hoc compilation.
- The wrap addresses parsing/preprocessing only; overall runtime is still bounded by the downstream clustering stage (DBSCAN or hierarchical clustering with complete linkage), and clustering large data sets may need GPU DBSCAN (`--use_gpu_cluster`).

## Evidence

- [methods] Optimized C++ parser exposed via Cython: "We implemented an optimized spectrum data parser and a parallelized preprocessor"
- [methods] Parsing is the runtime bottleneck due to peak data volume: "since the peak information takes up over 95% of the data volume in raw files."
- [methods] Multiprocessing distributes parsing across CPU cores: "HyperSpec utilizes multiprocessing to read each file in parallel and distribute the computation to k independent CPU cores"
- [methods] Preprocessing bottleneck context for the wrap: "This is one of the bottlenecks during spectral clustering, contributing 20–90% of the overall runtime for several state-of-the-art spectral clustering tools"
- [methods] Preprocessor vectorized to cut preprocessing time: [evidence span withheld — anchor status: not_found]
- [readme] CLI exposes the wrapped parser via preprocessing core count: "--cpu_core_preprocess       The number of CPU cores used for preprocessing. (default: 6)"
- [readme] HyperSpec is a Python library requiring compiled CUDA environment: "_HyperSpec_ requires `Python 3.8+` with `CUDA` environment. A GPU should be installed properly."
- [readme] Docker-based build route for the compiled parser: "docker build --no-cache -f ./docker/Dockerfile -t hyper_spec ."
- [readme] Input format the wrapped parser consumes: "_HyperSpec_ supports running using the command line and takes `MGF` peak files as input and exports the clustering result as a CSV file"
- [methods] Filter contract the parsed arrays must satisfy downstream: "Second, spectra with <5 valid peaks or with a <250 Da mass range between their minimum and maximum peaks are removed."
- [readme] GPU requirement for the surrounding pipeline: "Clustering for PXD000561 dataset requires GTX 3090 with larger memory"
