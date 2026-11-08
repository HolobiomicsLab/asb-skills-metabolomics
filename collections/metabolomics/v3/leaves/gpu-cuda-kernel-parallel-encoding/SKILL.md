---
name: gpu-cuda-kernel-parallel-encoding
description: Use when MS/MS spectra must be transformed into compact binary hypervectors
  for clustering at scale, and a CUDA GPU is available to parallelize the level-ID
  encoding and bit-packed HV storage.
when_to_use_negative:
- No CUDA-capable GPU is available — HyperSpec requires a CUDA environment, and the
  encoding module is optimized specifically for GPU execution
- Spectra have not yet been preprocessed (peaks filtered, L2-normalized, and intensities
  quantized to Q levels) — the encoder expects quantized peak m/z and intensity vectors
- The task is CPU-only spectral clustering where peak-list hashing (e.g. msCRUSH LSH)
  suffices; this skill targets GPU-parallel binary HD encoding
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0121
tools:
- name: HyperSpec
  role: Provides the level-ID HD encoding pipeline (ID/level HV generation, XOR-majority
    bundling, bit-packing, checkpointing) invoked with configurable --hd_dim, --hd_Q,
    --batch_size
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Numba
  role: Python CUDA kernel compilation for batched GPU-parallel HD encoding of spectra
    batches
- name: CuPy
  role: GPU array backend used to transfer hypervectors to GPU memory and run batched
    encoding with transfer/compute overlap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:49732ca49652a8012cd1a642e26f5d3c332c1e5409c92f4190714c36125f0534
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# gpu-cuda-kernel-parallel-encoding

## Summary

GPU-accelerated HD encoding of preprocessed MS/MS spectra into bit-packed binary hypervectors (default D=2048, Q=16) using pointwise XOR and majority bundling, with CUDA kernels (Numba/CuPy) overlapping hypervector transfers with batched computation (default batch size 5000).

## When to use

Use when MS/MS spectra must be transformed into compact binary hypervectors for clustering at scale, and a CUDA GPU is available to parallelize the level-ID encoding and bit-packed HV storage.

## When NOT to use

- No CUDA-capable GPU is available — HyperSpec requires a CUDA environment, and the encoding module is optimized specifically for GPU execution
- Spectra have not yet been preprocessed (peaks filtered, L2-normalized, and intensities quantized to Q levels) — the encoder expects quantized peak m/z and intensity vectors
- The task is CPU-only spectral clustering where peak-list hashing (e.g. msCRUSH LSH) suffices; this skill targets GPU-parallel binary HD encoding

## Inputs

- Preprocessed, quantized peak m/z and peak intensity vectors per bucket (from HyperSpec's preprocessing and bucket-division stages)
- Configurable HD parameters: hd_dim (D, default 2048), hd_Q (Q, default 16), hd_id_flip_factor (f)
- CUDA-capable NVIDIA GPU (GTX 1080Ti or 3090 tested) with Python 3.8+ environment

## Outputs

- Bit-packed binary spectrum hypervectors stored as integer arrays of length ⌈D/32⌉ per spectrum
- Checkpoint file containing the encoded HVs (e.g. --checkpoint hv_ckpt)

## How to apply

First load preprocessed spectra (quantized peak m/z and intensity vectors) per bucket, then stochastically generate ID hypervectors I (each i-th HV flips D/f bits, f = max peak index range) and level hypervectors L (each flips D/Q bits, Q default 16). For each spectrum, XOR the ID HV of each peak m/z with the level HV of its intensity and reduce all XOR results by pointwise majority to yield the D-dimensional binary spectrum HV. Bit-pack every 32 bits of the HV into a 32-bit integer so each HV is stored as ⌈D/32⌉ integers, reducing memory 8-fold versus byte storage. Move encoding HVs into GPU memory and run batched CUDA encoding (Numba/CuPy kernels, default batch_size 5000, hd_dim 2048) so data transfer overlaps with the batch being processed, then save the encoded bit-packed HVs to a checkpoint file.

## Related tools

- **HyperSpec** (Provides the level-ID HD encoding pipeline (ID/level HV generation, XOR-majority bundling, bit-packing, checkpointing) invoked with configurable --hd_dim, --hd_Q, --batch_size) — https://github.com/wh-xu/Hyper-Spec
- **Numba** (Python CUDA kernel compilation for batched GPU-parallel HD encoding of spectra batches)
- **CuPy** (GPU array backend used to transfer hypervectors to GPU memory and run batched encoding with transfer/compute overlap)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --batch_size 5000 --hd_dim 2048 --hd_Q 16 --checkpoint hv_ckpt
```

## Evaluation signals

- Every encoded spectrum must be a binary vector with entries in {0,1} of D bits (default D=2048)
- Each bit-packed HV must be stored as ⌈D/32⌉ 32-bit integers, yielding 8-fold compression versus byte storage
- ID hypervectors flip exactly D/f bits and level hypervectors flip exactly D/Q bits (Q=16 default) from the preceding hypervector in their sets
- HD encoding plus distance computation should reproduce the reported 3.1-12.5x runtime/compression gains, with encoded HVs written to the checkpoint file
- GPU encoding must process spectra in batches (default 5000) with hypervector data transfer overlapping batch computation

## Limitations

- Multi-GPU mode is not implemented in HyperSpec; it is proposed as future work for near-linear speedup
- Clustering the large PXD000561 dataset requires a GTX 3090 with larger GPU memory; smaller datasets run on GTX 1080Ti, and other NVIDIA GPUs need further testing
- HyperSpec has only been tested on GTX 1080Ti and GTX 3090 on Linux; an optimized distance function for comparing spectrum HVs is not yet derived

## Evidence

- [methods] bit-packed storage on GPU: "HVs are stored in a bit-packed data structure, where every 32 bits of a HV h are packed into a 32-bit integer"
- [methods] 8-fold memory reduction: "which reduces the memory requirements to store a HV 8-fold."
- [readme] GPU-optimized encoding module: "The encoding module is implemented and optimized for GPU for shorter runtime."
- [readme] system requirements: "HyperSpec requires `Python 3.8+` with `CUDA` environment. A GPU should be installed properly."
