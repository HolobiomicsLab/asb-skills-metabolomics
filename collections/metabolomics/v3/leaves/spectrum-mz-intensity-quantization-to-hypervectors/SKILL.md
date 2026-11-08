---
name: spectrum-mz-intensity-quantization-to-hypervectors
description: Use when preprocessed MGF spectra (precursor-filtered, L2-normalized,
  ≤50 peaks) must be converted into compact binary representations for fast pairwise
  distance computation on GPU.
when_to_use_negative:
- Input spectra are not yet preprocessed (still contain precursor peaks, low-intensity
  noise, or unnormalized intensities) — encode only after the parallelized preprocessor
  filter chain
- You need full floating-point peak fidelity for downstream peptide identification
  on raw spectra; binary HD encoding trades precision for speed
- A low-dimensional hashing projection is required (e.g. falcon-style); HDC encodes
  into high-dimensional binary vectors instead
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: HyperSpec
  role: Main library providing the GPU-accelerated level-ID HD encoding module, bit-packing,
    and checkpointing
  repo: https://github.com/wh-xu/Hyper-Spec
- name: Python
  role: Host language in which fast CUDA kernels (Numba/CuPy) are developed to exploit
    GPU data parallelism for HD encoding
- name: graphics processing units
  role: Executes batched (5000 spectra default) parallel HD encoding of XOR-bound
    hypervectors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:531cdc5c10fbcce85742874814d63983c06d8924db5a6e69d17139ea50789c19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - CuPy
  - HyperSpec
  - Python
  - graphics processing units
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# spectrum-mz-intensity-quantization-to-hypervectors

## Summary

Encodes preprocessed MS/MS spectra as D-dimensional binary hypervectors via level-ID hyperdimensional computing, then bit-packs each HV into 32-bit integers for GPU-resident Hamming-distance clustering.

## When to use

Use when preprocessed MGF spectra (precursor-filtered, L2-normalized, ≤50 peaks) must be converted into compact binary representations for fast pairwise distance computation on GPU.

## When NOT to use

- Input spectra are not yet preprocessed (still contain precursor peaks, low-intensity noise, or unnormalized intensities) — encode only after the parallelized preprocessor filter chain
- You need full floating-point peak fidelity for downstream peptide identification on raw spectra; binary HD encoding trades precision for speed
- A low-dimensional hashing projection is required (e.g. falcon-style); HDC encodes into high-dimensional binary vectors instead

## Inputs

- Preprocessed MGF spectra (precursor and <1% base-intensity peaks removed, ≥5 peaks and ≥250 Da m/z range, ≤50 highest-intensity peaks retained, L2-normalized, sorted by precursor m/z)
- ID hypervector set I (dimension D, bit-flip factor f = maximum peak index range)
- Level hypervector set L (dimension D, quantization levels Q = 16 default)

## Outputs

- Bit-packed binary spectrum hypervectors: integer array of length ⌈D/32⌉ per spectrum, entries {0,1} in binary form
- Checkpoint file with encoded HVs of spectra
- Bucket-level hypervector arrays ready for GPU Hamming distance computation

## How to apply

Build stochastic ID hypervectors I_i where each successive hypervector flips D/f bits (f = peak m/z index range) and level hypervectors L_j that flip D/Q bits (Q = intensity quantization levels, default 16); D defaults to 2048. For each spectrum, fetch I_i per peak m/z and L_j per peak intensity, pointwise-XOR the pairs (I_i ⊕ L_j), and bind all XOR products with a pointwise majority vote to yield one binary spectrum hypervector. Bit-pack each binary HV into 32-bit integers, storing ⌈D/32⌉ ints per spectrum so memory drops 8-fold versus byte storage. Transfer encoding hypervectors to GPU memory and batch-encode (batch_size = 5000 default) with CUDA kernels via Numba/CuPy so data transfer overlaps with ongoing batches. Checkpoint the encoded bit-packed HVs to disk so downstream Hamming-distance and DBSCAN/hierarchical-clustering stages can resume without re-encoding.

## Related tools

- **HyperSpec** (Main library providing the GPU-accelerated level-ID HD encoding module, bit-packing, and checkpointing) — https://github.com/wh-xu/Hyper-Spec
- **Python** (Host language in which fast CUDA kernels (Numba/CuPy) are developed to exploit GPU data parallelism for HD encoding)
- **graphics processing units** (Executes batched (5000 spectra default) parallel HD encoding of XOR-bound hypervectors)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cpu_core_preprocess=4 --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --refine --hd_dim 2048 --hd_Q 16 --batch_size 5000
```

## Evaluation signals

- Every encoded spectrum must be a compact binary vector of D=2048 bits stored in ⌈D/32⌉ 32-bit integers — 8-fold compression vs. byte storage
- Encoded HV entries must be strictly binary {0,1}; no non-binary values should survive bit-packing
- HD encoding plus GPU Hamming distance computation must reproduce the paper's 3.1–12.5× runtime/compression gains over non-HD baselines
- Encoded HVs must round-trip through checkpoint files without loss (saved as bit-packed 32-bit integer arrays)
- Increasing Q or D must measurably reduce incorrect clustering ratio and clustered-spectra ratio in downstream clustering

## Limitations

- Encoding fidelity is bounded by quantization level Q (default 16) and dimension D (default 2048); sub-optimal distance function for comparing spectrum HVs remains an open improvement (proposed in discussion)
- Multi-GPU mode is not yet implemented — only single-GPU batched encoding is supported
- PXD000561-scale clustering requires a GTX 3090 with larger memory; GTX 1080Ti suffices only for smaller datasets

## Evidence

- [methods] Stochastic ID and level hypervector generation with XOR + majority binding: [evidence span withheld — anchor status: not_found]
- [methods] Bit-packing rationale: "HVs are stored in a bit-packed data structure, where every 32 bits of a HV h are packed into a 32-bit integer"
- [methods] Memory reduction finding: "which reduces the memory requirements to store a HV 8-fold."
- [readme] Encoding module is GPU-optimized: "_HyperSpec_ first encodes the processed spectra into binary hypervector (HV) with ultra-high dimension (>1000) based on level-id encoding method. The encoding module is implemented and optimized for"
- [readme] Intrinsic data structure / dimension defaults: "--hd_dim                    The HD dimension. (default: 2048)
    --hd_Q                      The HD quantization level. (default: 16)"
