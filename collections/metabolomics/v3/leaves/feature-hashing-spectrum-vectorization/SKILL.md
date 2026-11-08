---
name: feature-hashing-spectrum-vectorization
description: Use when large collections of high-resolution MS/MS spectra (mzML, mzXML,
  or MGF peak files) must be converted into compact numeric vectors suitable for approximate
  similarity searching or indexing without exhaustive pairwise comparisons.
when_to_use_negative:
- Input is not spectrum data but an already-vectorized feature table or identification
  result — vectorization is redundant
- Metabolomics or top-down data where the default proteomics preprocessing (min 5
  peaks, 250 m/z range, 101–500 m/z window) does not apply; these falcon settings
  must be reduced/adjusted first
- Exact cosine distances between every spectrum pair are required — feature hashing
  only approximates the original cosine similarity
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: falcon
  role: Implements the binning + MurmurHash3 feature-hashing pipeline that converts
    spectra to low-dimensional vectors for clustering
  repo: https://github.com/bittremieux/falcon
- name: Pyteomics
  role: Loads MS/MS spectra from MGF peak files prior to preprocessing and vectorization
- name: spectrum_utils
  role: Preprocesses spectra (peak filtering, square-root intensity transformation)
    before hashing
- name: NumPy/SciPy
  role: Assemble and persist the hashed vector array consumed by the nearest-neighbor
    index
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_001
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:ad073df280ba0b8a4d1b665aea57a2d0359bac8ab4de74280845048ad09b127f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Faiss
  - NumPy
  - NumPy/SciPy
  - Numba
  - Pyteomics
  - Python
  - Python 3.8
  - SciPy
  - Scikit-Learn
  - ThermoRawFileParser
  - falcon
  - fastcluster
  - spectrum_utils
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# feature-hashing-spectrum-vectorization

## Summary

Converts high-resolution MS/MS spectra into fixed-length low-dimensional vectors by binning fragment m/z values and applying MurmurHash3 feature hashing, so that cosine similarity between the original spectra is approximately conserved for fast nearest-neighbor searching in falcon.

## When to use

Use when large collections of high-resolution MS/MS spectra (mzML, mzXML, or MGF peak files) must be converted into compact numeric vectors suitable for approximate similarity searching or indexing without exhaustive pairwise comparisons.

## When NOT to use

- Input is not spectrum data but an already-vectorized feature table or identification result — vectorization is redundant
- Metabolomics or top-down data where the default proteomics preprocessing (min 5 peaks, 250 m/z range, 101–500 m/z window) does not apply; these falcon settings must be reduced/adjusted first
- Exact cosine distances between every spectrum pair are required — feature hashing only approximates the original cosine similarity

## Inputs

- MS/MS peak files in mzML, mzXML, or MGF format (e.g. produced from Thermo raw files with ThermoRawFileParser)
- Preprocessed spectra (precursor/noise peaks removed, <=50 most intense peaks, square-root transformed intensities)

## Outputs

- Dense hashed spectrum vectors of length 800 (one vector per spectrum), e.g. persisted as hashed_vectors.npy
- Vector array consumable by the falcon nearest-neighbor index (Faiss inverted index per precursor-mass bucket)

## How to apply

First preprocess each spectrum (loaded with Pyteomics) by removing precursor-ion peaks and low-intensity noise peaks (below 10% of the base peak intensity), restricting to the 50 most intense peaks, discarding spectra with fewer than five peaks or an m/z range below 250 Da, and square-root transforming peak intensities with spectrum_utils. Convert each preprocessed spectrum to a sparse vector using a 0.05 m/z bin width so fragment masses are tightly captured. Then hash each sparse, high-dimensional vector to a dense length-800 vector using 32-bit MurmurHash3, mapping each mass bin separately to a small number of hash bins; feature hashing conserves cosine similarity between hashed vectors, allowing the hashed vectors to approximate similarities between the original spectra. Assemble the hashed vectors with NumPy/SciPy and persist them (e.g. hashed_vectors.npy) as the input to nearest-neighbor index construction. The vector length (low_dim) trades accuracy of the approximated cosine distance against nearest-neighbor searching time and memory.

## Related tools

- **falcon** (Implements the binning + MurmurHash3 feature-hashing pipeline that converts spectra to low-dimensional vectors for clustering) — https://github.com/bittremieux/falcon
- **Pyteomics** (Loads MS/MS spectra from MGF peak files prior to preprocessing and vectorization)
- **spectrum_utils** (Preprocesses spectra (peak filtering, square-root intensity transformation) before hashing)
- **NumPy/SciPy** (Assemble and persist the hashed vector array consumed by the nearest-neighbor index)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- hashed_vectors.npy exists, loads with numpy.load, and has shape (n_spectra, 800) with all values finite
- Feature hashing approximately conserves the Euclidean/cosine norm: cosine similarity between hashed vector pairs matches similarity computed on the original binned sparse spectra
- Every spectrum with at least min_peaks (default 5) peaks produces exactly one vector; low-quality spectra are dropped before hashing
- Downstream nearest-neighbor search over the hashed vectors retrieves neighbors consistent with 0.05 m/z fragment-bin spacing (fragment_tol 0.05)

## Limitations

- Hashed vectors only approximate the true cosine distance; larger low_dim vectors improve accuracy at the cost of search time and memory
- The intensity scaling method (square root, log, rank, or none) influences the cosine threshold eps, so vectorization settings interact with clustering purity
- Default preprocessing targets bottom-up proteomics; metabolomics/top-down data need adjusted min_peaks, min_mz_range, and m/z window settings
- The effect of preprocessing choices on alternative similarity measures (e.g. probabilistic or fragment-rarity scores) was not evaluated

## Evidence
