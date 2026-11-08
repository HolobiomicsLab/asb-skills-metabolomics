---
name: cosine-similarity-computation-msms
description: Use when you need to compare every reference spectrum against every query
  spectrum (or all-vs-all) from MGF/mzML inputs to identify similar spectra via cosine
  scores and matching peak counts.
when_to_use_negative:
- The spectra have not been cleaned and normalized — CosineGreedy assumes intensities
  normalized to [0,1] so score magnitudes are comparable.
- The analysis goal is ranking library candidates by chemical identity rather than
  spectral similarity — fingerprint-based comparisons (e.g., add_fingerprint, Spec2Vec,
  MS2DeepScore) would be more appropriate.
- The task only requires pairwise comparison of a few spectra — the all-pairs calculate_scores
  overhead is unnecessary.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Provides calculate_scores, CosineGreedy similarity function, load_from_mgf,
    and the Scores object for all-pairs spectrum comparison
  repo: https://github.com/matchms/matchms
- name: NumPy
  role: Underpins the NumPy-based score implementations enabling fast large-scale
    similarity computation
- name: Numba
  role: Supports the fast score computation backend for large-scale comparisons
- name: pyteomics
  role: Built-in importer backend for mzML/mzXML/MGF file parsing
- name: pymzml
  role: Alternative metadata parsing backend for mzML input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:2e1f0a08d1c555f4195f47f38abd12102d187b291e9f11e1ce7b470170297b08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - NumPy
  - Numba
  - SciPy
  - matchms
  - pymzml
  - pyteomics
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cosine-similarity-computation-msms

## Summary

Compute pairwise cosine similarity scores between all reference and query MS/MS spectra using matchms's calculate_scores with CosineGreedy, yielding per-pair score and matching-peak counts, with the paper's example threshold of ≥20 matching peaks.

## When to use

Use when you need to compare every reference spectrum against every query spectrum (or all-vs-all) from MGF/mzML inputs to identify similar spectra via cosine scores and matching peak counts.

## When NOT to use

- The spectra have not been cleaned and normalized — CosineGreedy assumes intensities normalized to [0,1] so score magnitudes are comparable.
- The analysis goal is ranking library candidates by chemical identity rather than spectral similarity — fingerprint-based comparisons (e.g., add_fingerprint, Spec2Vec, MS2DeepScore) would be more appropriate.
- The task only requires pairwise comparison of a few spectra — the all-pairs calculate_scores overhead is unnecessary.

## Inputs

- Pre-processed MS/MS spectra loaded from MGF (e.g., pesticides.mgf) or other supported formats (mzML, mzXML, msp, JSON, USI)
- Spectra previously passed through metadata-cleaning filters (default_filters) and intensity normalization (normalize_intensities)

## Outputs

- A matchms.Scores object containing per-pair entries (reference, query, score, n_matching)
- A CSV of retained pairs with columns reference_scan_id, query_scan_id, score, n_matching (only pairs meeting the n_matching ≥ 20 threshold, excluding self-comparisons)

## How to apply

Load the pre-processed spectra (previously cleaned with default_filters and normalize_intensities) from the MGF file via matchms.load_from_mgf. Call matchms.calculate_scores with references=spectrums, queries=spectrums, and similarity_function=CosineGreedy() so the Scores object contains a result for every reference–query pair. Iterate over the Scores object, unpack each entry as (reference, query, score, n_matching), and skip self-comparisons where reference is query. Apply the paper's example threshold of retaining only pairs with at least 20 matching peaks (n_matching ≥ 20), then collect reference scan id, query scan id, score, and n_matching from spectrum.metadata['scans']. Scores produced by CosineGreedy are bounded in [0, 1]. The rationale is that matchms computes these scores via NumPy-based implementations at comparably fast speed for large numbers of spectra.

## Related tools

- **matchms** (Provides calculate_scores, CosineGreedy similarity function, load_from_mgf, and the Scores object for all-pairs spectrum comparison) — https://github.com/matchms/matchms
- **NumPy** (Underpins the NumPy-based score implementations enabling fast large-scale similarity computation)
- **Numba** (Supports the fast score computation backend for large-scale comparisons)
- **pyteomics** (Built-in importer backend for mzML/mzXML/MGF file parsing)
- **pymzml** (Alternative metadata parsing backend for mzML input)

## Examples

```
from matchms.similarity import CosineGreedy; from matchms import calculate_scores; scores = calculate_scores(spectrums, spectrums, CosineGreedy())
```

## Evaluation signals

- Every output row must come from a distinct reference–query pair with reference != query (self-comparisons skipped).
- All retained rows must satisfy n_matching ≥ 20; verify by re-reading the CSV.
- All scores must lie in [0, 1] as produced by CosineGreedy.
- CSV columns must exactly be reference_scan_id, query_scan_id, score, n_matching.
- Scores object should contain one entry per non-trivial pair when all-pairs computation is run without sparse pre-selection.

## Limitations

- Not all pairs and scores need to be computed or stored — matchms >= 0.18.0 shifted to sparse scores handling, meaning null/uncomputed values are omitted.
- Large-scale comparisons rely on faster pre-selection measures and sparse storage formats; running naive all-pairs on several hundred thousand spectra may be impractical without these optimizations.
- Score magnitudes depend on prior normalization and filtering steps; inconsistent or wrongly labeled metadata in large collected datasets must be cleaned beforehand.
- Only pairs with ≥20 matching peaks are retained per the paper's example threshold; other similarity measures (e.g., Spec2Vec, MS2DeepScore, CudaCosineGreedy) follow the same API but are separate packages.

## Evidence

- [readme] All-pairs CosineGreedy scoring with calculate_scores: [evidence span withheld — anchor status: external_doc]
- [readme] Scores bounded [0,1] via CosineGreedy: [evidence span withheld — anchor status: not_found]
- [abstract] Import spectra from MGF: "Raw data can be imported from the commonly used formats msp, mzML (Martens et al., 2011), mzXML, MGF"
- [abstract] NumPy-based fast similarity scoring: "matchms facilitates easily deriving similarity measures between large number of spectra at comparably fast speed due to score implementations based on NumPy"
- [readme] Sparse scores in matchms >= 0.18.0: [evidence span withheld — anchor status: external_doc]
