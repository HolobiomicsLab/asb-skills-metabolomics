---
name: mass-difference-matrix-construction
description: Use when two MS/MS spectra (query and reference, e.g. MGF/h5py) must
  be encoded as Transformer input for TransExION spectral similarity scoring or post-hoc
  explanation, requiring aligned mass-difference and neutral-loss matrices.
when_to_use_negative:
- Only a single spectrum is available with no reference spectrum — the MDM is inherently
  a pairwise query–reference construction
- You need a simple baseline similarity (Cosine or Modified Cosine) that does not
  use query–reference fragment mass differences
- Spectra lack precursor m/z information, so the neutral-loss variant (precursor minus
  fragment) cannot be computed
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: TransExION
  role: Reference implementation whose mass difference matrix and aligned-matrix conventions
    (300 Da cut-off, 0–99 mass defect encoding, pad, CLS) must be matched
  repo: github.com/banhdzui/TransExION
- name: pyteomics
  role: Parses the deposited MGF training/test spectra keeping rounded m/z and relative-abundance
    intensities prior to MDM construction
- name: h5py
  role: Storage format for converted spectra and pair index files consumed when building
    the matrices
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:d428b302748a5df05c3f1d959f1ab05afde5d5caf1b7d6661f6cd42517a6379d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit
  - TransExION
  - h5py
  - pyteomics
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# mass-difference-matrix-construction

## Summary

Constructs the TransExION input representation for an MS/MS spectrum pair: a mass difference matrix (MDM) of absolute query–reference fragment differences, aligned by nominal mass (columns 0–299 Da) with rounded mass-defect values (0–99) and pad fills, plus a CLS entry. A parallel neutral-loss MDM (precursor minus fragment) is built the same way.

## When to use

Use when two MS/MS spectra (query and reference, e.g. MGF/h5py) must be encoded as Transformer input for TransExION spectral similarity scoring or post-hoc explanation, requiring aligned mass-difference and neutral-loss matrices.

## When NOT to use

- Only a single spectrum is available with no reference spectrum — the MDM is inherently a pairwise query–reference construction
- You need a simple baseline similarity (Cosine or Modified Cosine) that does not use query–reference fragment mass differences
- Spectra lack precursor m/z information, so the neutral-loss variant (precursor minus fragment) cannot be computed

## Inputs

- Query and reference MS/MS spectra in MGF format (e.g. GNPS_MassBank_train.mgf, GNPS_MassBank_test.mgf) or converted h5py files
- Precursor m/z values for each spectrum (needed to build neutral-loss spectra)
- Spectrum pair index file in h5py format (optional, via --pairs)

## Outputs

- Aligned mass difference matrix per spectrum pair (nominal-mass columns 0–299, mass-defect cell values 0–99, pad entries, CLS entry)
- Neutral-loss mass difference matrix and its aligned matrix per spectrum pair
- Optionally .npz/.npy files on disk keyed by spectrum pair for downstream training/testing

## How to apply

For each query–reference fragment pair, compute the absolute mass difference and restrict to values below the 300 Da cut-off; split each difference into its nominal mass (integer part, mapping to columns 0–299) and its mass defect rounded to two decimals multiplied by 100 (values 0–99). Fill each aligned-matrix row so query fragments sit in the nominal-mass column, inserting the pad value wherever a nominal mass is absent, and prepend a special CLS entry at the start so similarity information accumulates in Transformer self-attention. Repeat the identical procedure on the hypothetical neutral-loss spectra, where each fragment is replaced by precursor m/z minus fragment m/z, to obtain the neutral-loss MDM and aligned matrix. Before matrix construction, spectra should be noise-filtered (drop fragments <0.1% intensity or m/z <10 or >1000 Da; keep only the highest peak within a 3-Da sliding window) and stored in h5py via ms_convert_mgf2h5py.py. Validate by independently re-implementing the transformation and checking cell-by-cell exact agreement with the reference implementation on deposited test spectra.

## Related tools

- **TransExION** (Reference implementation whose mass difference matrix and aligned-matrix conventions (300 Da cut-off, 0–99 mass defect encoding, pad, CLS) must be matched) — github.com/banhdzui/TransExION
- **pyteomics** (Parses the deposited MGF training/test spectra keeping rounded m/z and relative-abundance intensities prior to MDM construction)
- **h5py** (Storage format for converted spectra and pair index files consumed when building the matrices)

## Examples

```
python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_test.mgf --out GNPS_MassBank_test.db
```

## Evaluation signals

- Aligned matrices from an independent re-implementation match the reference implementation exactly, cell-by-cell, on the deposited test spectra
- Every non-CLS cell is either a mass-defect value in 0–99 or the pad value — no other entries
- No aligned-matrix column index exceeds 299 (all mass differences below the 300 Da cut-off)
- Neutral-loss matrices reproduce the same procedure with precursor-minus-fragment values and match element-wise on recomputation
- Recomputed matrices from the deposited MGF files match prior runs element-wise within the two-decimal rounding tolerance

## Limitations

- Mass differences at or above 300 Da are discarded by the nominal-mass column cut-off, so large-mass fragment relationships are not encoded
- Mass defects are quantized to two decimals (0–99), losing finer fractional-mass information
- Missing nominal masses are filled with a pad value rather than interpolated, so sparse spectra yield mostly padded rows

## Evidence

- [abstract] mass difference is the core signal: "TransExION detects related fragments between MS/MS spectra through their mass difference"
- [intro] novelty includes mass differences: "The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered"
- [readme] h5py input format requirement: "To work with large scale dataset, the input data files are converted into h5py format."
