---
name: score-table-csv-export
description: Use when an all-vs-all matchms Scores object (CosineGreedy over pre-filtered
  MGF spectra) must be reduced to a tabular record of significant pairs, skipping
  self-comparisons and keeping only pairs with at least 20 matching peaks.
when_to_use_negative:
- The spectra have not yet been cleaned with default_filters and normalize_intensities
  — unfiltered peak lists yield unreliable CosineGreedy pairs before tabulation
- A score row is needed for every possible pair regardless of peak overlap — the n_matching
  >= 20 threshold deliberately discards weakly matching pairs
- Results must stay in matchms' native sparse Scores representation (e.g. for Pipelines
  using filter_by_range or subsequent scoring stages) rather than being flattened
  to CSV
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3517
tools:
- name: matchms
  role: Provides load_from_mgf, calculate_scores, the CosineGreedy similarity function,
    and the Scores object iterated to extract (reference, query, score, n_matching)
    rows for CSV export
  repo: https://github.com/matchms/matchms
- name: NumPy
  role: Underlies matchms' score implementations, enabling fast all-pairs CosineGreedy
    computation before tabular export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.21105/joss.02411
    title: matchms - processing and similarity evaluation of mass spectrometry data.
  source_content_hash: sha256:05a27d390302ddc2ffffae673f98dde9ea6587a714ae869194721fa1b2dd2bb0
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# score-table-csv-export

## Summary

Persist the all-pairs CosineGreedy similarity results produced by matchms.calculate_scores into a flat CSV table with columns reference_scan_id, query_scan_id, score, and n_matching, retaining only distinct spectrum pairs passing the paper's n_matching >= 20 threshold. This makes the pairwise scores (bounded in [0, 1]) auditable and reusable outside the matchms Scores object.

## When to use

Use when an all-vs-all matchms Scores object (CosineGreedy over pre-filtered MGF spectra) must be reduced to a tabular record of significant pairs, skipping self-comparisons and keeping only pairs with at least 20 matching peaks.

## When NOT to use

- The spectra have not yet been cleaned with default_filters and normalize_intensities — unfiltered peak lists yield unreliable CosineGreedy pairs before tabulation
- A score row is needed for every possible pair regardless of peak overlap — the n_matching >= 20 threshold deliberately discards weakly matching pairs
- Results must stay in matchms' native sparse Scores representation (e.g. for Pipelines using filter_by_range or subsequent scoring stages) rather than being flattened to CSV

## Inputs

- Pre-processed MGf-format spectra file (e.g. tests/testdata/pesticides.mgf), previously passed through default_filters and normalize_intensities
- matchms.Scores object returned by matchms.calculate_scores(references=spectrums, queries=spectrums, similarity_function=CosineGreedy())
- Per-spectrum metadata field 'scans' used as the scan identifier source

## Outputs

- CSV file with columns reference_scan_id, query_scan_id, score, n_matching, one row per retained spectrum pair (n_matching >= 20, self-comparisons excluded)
- CosineGreedy score values in [0, 1] and integer matching-peak counts per pair

## How to apply

Load the pre-processed spectra (previously cleaned with default_filters and normalize_intensities) from the MGF file with matchms.load_from_mgf, then call matchms.calculate_scores with references=spectrums, queries=spectrums, and similarity_function=CosineGreedy() so every reference is compared to every query through the matchms.Scores object. Iterate over the Scores entries, unpacking each as (reference, query, score, n_matching), and skip entries where the reference is the query (self-comparisons). Apply the paper's threshold of retaining only pairs with n_matching >= 20 matching peaks. For each retained pair, extract the scan identifiers from spectrum.metadata['scans'] and collect reference_scan_id, query_scan_id, score, and n_matching. Write the records as a CSV with the header reference_scan_id,query_scan_id,score,n_matching; CosineGreedy scores are cosine similarities so they fall in [0, 1]. Rationale: matchms bases its score implementations on NumPy, enabling fast all-pairs similarity derivation, and newer matchms versions (>= 0.18) store scores sparsely, so explicit row-wise collection is needed before export.

## Related tools

- **matchms** (Provides load_from_mgf, calculate_scores, the CosineGreedy similarity function, and the Scores object iterated to extract (reference, query, score, n_matching) rows for CSV export) — https://github.com/matchms/matchms
- **NumPy** (Underlies matchms' score implementations, enabling fast all-pairs CosineGreedy computation before tabular export)

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; workflow = create_workflow(score_computations=[["cosinegreedy", {"tolerance": 1.0}]]); Pipeline(workflow).run("pesticides.mgf")
```

## Evaluation signals

- Re-read the CSV and confirm the exact column schema: reference_scan_id, query_scan_id, score, n_matching
- Verify every row is a distinct spectrum pair (reference_scan_id != query_scan_id), i.e. self-comparisons were skipped
- Confirm all retained rows satisfy n_matching >= 20 (the paper's example threshold)
- Check all score values lie within [0, 1], as expected for CosineGreedy cosine scores
- Confirm scan identifiers were sourced from spectrum.metadata['scans'] for both reference and query columns

## Limitations

- Since matchms >= 0.18 scores are handled sparsely (only actually computed, non-null values are stored), so absent pairs may simply be uncomputed rather than zero-scored
- The n_matching >= 20 cutoff is the paper's example threshold; other analyses may require different matching-peak cutoffs
- Self-comparisons (reference is query) are excluded by this export procedure and will not appear in the CSV

## Evidence

- [abstract] matchms facilitates easily deriving similarity measures between large number of spectra at comparably fast speed due to score implementations based on NumPy: "matchms facilitates easily deriving similarity measures between large number of spectra at comparably fast speed due to score implementations based on NumPy"
- [readme] A key feature of matchms is its ability to apply various pairwise similarity measures for comparing extensive amounts of spectra.: [evidence span withheld — anchor status: external_doc]
- [readme] we now shifted to a sparse handling of scores in matchms (that means: only storing actually computed, non-null values): [evidence span withheld — anchor status: external_doc]
- [abstract] calculate similarity scores between spectra (cosine score, modified cosine score): [evidence span withheld — anchor status: not_found]
