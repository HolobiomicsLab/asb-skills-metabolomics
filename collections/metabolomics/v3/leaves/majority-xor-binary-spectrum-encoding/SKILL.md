---
name: majority-xor-binary-spectrum-encoding
description: Use when you have quantized peak m/z and intensity vectors from preprocessed
  MS/MS spectra and need compact binary high-dimensional representations (HVs) for
  fast GPU distance computation in spectral clustering pipelines like HyperSpec.
when_to_use_negative:
- Spectra have not been preprocessed yet — raw spectra must first go through HyperSpec's
  spectrum preprocessing (precursor/low-intensity peak removal, <5-peak/<250 Da range
  filtering, 50-peak cap, L2 normalization) and bucket division before encoding
- You need low-dimensional hashed spectral vectors rather than binary high-dimensional
  hypervectors — the article contrasts HDC encoding with hashing-based methods like
  msCRUSH and falcon
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: HyperSpec
  role: Host library implementing the level-id HD encoding module, GPU-optimized for
    batched encoding of spectra into binary hypervectors
  repo: https://github.com/wh-xu/Hyper-Spec
- name: CUDA (Numba/CuPy)
  role: GPU backend used to perform batched parallel HD encoding so hypervector data
    transfer overlaps with batch processing
- name: Python
  role: Language in which the fast GPU kernels for the binary HDC operations are written
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:fac739cf65a084efef8e075f80d714b20013cbb604d30ef4d1255b332ff8371f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wh-xu/Hyper-Spec
  tools:
  - CUDA (Numba/CuPy)
  - CuPy
  - HyperSpec
  - Python
  - graphics processing units
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# Majority-XOR Binary Spectrum Encoding

## Summary

Encodes preprocessed MS/MS spectra as bit-packed binary hypervectors (default D = 2048, Q = 16) using level-id HD encoding: XOR the ID hypervector of each peak m/z with the level hypervector of its intensity and bind all pairs with a pointwise majority function, then batch-encode on GPU for HyperSpec clustering.

## When to use

Use when you have quantized peak m/z and intensity vectors from preprocessed MS/MS spectra and need compact binary high-dimensional representations (HVs) for fast GPU distance computation in spectral clustering pipelines like HyperSpec.

## When NOT to use

- Spectra have not been preprocessed yet — raw spectra must first go through HyperSpec's spectrum preprocessing (precursor/low-intensity peak removal, <5-peak/<250 Da range filtering, 50-peak cap, L2 normalization) and bucket division before encoding
- You need low-dimensional hashed spectral vectors rather than binary high-dimensional hypervectors — the article contrasts HDC encoding with hashing-based methods like msCRUSH and falcon

## Inputs

- Preprocessed spectra: quantized peak m/z vectors and L2-normalized peak intensity vectors (at most 50 highest-intensity peaks, intensities in [0, 1])
- HD encoding parameters: hypervector dimension D (default 2048), quantization level Q (default 16), ID flip factor f
- Preprocessed spectra organized into buckets from prior bucket division

## Outputs

- Bit-packed binary spectrum hypervectors stored as 32-bit integer arrays of length ⌈D/32⌉ per spectrum
- Encoding hypervector sets I and L
- Checkpoint file containing the encoded HVs of spectra

## How to apply

First stochastically generate the ID hypervector set I and level hypervector set L: the i-th ID hypervector flips D/f bits (f = maximum peak index range) from the preceding one, and the i-th level hypervector flips D/Q bits (default Q = 16) from the preceding one. For each spectrum, fetch I_i for each peak m/z and L_j for each peak intensity, pointwise XOR them (I_i ⊕ L_j), and reduce all XOR results with a pointwise majority function to produce a D-dimensional binary spectrum hypervector. Bit-pack each HV so every 32 bits are stored in a 32-bit integer, yielding an integer array of length ⌈D/32⌉ and reducing HV memory requirements 8-fold. Transfer the encoding hypervectors to GPU memory and run batched CUDA encoding (Numba/CuPy, default batch size 5000, default HD dimension D = 2048) so data transfer overlaps with processing a batch of spectra. Save the encoded bit-packed HVs to a checkpoint file before HD distance computation. All HV entries must remain binary {0,1}.

## Related tools

- **HyperSpec** (Host library implementing the level-id HD encoding module, GPU-optimized for batched encoding of spectra into binary hypervectors) — https://github.com/wh-xu/Hyper-Spec
- **CUDA (Numba/CuPy)** (GPU backend used to perform batched parallel HD encoding so hypervector data transfer overlaps with batch processing)
- **Python** (Language in which the fast GPU kernels for the binary HDC operations are written)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cluster_alg dbscan --use_gpu_cluster --cluster_charges 2 3 --eps=0.2 --batch_size 5000 --hd_dim 2048 --hd_Q 16 --checkpoint ./encoded_hvs.pkl
```

## Evaluation signals

- Each encoded spectrum hypervector is a compact binary vector of D bits stored in ⌈D/32⌉ 32-bit integers — verify the 8-fold compression vs byte storage holds
- All hypervector entries must be binary {0,1}
- With default parameters, each HV has dimension D = 2048 and intensities are quantized at Q = 16 levels
- HD encoding plus distance computation should yield the reported 3.1–12.5x runtime/compression gains
- Checkpoint file contains the encoded HVs for all spectra in each bucket

## Limitations

- Clustering for the largest dataset (PXD000561) requires a GTX 3090 with larger GPU memory; smaller-scale datasets run on GTX 1080Ti
- A multi-GPU mode is not yet implemented in HyperSpec — a single GPU bounds throughput
- The optimized distance function for comparing spectrum HVs has not yet been derived, so clustering quality may improve with future encoding/distance variants

## Evidence

- [methods] Level-id encoding binds ID and level hypervectors via XOR then majority: [evidence span withheld — anchor status: not_found]
- [methods] Bit packing reduces HV storage 8-fold: [evidence span withheld — anchor status: not_found]
- [methods] HVs stored bit-packed, every 32 bits into a 32-bit integer: "HVs are stored in a bit-packed data structure, where every 32 bits of a HV h are packed into a 32-bit integer"
- [intro] HDC encodes spectra into binary high-dimensional hypervectors: "HDC instead encodes spectra into binary high-dimensional vectors, called hypervectors (HVs)."
- [readme] GPU-based encoding module with level-id encoding on high dimension: "HyperSpec first encodes the processed spectra into binary hypervector (HV) with ultra-high dimension (>1000) based on level-id encoding method. The encoding module is implemented and optimized for GPU"
- [readme] Default HD dimension and quantization parameters from CLI: "--hd_dim                    The HD dimension. (default: 2048) --hd_Q                      The HD quantization level. (default: 16)"
