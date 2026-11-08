---
name: matchms-scores-object-inspection
description: Use when you have already loaded and filtered MS/MS spectra (e.g. from
  an MGF file with default_filters and normalize_intensities) and need all-pairs reference-vs-query
  cosine similarity scores with matching-peak counts.
when_to_use_negative:
- The goal is metadata cleaning or peak filtering of individual spectra (use matchms
  filter functions like default_filters, normalize_intensities, select_by_mz) rather
  than pairwise scoring
- The analysis requires structural/fingerprint or machine-learned similarity (e.g.
  Spec2Vec or MS2DeepScore) rather than the peak-based cosine score
- You need a single pairwise comparison between two known spectra rather than the
  many-to-many Scores matrix
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides calculate_scores, the Scores object, and the CosineGreedy similarity
    function used to produce and inspect per-pair score results
  repo: https://github.com/matchms/matchms
- name: NumPy
  role: Underlies the score implementations that make large numbers of all-pairs comparisons
    comparably fast
- name: pyteomics
  role: Backs the MGF (and mzML, mzXML) file importers used to load the spectra before
    scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:36dd7fbc731ec0d2243c31234307ea7657aa5807ecafb216a0152fe6a015ff13
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# matchms-scores-object-inspection

## Summary

Computes and inspects all-pairs spectral similarity results with matchms.calculate_scores using the CosineGreedy similarity function, unpacking per-pair (reference, query, score, n_matching) tuples from the Scores object. This is how matchms turns a filtered MGF spectrum set into a table of cosine scores with matching-peak counts.

## When to use

Use when you have already loaded and filtered MS/MS spectra (e.g. from an MGF file with default_filters and normalize_intensities) and need all-pairs reference-vs-query cosine similarity scores with matching-peak counts.

## When NOT to use

- The goal is metadata cleaning or peak filtering of individual spectra (use matchms filter functions like default_filters, normalize_intensities, select_by_mz) rather than pairwise scoring
- The analysis requires structural/fingerprint or machine-learned similarity (e.g. Spec2Vec or MS2DeepScore) rather than the peak-based cosine score
- You need a single pairwise comparison between two known spectra rather than the many-to-many Scores matrix

## Inputs

- Pre-processed MS/MS spectra loaded from an MGF file (previously cleaned with default_filters and normalize_intensities)
- Similarity function choice, e.g. matchms.similarity.CosineGreedy (or modified cosine score)
- Optional matching-peak threshold (e.g. n_matching >= 20) and peak-match tolerance (e.g. tolerance: 1.0 in the README example)

## Outputs

- A matchms Scores object containing per-pair (reference, query, score, n_matching) entries
- A CSV file with columns reference_scan_id, query_scan_id, score, n_matching for retained pairs

## How to apply

Load the pre-processed spectra from the MGF input with matchms.load_from_mgf, then call matchms.calculate_scores with references=spectrums, queries=spectrums and similarity_function=CosineGreedy() so every reference spectrum is compared to every query spectrum through the matchms.Scores object. Iterate over the Scores object and unpack each entry as (reference, query, score, n_matching), skipping self-comparisons where reference is query. Apply a matching-peak threshold such as the paper's example of n_matching >= 20 to retain only pairs with sufficient peak overlap. For each retained pair, pull the scan identifiers from spectrum.metadata['scans'] and write records (reference_scan_id, query_scan_id, score, n_matching) to a CSV. Note that newer matchms versions (>= 0.18.0) shift to sparse score storage, storing only computed, non-null values, and offer a Pipeline/yaml alternative for the same workflow.

## Related tools

- **matchms** (Provides calculate_scores, the Scores object, and the CosineGreedy similarity function used to produce and inspect per-pair score results) — https://github.com/matchms/matchms
- **NumPy** (Underlies the score implementations that make large numbers of all-pairs comparisons comparably fast)
- **pyteomics** (Backs the MGF (and mzML, mzXML) file importers used to load the spectra before scoring)

## Examples

```
from matchms import calculate_scores, import_from_mgf; from matchms.similarity import CosineGreedy; spectrums = list(import_from_mgf('pesticides.mgf')); scores = calculate_scores(spectrums, spectrums, CosineGreedy())
```

## Evaluation signals

- Re-read the output CSV and verify the exact column schema: reference_scan_id, query_scan_id, score, n_matching
- Confirm no self-comparison rows exist (reference scan id never equals query scan id)
- Verify every retained row has n_matching >= 20 (the paper's example threshold)
- Verify all score values fall in [0, 1] as produced by CosineGreedy
- Row count should not exceed the number of distinct reference/query pairs in the input MGF

## Limitations

- Newer matchms versions (>= 0.18.0) handle scores sparsely — only computed, non-null values are stored — so iteration over the Scores object will not yield entries for pairs that were not computed
- The CosineGreedy score is a peak-based measure; the article also lists the modified cosine score as an alternative, so pair retention depends on the chosen similarity function and tolerance
- All-pairs comparison can be large; the README notes that not all pairs and scores are equally important and recommends sparse storage or pre-selection with faster measures for very large spectrum sets

## Evidence

- [abstract] CosineGreedy all-pairs scoring: "matchms provides functions to derive different similarity scores between spectra. Those include the established spectra-based measures of the cosine score or modified cosine score"
- [abstract] Speed of score computation: "matchms facilitates easily deriving similarity measures between large number of spectra at comparably fast speed due to score implementations based on NumPy"
- [readme] Sparse scores array in matchms >= 0.18.0: [evidence span withheld — anchor status: external_doc]
- [readme] README pipeline score-computation example: [evidence span withheld — anchor status: external_doc]
