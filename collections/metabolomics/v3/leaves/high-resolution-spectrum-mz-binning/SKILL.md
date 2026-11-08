---
name: high-resolution-spectrum-mz-binning
description: Use when peak files (mzML, mzXML, or MGF) of MS/MS spectra must be vectorized
  for fast similarity searching or clustering, and exhaustive pairwise spectrum comparison
  is infeasible (e.g. millions of spectra).
when_to_use_negative:
- Input is already a feature table, spectral library embedding, or other fixed-length
  vector representation — re-binning and hashing would destroy existing intensity
  information
- Spectra are so low-quality that preprocessing discards nearly all of them (<5 peaks
  or <250 Da m/z range) — no reliable vector can be formed
- Metabolomics or top-down data analyzed with falcon's bottom-up proteomics defaults
  — the min_peaks/min_mz_range and m/z window settings (default 101–500 m/z) likely
  need adjustment before binning
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Implements the binning + feature hashing pipeline that converts spectra to
    low-dimensional vectors; open source under the BSD license
  repo: https://github.com/bittremieux/falcon
- name: spectrum_utils
  role: Used for spectrum preprocessing (peak filtering, intensity transformation)
    prior to binning
- name: Pyteomics
  role: Loads MS/MS spectra from MGF peak files
- name: NumPy
  role: Assembles the hashed vectors into an array (hashed_vectors.npy) consumed by
    the nearest neighbor index
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:2bc2e23575528e8a39f0b3aed41ad901f13e4edfe6e65a02ef6fac5b2a6b8eb3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - NumPy
  - Numba
  - Pyteomics
  - Python
  - SciPy
  - falcon
  - spectrum_utils
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# high-resolution-spectrum-mz-binning

## Summary

Converts high-resolution MS/MS spectra into compact, low-dimensional vectors by first binning peaks into small m/z bins and then applying MurmurHash3 feature hashing, so that cosine similarity between original spectra is conserved and nearest neighbor indexes can be built efficiently for large-scale clustering.

## When to use

Use when peak files (mzML, mzXML, or MGF) of MS/MS spectra must be vectorized for fast similarity searching or clustering, and exhaustive pairwise spectrum comparison is infeasible (e.g. millions of spectra).

## When NOT to use

- Input is already a feature table, spectral library embedding, or other fixed-length vector representation — re-binning and hashing would destroy existing intensity information
- Spectra are so low-quality that preprocessing discards nearly all of them (<5 peaks or <250 Da m/z range) — no reliable vector can be formed
- Metabolomics or top-down data analyzed with falcon's bottom-up proteomics defaults — the min_peaks/min_mz_range and m/z window settings (default 101–500 m/z) likely need adjustment before binning

## Inputs

- MS/MS peak files in mzML, mzXML, or MGF format
- Preprocessed spectra (precursor peaks and low-intensity noise peaks removed, top-50 most intense peaks retained, spectra with <5 peaks or <250 Da m/z range discarded)
- Vectorization parameters: mass bin width (e.g. 0.05 m/z), target low dimensionality (low_dim, e.g. 800), hash function (MurmurHash3)

## Outputs

- Dense low-dimensional hashed spectrum vectors, one per spectrum (e.g. an (n_spectra, 800) array persisted as hashed_vectors.npy)
- Vector representation consumed downstream to build nearest neighbor indexes and compute the sparse pairwise cosine distance matrix

## How to apply

Load spectra from the peak files and preprocess each one: remove precursor-ion peaks and low-intensity noise peaks, restrict to the 50 most intense peaks, discard spectra with fewer than five peaks or an m/z range below 250 Da, and optionally apply square-root intensity scaling (the falcon default is no scaling, but square-root scaling often gives good results and shifts the appropriate cosine eps threshold). Convert each preprocessed spectrum to a sparse vector using a small mass bin width (e.g. 0.05 m/z) to tightly capture fragment masses. Then hash each sparse high-dimensional vector to a low-dimensional dense vector (length controlled by the low_dim setting, e.g. 800) using the non-cryptographic MurmurHash3 hash function, which maps each mass bin separately to a small number of hash bins. The rationale is that this feature hashing approximately conserves the cosine similarity (and Euclidean norm) between hashed vectors, so the low-dimensional vectors can be used in the subsequent Voronoi-diagram inverted-index nearest neighbor search instead of raw spectra.

## Related tools

- **falcon** (Implements the binning + feature hashing pipeline that converts spectra to low-dimensional vectors; open source under the BSD license) — https://github.com/bittremieux/falcon
- **spectrum_utils** (Used for spectrum preprocessing (peak filtering, intensity transformation) prior to binning)
- **Pyteomics** (Loads MS/MS spectra from MGF peak files)
- **NumPy** (Assembles the hashed vectors into an array (hashed_vectors.npy) consumed by the nearest neighbor index)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- hashed_vectors.npy exists, loads with numpy.load, and has shape (n_spectra, low_dim) with finite values (e.g. 800 columns)
- Feature hashing approximately conserves the Euclidean norm (and cosine similarity) of the original spectra — spot-check cosine distances between a few vector pairs against direct spectrum similarity
- The number of vectors matches the number of spectra surviving preprocessing (spectra with <5 peaks or <250 Da m/z range should be absent)
- Downstream nearest neighbor search on the hashed vectors reproduces cosine distance approximations consistent with a fragment mass tolerance of ~0.05 Da

## Limitations

- Hashed vectors only approximate the true cosine distance; smaller low_dim vectors are faster and cheaper but less accurate, and n_probe/n_neighbors settings trade clustering speed against accuracy
- The intensity scaling method (square root, logarithm, rank, or none) influences the cosine eps threshold used in downstream DBSCAN clustering, so the scaling choice must be paired with the right eps
- The paper notes the influence of MS/MS spectrum preprocessing on various similarity measures was not evaluated; the full effect of chimeric spectra on clustering also remains an open question

## Evidence
