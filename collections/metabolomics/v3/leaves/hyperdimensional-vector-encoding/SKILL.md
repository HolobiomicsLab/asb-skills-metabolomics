---
name: hyperdimensional-vector-encoding
description: Use when preprocessed, quantized peak m/z and intensity vectors must
  be converted into fixed-length binary hypervectors prior to Hamming-distance-based
  clustering of large MS/MS data sets.
when_to_use_negative:
- The spectra have not yet been preprocessed and bucket-divided (peaks not quantized/normalized)
  — encoding expects quantized m/z and intensity vectors per bucket
- The downstream analysis requires real-valued spectral similarity rather than binary
  Hamming distances
- 'Clustering must preserve exact spectrum distances: HDC encoding trades some information
  for speed, and the authors note an optimized distance function for comparing spectrum
  HVs is not yet derived'
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Implements the level-ID HD encoding module with bit-packing and GPU-batched
    encoding of spectra into binary hypervectors
  repo: https://github.com/wh-xu/Hyper-Spec
- name: graphics processing units
  role: Executes batched parallel HD encoding via CUDA kernels (Numba/CuPy) with overlapped
    hypervector data transfer
- name: Python
  role: Language in which the fast GPU kernels for binary HDC encoding are developed
- name: CUDA
  role: GPU compute platform used for the batched encoding kernels
- name: CuPy
  role: CUDA-backed array library used for the GPU parallel encoding implementation
- name: Numba
  role: JIT compilation framework used for the GPU parallel encoding implementation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:1733f1b9367bd1ee0f1ff5d959a71aa5180ec67388de4b17ac8ade58deb40fb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - CUDA
  - CuPy
  - HyperSpec
  - Numba
  - Python
  - graphics processing units
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hyperdimensional-vector-encoding

## Summary

Encodes preprocessed MS/MS spectra as binary hypervectors (HVs) of configurable dimension D using level-ID hyperdimensional computing, bit-packed into 32-bit integers for an 8-fold memory reduction, with GPU (CUDA/Numba/CuPy) batched encoding in HyperSpec.

## When to use

Use when preprocessed, quantized peak m/z and intensity vectors must be converted into fixed-length binary hypervectors prior to Hamming-distance-based clustering of large MS/MS data sets.

## When NOT to use

- The spectra have not yet been preprocessed and bucket-divided (peaks not quantized/normalized) — encoding expects quantized m/z and intensity vectors per bucket
- The downstream analysis requires real-valued spectral similarity rather than binary Hamming distances
- Clustering must preserve exact spectrum distances: HDC encoding trades some information for speed, and the authors note an optimized distance function for comparing spectrum HVs is not yet derived

## Inputs

- Preprocessed spectra per bucket: quantized peak m/z vectors and peak intensity vectors (L2-normalized, ≤50 highest-intensity peaks retained)
- Encoding parameters: HD dimension D (default 2048), quantization level Q (default 16), ID flip factor f (maximum peak index range)
- GPU environment: NVIDIA GPU with CUDA (tested on GTX 1080Ti and RTX 3090)

## Outputs

- Bit-packed binary spectrum hypervectors stored as 32-bit integer arrays of length ⌈D/32⌉ per spectrum, entries in {0,1}
- Checkpoint file containing the encoded hypervectors

## How to apply

Load the quantized peak m/z and peak intensity vectors for each bucket produced by preprocessing and bucket division, then stochastically generate the ID hypervector set I (the i-th HV flips D/f bits, where f is the maximum peak index range) and the level hypervector set L (the i-th HV flips D/Q bits, Q = intensity quantization levels, default 16). For each spectrum, fetch I_i for each peak m/z and L_j for each peak intensity, pointwise XOR them (I_i ⊕ L_j), and reduce all XOR results with a pointwise majority function to obtain the D-dimensional binary spectrum hypervector (default D = 2048). Bit-pack each binary HV into 32-bit integers so it is stored as an integer array of length ⌈D/32⌉, reducing memory requirements 8-fold versus byte storage. Transfer the encoding hypervectors to GPU memory and run batched CUDA encoding (Numba/CuPy, default batch size 5000) so that hypervector data transfer overlaps with a batch of spectra being processed. Save the encoded bit-packed HVs to a checkpoint file.

## Related tools

- **HyperSpec** (Implements the level-ID HD encoding module with bit-packing and GPU-batched encoding of spectra into binary hypervectors) — https://github.com/wh-xu/Hyper-Spec
- **graphics processing units** (Executes batched parallel HD encoding via CUDA kernels (Numba/CuPy) with overlapped hypervector data transfer)
- **Python** (Language in which the fast GPU kernels for binary HDC encoding are developed)
- **CUDA** (GPU compute platform used for the batched encoding kernels)
- **CuPy** (CUDA-backed array library used for the GPU parallel encoding implementation)
- **Numba** (JIT compilation framework used for the GPU parallel encoding implementation)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --hd_dim 2048 --hd_Q 16 --batch_size 5000 --checkpoint hv_checkpoint.pkl
```

## Evaluation signals

- Each encoded spectrum must be a binary vector of D bits (entries ∈ {0,1}) stored in ⌈D/32⌉ 32-bit integers, confirming 8-fold compression versus byte storage
- With default D = 2048, each HV must occupy exactly 64 32-bit integers; increasing D from 128 to 8192 should decrease both the incorrect clustering ratio and the clustered spectra ratio
- HD encoding plus distance computation should yield the reported 3.1–12.5x runtime/compression gains relative to alternative representations
- The checkpoint file must contain bit-packed HVs resolvable back to the source spectra (bucket, precursor_charge, precursor_mz, identifier, scan ordering)
- Encoding batch size defaults to 5000; GPU memory transfer should overlap with in-flight batch processing rather than serialize

## Limitations

- Multi-GPU mode is not yet implemented; only single-GPU encoding is available (proposed as future work with near-linear speedup)
- An optimized distance function for comparing spectrum HVs has not been derived, leaving potential clustering-quality gains unrealized
- As D increases, DBSCAN completeness decreases (0.8979 at low D to 0.8615 at high D) even though hierarchical clustering completeness improves, so the encoding dimension interacts with the downstream clustering algorithm
- Larger data sets such as PXD000561 require a high-memory GPU (RTX 3090); a GTX 1080Ti suffices only for smaller data sets
- Bit-packing assumes D divisible or handled via ceil to 32-bit words; HV dimension below ~1000 loses the ultra-high-dimensional regime HyperSpec relies on

## Evidence

- [methods] Stochastically generate the ID hypervector set I and level hypervector set L: [evidence span withheld — anchor status: not_found]
- [methods] pointwise XOR and majority reduction: [evidence span withheld — anchor status: not_found]
- [methods] 8-fold memory reduction via bit-packing: [evidence span withheld — anchor status: not_found]
- [methods] HV storage structure: "HVs are stored in a bit-packed data structure, where every 32 bits of a HV h are packed into a 32-bit integer"
- [readme] GPU encoding module in README: "HyperSpec first encodes the processed spectra into binary hypervector (HV) with ultra-high dimension (>1000) based on level-id encoding method. The encoding module is implemented and optimized for GPU"
- [intro] HDC encoding rationale: "HDC instead encodes spectra
into binary high-dimensional vectors, called hypervectors (HVs)"
- [results] effect of HV dimension on clustering: "as the HV dimension D increases, the
incorrect clustering ratio and the clustered spectra ratio for both
clustering algorithms decreased"
