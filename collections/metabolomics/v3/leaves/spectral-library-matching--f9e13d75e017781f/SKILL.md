---
name: spectral-library-matching
description: Use when you have cleaned MGF (or mzML/mzXML/msp/USI) spectra and need
  all-pairs similarity scores, e.g. cosine or modified cosine, plus matching-peak
  counts to flag related spectra pairs.
when_to_use_negative:
- The input spectra have not yet passed metadata cleaning and peak filters (e.g. require_minimum_number_of_peaks,
  normalize_intensities) — raw noisy spectra will yield unreliable cosine scores
- You need a fingerprint-informed or retention-time-informed comparison rather than
  pure peak-list cosine scoring — Spec2Vec, MS2DeepScore, or RIAssigner-style methods
  from the matchms ecosystem are the intended route
- You want library annotation of a single unknown against a database entry with structure
  identification, rather than an all-pairs similarity scan within one collection
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Calculates pairwise similarity scores between all reference and query spectra
    via calculate_scores with CosineGreedy, returning a Scores object
  repo: https://github.com/matchms/matchms
- name: NumPy
  role: Backend for the score implementations that makes all-pairs comparison comparably
    fast
- name: SciPy
  role: Used together with NumPy and Numba in the similarity score implementations
- name: Numba
  role: Accelerates the similarity score computations in matchms
- name: pyteomics
  role: Underlies the file importers that supply the spectra to be scored
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:64d7788e3ae12e430ff8ab0fe38fe1692bb86fbc13199faac501cbd5ad9cd09e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - NumPy
  - Numba
  - SciPy
  - matchms
  - pyteomics
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-library-matching

## Summary

Pairwise MS/MS spectral similarity scoring across a reference and query spectrum collection using matchms, e.g. comparing all spectra to all spectra with the cosine-based CosineGreedy score and counting matching peaks per pair.

## When to use

Use when you have cleaned MGF (or mzML/mzXML/msp/USI) spectra and need all-pairs similarity scores, e.g. cosine or modified cosine, plus matching-peak counts to flag related spectra pairs.

## When NOT to use

- The input spectra have not yet passed metadata cleaning and peak filters (e.g. require_minimum_number_of_peaks, normalize_intensities) — raw noisy spectra will yield unreliable cosine scores
- You need a fingerprint-informed or retention-time-informed comparison rather than pure peak-list cosine scoring — Spec2Vec, MS2DeepScore, or RIAssigner-style methods from the matchms ecosystem are the intended route
- You want library annotation of a single unknown against a database entry with structure identification, rather than an all-pairs similarity scan within one collection

## Inputs

- Processed MS/MS spectra (matchms Spectrum objects) imported from MGF, mzML, mzXML, msp, JSON, or metabolomics-USI
- Prior filtering metadata (ionmode, adduct, charge, compound name, InChI/SMILES/InChIKey) already cleaned via matchms filter functions
- Matching-peak threshold (e.g. n_matching >= 20) and score tolerance settings for the chosen similarity function

## Outputs

- matchms.Scores object containing per-pair reference, query, score, and matching-peak count
- CSV of retained pairs with columns reference_scan_id, query_scan_id, score, n_matching
- Filterable set of spectra pairs meeting the matching-peak criterion (e.g. >= 20 matching peaks)

## How to apply

Load the already filtered and intensity-normalized spectra (e.g. via matchms.load_from_mgf after applying default_filters and normalize_intensities), then call matchms.calculate_scores with references=queries=spectrums and similarity_function=CosineGreedy() so the matchms.Scores object holds one score per reference-query pair. Iterate over Scores entries as (reference, query, score, n_matching), skipping self-comparisons, and apply the paper's example criterion of retaining only pairs with at least 20 matching peaks. Record spectrum.metadata['scans'] for reference and query, the cosine score in [0, 1], and n_matching for each retained pair, then export the records as CSV with columns reference_scan_id, query_scan_id, score, n_matching. Rationale: matchms derives similarity measures between large numbers of spectra at comparably fast speed because scores are implemented with NumPy/SciPy/Numba; for speed-critical or very large collections, sparse score storage or GPU implementations such as CudaCosineGreedy from the matchms ecosystem can pre-select pairs.

## Related tools

- **matchms** (Calculates pairwise similarity scores between all reference and query spectra via calculate_scores with CosineGreedy, returning a Scores object) — https://github.com/matchms/matchms
- **NumPy** (Backend for the score implementations that makes all-pairs comparison comparably fast)
- **SciPy** (Used together with NumPy and Numba in the similarity score implementations)
- **Numba** (Accelerates the similarity score computations in matchms)
- **pyteomics** (Underlies the file importers that supply the spectra to be scored)

## Examples

```
from matchms.importing import load_from_mgf; from matchms import calculate_scores; from matchms.similarity import CosineGreedy; spectrums = load_from_mgf('pesticides.mgf'); scores = calculate_scores(spectrums, spectrums, CosineGreedy())
```

## Evaluation signals

- Re-read the output CSV and confirm the exact column schema (reference_scan_id, query_scan_id, score, n_matching) is present
- Confirm all reported scores fall in the valid CosineGreedy range [0, 1]
- Confirm no row compares a spectrum to itself (reference is query pairs must be absent)
- Confirm every retained row satisfies the matching-peak threshold (e.g. n_matching >= 20)
- Spot-check that recorded scan ids match spectrum.metadata['scans'] of the corresponding reference and query spectra

## Limitations

- Cosine score ignores precursor mass differences; for structure-aware analog matching the modified cosine score is the matchms alternative
- Matching-peak count and score alone may still admit unrelated pairs if spectra were not metadata-cleaned first; matchms notes collected MS/MS datasets can contain a significant fraction of inconsistent, wrongly labeled, or incorrect metadata
- Exhaustive all-pairs scoring can be expensive on very large libraries; matchms README notes that fast similarity measures for initial pre-selection and sparse storage are used to enable comparing several hundred thousands of spectra
- Cosine/modified cosine are spectra-based measures and do not use molecular fingerprints or machine-learning scores

## Evidence

- [results] calculate_scores with CosineGreedy yields per-pair results (reference, query, score, n_matching): [evidence span withheld — anchor status: not_found]
- [abstract] matchms derives similarity measures between large numbers of spectra at comparably fast speed: "matchms facilitates easily deriving similarity measures between large number of spectra at comparably fast speed due to score implementations based on NumPy"
- [abstract] matchms provides functions to derive different similarity scores between spectra, including cosine and modified cosine: "matchms provides functions to derive different similarity scores between spectra. Those include the established spectra-based measures of the cosine score or modified cosine score"
- [readme] Sparse scores handling and pre-selection for very large comparisons: [evidence span withheld — anchor status: external_doc]
- [abstract] metadata quality caveat motivating prior cleaning before scoring: "large collected datasets contain data acquired using different instruments and measurement conditions, and can further contain a significant fraction of inconsistent, wrongly labeled, or incorrect"
