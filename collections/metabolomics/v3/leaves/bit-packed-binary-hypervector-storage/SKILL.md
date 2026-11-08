---
name: bit-packed-binary-hypervector-storage
description: Use when MS/MS spectra (MGF peak files with quantized peak m/z and L2-normalized
  intensities) must be represented compactly as binary hypervectors for fast distance-based
  clustering, or when HV memory footprint dominates storage.
when_to_use_negative:
- Input spectra have not passed HyperSpec preprocessing (precursor-peak removal, <5
  valid peaks or <250 Da mass-range filtering, top-50 peak retention, L2 normalization)
  — encoding raw spectra will produce noisy HVs
- The goal is exact peak-level comparison rather than approximate similarity — binary
  HV encoding trades exactness for the 8-fold compression and GPU speedup
- No GPU/CUDA environment is available and the dataset scale does not warrant HDC
  encoding overhead
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: HyperSpec
  role: Implements the level-ID HDC encoding, bit-packing, GPU batching, and checkpoint
    saving of spectrum HVs
  repo: https://github.com/wh-xu/Hyper-Spec
- name: graphics processing units
  role: Runs the batched parallel HD encoding kernels (Numba/CuPy CUDA) with batch
    size 5000
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00612
    title: 'HyperSpec: Ultrafast Mass Spectra Clustering in Hyperdimensional Space'
  source_content_hash: sha256:7215fea16d338bed08652814eb9a262b540e4f31adc9b3102df9db3cfe7cbe7b
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
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# bit-packed-binary-hypervector-storage

## Summary

Encodes mass spectra as binary hypervectors (default D=2048, quantization Q=16) using HyperSpec's level-ID HDC encoding, then bit-packs each HV into 32-bit integers so each spectrum HV is stored as ⌈D/32⌉ integers — an 8-fold memory reduction versus byte storage.

## When to use

Use when MS/MS spectra (MGF peak files with quantized peak m/z and L2-normalized intensities) must be represented compactly as binary hypervectors for fast distance-based clustering, or when HV memory footprint dominates storage.

## When NOT to use

- Input spectra have not passed HyperSpec preprocessing (precursor-peak removal, <5 valid peaks or <250 Da mass-range filtering, top-50 peak retention, L2 normalization) — encoding raw spectra will produce noisy HVs
- The goal is exact peak-level comparison rather than approximate similarity — binary HV encoding trades exactness for the 8-fold compression and GPU speedup
- No GPU/CUDA environment is available and the dataset scale does not warrant HDC encoding overhead

## Inputs

- Preprocessed spectra per bucket (quantized peak m/z values and peak intensity vectors, intensities L2-normalized to [0,1], at most 50 highest-intensity peaks retained)
- ID hypervector set I and level hypervector set L generated with hd_dim (default 2048), hd_Q (default 16), and hd_id_flip_factor parameters
- CUDA-capable NVIDIA GPU (e.g. GTX 1080Ti / RTX 3090) with Numba/CuPy environment

## Outputs

- Bit-packed binary spectrum hypervectors stored as integer arrays of length ⌈D/32⌉ with entries in {0,1}
- Checkpoint file containing the encoded bit-packed HVs (--checkpoint argument)

## How to apply

After HyperSpec preprocessing and bucket division, stochastically generate the ID hypervector set I, where the i-th ID HV flips D/f bits (f = maximum peak index range) from the preceding HV, and the level HV set L, where the i-th level HV flips D/Q bits (Q = 16 intensity quantization levels default). For each spectrum, XOR the ID HV I_i of each peak m/z with the level HV L_j of its intensity and reduce all XOR results with a pointwise majority function to produce the D-dimensional binary spectrum HV. Bit-pack every 32 bits of the HV into a 32-bit integer, storing each HV as an integer array of length ⌈D/32⌉ (default 64 integers for D=2048), reducing memory 8-fold. Transfer encoding hypervectors to GPU memory and run batched CUDA encoding (Numba/CuPy) with default batch size 5000 so HV data transfer overlaps with encoding of a batch, then save encoded bit-packed HVs to a checkpoint file.

## Related tools

- **HyperSpec** (Implements the level-ID HDC encoding, bit-packing, GPU batching, and checkpoint saving of spectrum HVs) — https://github.com/wh-xu/Hyper-Spec
- **graphics processing units** (Runs the batched parallel HD encoding kernels (Numba/CuPy CUDA) with batch size 5000)

## Examples

```
python src/main.py ~/dataset/ ./output.csv --cluster_alg dbscan --use_gpu_cluster --hd_dim 2048 --hd_Q 16 --batch_size 5000 --checkpoint encoded_hvs.npy
```

## Evaluation signals

- Each encoded spectrum is a compact binary vector of D bits with entries strictly in {0,1}
- Each bit-packed HV occupies exactly ⌈D/32⌉ 32-bit integers (64 for D=2048), confirming 8-fold compression vs byte storage
- Encoded HVs can be reloaded from the checkpoint file and downstream HD distance computation reproduces the reported 3.1–12.5x runtime/compression gains
- Changing hd_dim within 128–8192 shifts clustering behavior consistently (higher D lowers both incorrect clustering ratio and clustered spectra ratio)
- GPU encoding processes batches of ~5000 spectra with HV transfer overlapping computation, matching --batch_size=5000 default

## Limitations

- Multi-GPU mode is not implemented; a single GPU must hold the encoding hypervectors (RTX 3090 memory required for PXD000561-scale data)
- HV dimension trades quality and recall: as D increases, incorrect clustering ratio and clustered spectra ratio both decrease, and DBSCAN completeness fell from 0.8979 to 0.8615
- No optimized HV distance function exists yet; the article proposes deriving one as future work to improve clustering quality

## Evidence

- [methods] Storing binary HVs bit-packed reduces memory requirements 8-fold: [evidence span withheld — anchor status: not_found]
- [methods] Pointwise majority binding of XORed ID/level HVs: "HVs are stored in a bit-packed data structure, where every 32 bits of a HV h are packed into a 32-bit integer"
- [readme] README workflow description: "HyperSpec first encodes the processed spectra into binary hypervector (HV) with ultra-high dimension (>1000) based on level-id encoding method."
- [intro] HDC encoding rationale: "HDC instead encodes spectra into binary high-dimensional vectors, called hypervectors (HVs)."
- [readme] GPU batching default: "--batch_size                The batch size for HD encoding on GPU. (default: 5000)"
